# source
##### 项目地址：[css-spring](https://github.com/codepunkt/css-spring)
(言语粗糙，请结合源码阅读)

## feature
##### 由js生成**弹簧**效果的css动画帧，以满足贝塞尔无法覆盖到的场景，适用于如styled-component等的css in js方案。

## main dependencies
- css-tree
- springer
- lodash

## start
项目思路，暴露spring函数给用户，生成一个类似如下的字符串。
```
/**
 * Function spring
 * @param {Object} startStyles (from)
 * @param {Object} endStyles (to)
 * @param {Object} options（steps，tension，wobble等）
 */
```
```
    { '0%': { border: '1px solid #0000fe' },
    `
    `
    `
    '100%': { border: '25px solid #bada55' } }
```
值得记录的是各工具函数对参数的处理

### 工具函数分析

#### parseDeclarations
在这个函数里调用css-tree库，parse转为ast树，返回其中的Declaration节点。

#### getAnimationValues
寻找开始与结束两者都存在的Declaration节点
1、将Declaration节点内的'property'(即css key)map出来作为数组，传给reduce作为数组参数，在构造一个函数作为reduce的函数参数；

2、函数参数的作用：返回一个类似下面这种的对象；
```
{
    border: {
        start: [Declaration节点],
        end: [Declaration节点]
    }
}
```
(其中主要借助了lodash的intersectionBy, map, reduce方法)

#### sanitizeValues
将getAnimationValues返回对象里的start和end对象进行整合，合并其中相同的属性，将value分为start和end两部分，返回值：
```
{ 
    border: [
        { type: 'Dimension', unit: 'px', start: 1, end: 25 },
        { type: 'Fixed', value: ' ' },
        { type: 'Fixed', value: 'solid' },
        { type: 'Fixed', value: ' ' },
        { type: 'HexColor', start: '00f', end: 'bada55' }
    ]
}
```

#### getInterpolator
参数：tension(张力值)、wobble(摇晃值)、steps(步骤数)；

将tension和wobble传入springer模块，经插件处理生成spring函数（缓动函数easing，输入0~1的参数）；

依据steps，返回对应类型的插值函数，类型是Declaration节点的属性：fixed（固定值）、number（数字），hex（颜色）；

- fixed：每一步的固定值都一样，如边框的'solid'属性，每一步都是相同的；

- number：数值则使用插值函数lerp, 将spring(i / steps)作为加速度传入，得到每一步的数值；

- hex：将颜色转换为rgb再进行插值运算，得到每一步的颜色；

这里面有两个小技巧：

1、需要对一个有长度的空数组进行map，可以这样写`[...Array(length)].map(fun)`，(可以用在react render渲染固定length列表);

2、使用位运算去将16进制色值转为rgb，再转换回去
```
  const hex = parseInt(a.replace(/#/g, ''), 16)
  const r = hex >> 16,
    g = (hex >> 8) & 0xff,
    b = hex & 0xff
```
tips:lerp线性插值原理自行google
