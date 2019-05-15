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

​	2.初始化

```
lastBitArr:[];
finalScore:number = 1;
binMax = patternLen + textLen;
mask = 1 << (patternLen - 1)
```





