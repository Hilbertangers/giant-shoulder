### Default

```
const defaultList = ['Apple', 'Orange', 'Banana']
const defaultOptions = {
  location: 0,
  distance: 100,
  threshold: 0.6,
  maxPatternLength: 32,
  isCaseSensitive: false,
  tokenSeparator: / +/g,
  findAllMatches: false,
  minMatchCharLength: 1,
  id: null,
  keys: [],
  shouldSort: true,
  getFn: deepValue,
  sortFn: (a, b) => (a.score - b.score),
  tokenize: false,
  matchAllTokens: false,
  includeMatches: false,
  includeScore: false,
  verbose
}
```

### Result

1.纯字符串 完全匹配

```
fuse.search('Orange') // expect 1
```

