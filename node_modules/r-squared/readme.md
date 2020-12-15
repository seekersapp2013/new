# r-squared

[![build status](http://img.shields.io/travis/Balou9/r-squared.svg?style=flat)](http://travis-ci.org/Balou9/r-squared) [![AppVeyor Build Status](https://ci.appveyor.com/api/projects/status/github/Balou9/r-squared?branch=master&svg=true)](https://ci.appveyor.com/project/Balou9/r-squared)

***

r-squared tells us how well a regression line predicts or estimates actual values

***

## Get it!

```
npm install --save r-squared
```

***

## Usage

``` js
var rSquared = require('rSquared')

var x = [1,2,3,4,5]
var y = [2,4,5,4,5]

rSquared.rSquared(x, y, function (err, data) {
  if (err) throw err
  console.log(data)
})

```

***

## API

### `r-squared(x, y, callback)`

`x` actual values  
`y` estimated values  
`callback` error first callback

***

## License

[MIT](./license.md)
