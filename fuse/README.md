# source

##### 项目地址：[Fuse.js](https://github.com/krisk/fuse)

v3.4.3

## feature

##### 由js完成的轻量模糊搜索库，没有依赖。

##  start

由单元测试入手，会好理解一些。

##  unit test

使用的是jest。测试用例有很多，可以用fdescribe，fit语法，逐个进行分析。

我简单打印出了用例的[结果](./test.mock.md)。

## options

[参数及其作用](./option.md)

## analysis

入口文件是src/index.js文件，定义Fuse类，调用类中的search方法则开始搜索，并返回结果。

#### search:

###### A.处理参数

​	1.调用_prepareSearchers方法，传入pattern，返回tokenSearchers，fullSearcher。内部使用了bitap类。([bitap算法分析](./bitap.md))。

​	2.如**test1**中，fullSearcher返回了如下对象：

```js
{
    "options":{
        "location":0,
        "distance":100,
        "threshold":0.6,
        "maxPatternLength":32,
        "isCaseSensitive":false,
        "tokenSeparator":{

        },
        "findAllMatches":false,
        "minMatchCharLength":1
    },
    "pattern":"orange",
    "patternAlphabet":{
        "o":32,
        "r":16,
        "a":8,
        "n":4,
        "g":2,
        "e":1
    },
    search (text) {
      ......
      return {
        ......
      }
    }
}
```

###### B.搜索函数

​	1.将tokenSearchers，fullSearcher赋值给\_search方法。

​	2.执行\_search，初始化

```
resultMap:{}
results:[]
weight:{} | null
```

​	对list列表循环调用[\_analyze函数](./_analyze.md)。

​	**规定**：对列表list[0]进行判断，根据数据类型的不同，赋值给\_analyze的参数也不同,若是string则plan a，否则plan b。

​	plan a: ….(一些固定值)

​	plan b: 对`options.key`进行遍历，得到对象key`{name,  weight}`,并赋值给weight，然后将key变成字符串，执行\_analyze。

​	3.\_analyze会对results进行push。

​	4.如**test1**中，最后返回：

```js
{
    "weights":null,
    "results":[
        {
            "item":1,
            "output":[
                {
                    "key":"",
                    "arrayIndex":-1,
                    "value":"Orange",
                    "score":0,
                    "matchedIndices":[
                        [
                            0,
                            5
                        ]
                    ]
                }
            ]
        }
    ]
}
```

###### C.计算每个匹配项的相似分数

​	1.将weight和results赋值给\_computeScore;

​	2.对result和result.output循环，计算出`nScore = score * weight`,再定义当前分数`currScore = 1`，叠乘`currScore *= nScore`。

​	3.将currScore赋值给result[i]的score属性。

###### D.根据分数排序

​	默认根据score进行sort升序。

###### E.根据配置项过滤结果值，做美化

​	1.根据limit限制匹配数量；

​	2.根据includeMatches，includeScore，id做对应build。









