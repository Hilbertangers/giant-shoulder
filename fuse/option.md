|       option       |  default  |                         introduction                         |
| :----------------: | :-------: | :----------------------------------------------------------: |
|      location      |     0     |                    期待参数被查询到的位置                    |
|      ditance       |    100    |                                                              |
|     threshold      |    0.6    |     匹配阈值，比如0的话需要最优解，<br />1的话匹配任意值     |
|  maxPatternLength  |    32     |                        参数的最大长度                        |
|  isCaseSensitive   |   false   |                        是否区分大小写                        |
|   tokenSeparator   |   / +/g   |       用于分割参数的正则。<br />只有tokenize为true生效       |
|   findAllMatches   |   false   |            会一直搜索到最后，即时你已经找到最优解            |
| minMatchCharLength |     1     |                       匹配值的最小长度                       |
|         id         |   null    |                 ⭐️指定返回匹配项的某个属性值                  |
|        keys        |    []     |            对象将会被搜索的属性，也支持嵌套的属性            |
|     shouldSort     |   true    |              是否依据匹配分数的高低将匹配项排序              |
|   includeMatches   |   false   |                     返回值是否包含匹配项                     |
|    includeScore    |   false   |                    返回值是否包含匹配分数                    |
|       getFn        | deepValue |                     对象属性深度获取函数                     |
|       sortFn       | 按分升序  |  多个匹配值的排序算法，<br />(a, b) => (a.score - b.score)   |
|      verbose       |   false   |                    是否开启调试(console)                     |
|      tokenize      |   false   | 搜索算法将搜索单个单词和整个字符串<br />不是很理解 看下代码吧 |
|   matchAllTokens   |   false   |               当tokenize为true 匹配所有的token               |