# source
##### 项目地址：[css-spring](https://github.com/codepunkt/css-spring)
(言语粗糙，请结合源码阅读)

## feature
##### 由js生成**弹簧**效果的css动画帧，以满足贝塞尔无法覆盖到的场景，适用于如styled-component等的css in js方案。

## main dependencies
- css-tree
- springer

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

