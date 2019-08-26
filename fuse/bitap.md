### introduction

**bitap算法**是一种[字符串近似匹配](https://zh.wikipedia.org/wiki/字符串近似匹配)算法。此算法可以计算一段给定的字符串是否含有“约等于”给定模式串的子串，其中的“约等于”是用[莱文斯坦距离](https://zh.wikipedia.org/wiki/萊文斯坦距離)定义的——如果子串和模式串的距离小于等于给定的*k*，算法就认为它们是匹配的。该算法先进行预处理计算一个[掩码](https://zh.wikipedia.org/wiki/掩码)集，其中每个位代表一个字符是否在模式串中出现过。于是，几乎所有的操作都是**位操作**，速度非常快。(来自维基)

莱文斯坦距离：最小编辑距离，之前有用动态规划做过  [看这里](https://github.com/Hilbertangers/codeWar/blob/master/questions/questions_03.md)

### helper

###### A.bitap_pattern_alphabet

​	将pattern字符串的每个字符进行按位操作,

```js
pattern.chart(i) |= (1 << (length - i -1))// 或 一个左移操作，左移的比特位根据遍历递减
```

###### B.bitap_score

​	匹配项分数计算，

```js
  const accuracy = errors / pattern.length
  const proximity = Math.abs(expectedLocation - currentLocation)
  if (!distance) {
    return proximity ? 1.0 : accuracy
  }
  return accuracy + (proximity / distance)
```



### algorithm

###### A.先看简单的一种情况，参数长度超过maxPatternLength

使用正则搜索，对应代码里的`bitapRegexSearch`方法。

```js
  /**
   * @param {string} text 给定字符串
   * @param {string} pattern 待匹配字符串
   * @param {Regex} tokenSeparator 正则匹配规律
   */
```

​	1.过滤掉pattern的特殊符号，并根据tokenSeparator分割pattern；

​	2.使用`text.match(regex)`判断是否匹配，若存在匹配值，则赋值matchedIndices。

###### B.常规情况下bitap search

```js
  /**
   * @param {string} text 给定字符串
   * @param {string} pattern 待匹配字符串
   * @param {object} patternAlphabet 待匹配字符串的首字母对象
   * @param {object} options 参数项
   */
	return {
    isMatch: bestLocation >= 0,
    score: finalScore === 0 ? 0.001 : finalScore,
    matchedIndices: matchedIndices(matchMask, minMatchCharLength)
  }
```

​	1.`text.indexOf(pattern, expectedLocation)!==-1`即text中包含pattern，则直接调用:

```js
let score = bitapScore(pattern, {
  error: 0,
  currentLocation: bestLocation,
  expectedLocation,
  distance
})
```

得到currentThreshold值。

​	2.进入bitap查询。

先*根据字符长度和location，distance，threshold三个字段判断做bitap查询的范围*

```javascript
  let binMin = 0
    let binMid = binMax

    while (binMin < binMid) {
      const score = bitapScore(pattern, {
        errors: i,
        currentLocation: expectedLocation + binMid,
        expectedLocation,
        distance
      })
      // console.log("TCL: score", score)
      // console.log("TCL: currentThreshold", currentThreshold)

      if (score <= currentThreshold) {
        binMin = binMid
      } else {
        binMax = binMid
      }

      binMid = Math.floor((binMax - binMin) / 2 + binMin)
    }

    // Use the result from this iteration as the maximum for the next.
    binMax = binMid
    console.log("TCL: binMid", binMid)
    // 根据字符长度和location，distance，threshold三个字段判断做bitap查询的范围

    let start = Math.max(1, expectedLocation - binMid + 1)
    let finish = findAllMatches ? textLen : Math.min(expectedLocation + binMid, textLen) + patternLen

```

得到start和finish,定义bitArr数组，并循环text。

提前计算得到pattern的字母集

比如这种场景时`pattern='ldman';text='oldmanb'`，得到字母集为`patternAlphabet = { l: 16, d: 8, m: 4, a: 2, n: 1 }`,准备工作做好之后进入字符匹配阶段。

先贴代码

```javascript
    for (let j = finish; j >= start; j -= 1) {
      let currentLocation = j - 1
      let charMatch = patternAlphabet[text.charAt(currentLocation)]

      // 是否匹配当前字符串
      if (charMatch) {
        matchMask[currentLocation] = 1
      }
      
      console.log("TCL: charMatch", charMatch)
      // First pass: exact match
      // 是否连续匹配
      bitArr[j] = ((bitArr[j + 1] << 1) | 1) & charMatch
      console.log("TCL: bitArr", bitArr)

      // Subsequent passes: fuzzy match
      if (i !== 0) {
        bitArr[j] |= (((lastBitArr[j + 1] | lastBitArr[j]) << 1) | 1) | lastBitArr[j + 1]
        console.log("TCL: 2 bitArr", bitArr, j)
        // console.log("TCL: lastBitArr[j + 1]", lastBitArr[j + 1])
        console.log("TCL: lastBitArr[j]", lastBitArr[j])
      }

      if (bitArr[j] & mask) {
        finalScore = bitapScore(pattern, {
          errors: i,
          currentLocation,
          expectedLocation,
          distance
        })
        console.log("TCL: finalScore", finalScore)

        // This match will almost certainly be better than any existing match.
        // But check anyway.
        if (finalScore <= currentThreshold) {
          // Indeed it is
          currentThreshold = finalScore
          bestLocation = currentLocation

          // Already passed `loc`, downhill from here on in.
          if (bestLocation <= expectedLocation) {
            break
          }

          // When passing `bestLocation`, don't exceed our current distance from `expectedLocation`.
          start = Math.max(1, 2 * expectedLocation - bestLocation)
        }
      }
    }
```



3.精准匹配

定义掩码`mask=1 << (patternLen - 1)`,在这个例子里就是`1 << 4`.

再通过`charMatch = patternAlphabet[text.charAt(currentLocation)]`得到text当前字符对应pattern的字母。

*重点*：对bitArr[j]赋值：`bitArr[j] = ((bitArr[j + 1] << 1) | 1) & charMatch`

这一步位运算可以得出pattern在text内是否连续匹配。

如例子中的字符，

text循环中最后一位b在patternAlphabet中不存在 => charMatch===undefined => 当前的bitArr[j] = 0;

倒数第二位n在patternAlphabet中存在 => charMatch===1 => 当前的bitArr[j] = 1;

倒数第三位a在patternAlphabet中存在 => charMatch===2 => 当前的bitArr[j] = 2;

以此类推到倒数第六位l是 => charMatch === 16 => bitArr[j] = 16,此时是不是发现16 === 1<<4,也就是`bitArr[j] & mask`为true，精准匹配成功，计算出finalScore。

*分析*：`bitArr[j] = ((bitArr[j + 1] << 1) | 1) & charMatch`

bitArr数组定义时，给他的最后一个元素赋值0。每次求当前值就对前一个值，左移一位，再或上一个1，保证第一位是有值的，然后与上当前匹配值charMatch。这样的话若前一个值是不匹配的0，那当前值必须是pattern字符的最后一个字母，否则与上charMatch的结果也会是0。

4.模糊匹配

若精准匹配得到的finalScore小于既定的一个score，则进入pattern的下一次循环，此时进行精准匹配+模糊匹配

```javascript
      bitArr[j] = ((bitArr[j + 1] << 1) | 1) & charMatch

      // Subsequent passes: fuzzy match
      if (i !== 0) {
        bitArr[j] |= (((lastBitArr[j + 1] | lastBitArr[j]) << 1) | 1) | lastBitArr[j + 1]
      }
```

