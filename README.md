# d3-interpolate

这个模块提供了在两个值之间多种插值方法。值可能是数值, 颜色, 字符串, 数组甚至是多层嵌套的数组. 例如：

```js
var i = d3.interpolateNumber(10, 20);
i(0.0); // 10
i(0.2); // 12
i(0.5); // 15
i(1.0); // 20
```

返回的函数 `i` 被称为 *interpolator*(插值器). 给定一个起始值 *a* 和一个终止值 *b*, 传入一个范围在 [0, 1] 之间的参数 *t* 会返回一个对应的在 *a* 和 *b* 之间的值。插值器通常在 *t* = 0 时候返回 *a*, 并且在 *t* = 1 时返回 *b*.

你可以对数值之外的其他值进行插值。比如返回 `steelblue` 和 `brown` 之间感知上处于中点的颜色:

```js
d3.interpolateLab("steelblue", "brown")(0.5); // "rgb(142, 92, 109)"
```

下面是一个更详细的例子说明 [interpolate](#interpolate) 的类型推断:

```js
var i = d3.interpolate({colors: ["red", "blue"]}, {colors: ["white", "black"]});
i(0.0); // {colors: ["rgb(255, 0, 0)", "rgb(0, 0, 255)"]}
i(0.5); // {colors: ["rgb(255, 128, 128)", "rgb(0, 0, 128)"]}
i(1.0); // {colors: ["rgb(255, 255, 255)", "rgb(0, 0, 0)"]}
```

请注意，通用值插值器不仅会检测嵌套对象和数组，还会检测字符串中嵌入的颜色字符串和数字！

## Installing

`NPM` 安装: `npm install d3-interpolate`. 此外还可以下载 [latest release](https://github.com/d3/d3-interpolate/releases/latest). 你可以直接从 [d3js.org](https://d3js.org) 作为 [standalone library](https://d3js.org/d3-interpolate.v1.min.js) 或作为 [D3 4.0](https://github.com/d3/d3) 的一部分直接引入. 支持 `AMD`, `CommonJS` 以及基本的标签引入形式。如果使用标签引入则会暴露全局 `d3` 变量:

```html
<script src="https://d3js.org/d3-color.v1.min.js"></script>
<script src="https://d3js.org/d3-interpolate.v1.min.js"></script>
<script>

var interpolate = d3.interpolateRgb("steelblue", "brown");

</script>
```

[在浏览器中测试 `d3-interpolate`.](https://tonicdev.com/npm/d3-interpolate)

## API Reference

<a name="interpolate" href="#interpolate">#</a> d3.<b>interpolate</b>(<i>a</i>, <i>b</i>)

返回一个在两个任意类型值  *a* 和 *b* 之间插值的插值器。插值算法的实现基于终止值 *b* 的类型，使用如下算法:

1. 如果 *b* 为 `null`, `undefined` 或者 `boolean`, 则使用常量 *b*.
2. 如果 *b* 为 `number`, 使用 [interpolateNumber](#interpolateNumber).
3. 如果 *b* 为 [color](https://github.com/d3/d3-color#color) 或可以转为颜色的字符串, 使用 [interpolateRgb](#interpolateRgb).
4. 如果 *b* 为 [date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date), 使用 [interpolateDate](#interpolateDate).
5. 如果 *b* 为 `string`, 使用 [interpolateString](#interpolateString).
6. 如果 *b* 为 [array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray), 使用 [interpolateArray](#interpolateArray).
7. 如果 *b* 可以转为 `number`, 使用 [interpolateNumber](#interpolateNumber).
8. 使用 [interpolateObject](#interpolateObject).

基于选中的插值器，*a* 会被强制转为对应的类型.

<a name="interpolateNumber" href="#interpolateNumber">#</a> d3.<b>interpolateNumber</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/number.js "Source")

返回一个在两个数值 *a* 和 *b* 之间插值的插值器。返回的插值器等价于:

```js
function interpolator(t) {
  return a * (1 - t) + b * t;
}
```

警告: 避免以 `0` 作为插值的起点或终点因为可能会生成字符串。当数字非常小的时候会使用科学计数法表示，这种标记可能会被解析为错误的属性或样式值。比如，数值 `0.0000001` 会被转为字符串 `"1e-7"`。这种现象在对不透明度插值时会尤其明显。为避免科学计数法，可以起于或结束于 `1e-6`：科学计数法中没有被字符串化的最小值。

<a name="interpolateRound" href="#interpolateRound">#</a> d3.<b>interpolateRound</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/round.js "Source")

返回一个在两个数值 *a* 和 *b* 之间插值的插值器; 这个插值器与 [interpolateNumber](#interpolateNumber) 类似但是会对返回的结果进行四舍五入。.

<a name="interpolateString" href="#interpolateString">#</a> d3.<b>interpolateString</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/string.js "Source")

Returns an interpolator between the two strings *a* and *b*. The string interpolator finds numbers embedded in *a* and *b*, where each number is of the form understood by JavaScript. A few examples of numbers that will be detected within a string: `-1`, `42`, `3.14159`, and `6.0221413e+23`.

For each number embedded in *b*, the interpolator will attempt to find a corresponding number in *a*. If a corresponding number is found, a numeric interpolator is created using [interpolateNumber](#interpolateNumber). The remaining parts of the string *b* are used as a template: the static parts of the string *b* remain constant for the interpolation, with the interpolated numeric values embedded in the template.

For example, if *a* is `"300 12px sans-serif"`, and *b* is `"500 36px Comic-Sans"`, two embedded numbers are found. The remaining static parts (of string *b*) are a space between the two numbers (`" "`), and the suffix (`"px Comic-Sans"`). The result of the interpolator at *t* = 0.5 is `"400 24px Comic-Sans"`.

<a name="interpolateDate" href="#interpolateDate">#</a> d3.<b>interpolateDate</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/date.js "Source")

返回一个在两个 [dates](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) *a* 和 *b* 之间插值的插值器;

注意: 返回的日期在创建时 **no defensive copy(没有防御式拷贝)**; 插值器在每次计算时都返回同一个实例(例如: `interpolator(0.1) === interpolator(0.5)` is `true`)。没有拷贝是处于性能方面的考虑; 插值器通常用是 [animated transitions](https://github.com/d3/d3-transition) 内循环的一部分。

<a name="interpolateArray" href="#interpolateArray">#</a> d3.<b>interpolateArray</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/array.js "Source")

返回一个在两个数组 *a* 和 *b* 之间插值的插值器。在内部，会创建一个与数组 *b* 等长的临时数组模板。对于数组 *b* 的每一个元素，如果在 *a* 中也存在则会在这两个元素之间使用一个通用的 [interpolate](#interpolate)。如果 *a* 中没有对应的元素, 则这个值会被当为静态值放到临时数组模板中。然后根据指定的参数 *t*, 计算出临时数组模板中每一个元素的值最后返回。

例如，如果  *a* 为 `[0, 1]` 并且 *b* 为 `[1, 10, 100]`。则 *t* = 0.5 时结果为 `[0.5, 5.5, 100]`.

注意: 返回的数组在创建时 **no defensive copy(没有防御式拷贝)**; 插值器在每次计算时都返回同一个实例(例如: `interpolator(0.1) === interpolator(0.5)` is `true`)。没有拷贝是处于性能方面的考虑; 插值器通常用是 [animated transitions](https://github.com/d3/d3-transition) 内循环的一部分。

<a name="interpolateObject" href="#interpolateObject">#</a> d3.<b>interpolateObject</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/object.js "Source")

返回一个在两个对象 *a* 和 *b* 之间插值的插值器。在内部，会创建一个与对象 *b* 有相同属性的临时对象。对于对象 *b* 的每一个属性，如果在 *a* 中也存在则会在这两个元素之间使用一个通用的 [interpolate](#interpolate)。如果 *a* 中没有对应的属性, 则这个值会被当为静态值放到临时对象模板中。然后根据指定的参数 *t*, 计算出临时对象模板中每一个属性的值最后返回。

例如, 如果 *a* 为 `{x: 0, y: 1}` 并且 *b* 为 `{x: 1, y: 10, z: 100}`, 则 *t* = 0.5 时返回的结果为 `{x: 0.5, y: 5.5, z: 100}`.

对象插值特别适用于 *dataspace interpolation(数据空间插值)*, 也就是对数据插值而不是对属性进行插值。例如你可以对一个描述 `arc` 或者 `pie` 的对象进行插值，然后使用 `d3.arc` 来计算新的路径数据。

注意: 返回的对象在创建时 **no defensive copy(没有防御式拷贝)**; 插值器在每次计算时都返回同一个实例(例如: `interpolator(0.1) === interpolator(0.5)` is `true`)。没有拷贝是处于性能方面的考虑; 插值器通常用是 [animated transitions](https://github.com/d3/d3-transition) 内循环的一部分。

<a name="interpolateTransformCss" href="#interpolateTransformCss">#</a> d3.<b>interpolateTransformCss</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/transform/index.js#L62 "Source")

Returns an interpolator between the two 2D CSS transforms represented by *a* and *b*. Each transform is decomposed to a standard representation of translate, rotate, *x*-skew and scale; these component transformations are then interpolated. This behavior is standardized by CSS: see [matrix decomposition for animation](http://www.w3.org/TR/css3-2d-transforms/#matrix-decomposition).

<a name="interpolateTransformSvg" href="#interpolateTransformSvg">#</a> d3.<b>interpolateTransformSvg</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/transform/index.js#L63 "Source")

Returns an interpolator between the two 2D SVG transforms represented by *a* and *b*. Each transform is decomposed to a standard representation of translate, rotate, *x*-skew and scale; these component transformations are then interpolated. This behavior is standardized by CSS: see [matrix decomposition for animation](http://www.w3.org/TR/css3-2d-transforms/#matrix-decomposition).

<a name="interpolateZoom" href="#interpolateZoom">#</a> d3.<b>interpolateZoom</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/zoom.js "Source")

Returns an interpolator between the two views *a* and *b* of a two-dimensional plane, based on [“Smooth and efficient zooming and panning”](http://www.win.tue.nl/~vanwijk/zoompan.pdf) by Jarke J. van Wijk and Wim A.A. Nuij. Each view is defined as an array of three numbers: *cx*, *cy* and *width*. The first two coordinates *cx*, *cy* represent the center of the viewport; the last coordinate *width* represents the size of the viewport.

The returned interpolator exposes a *duration* property which encodes the recommended transition duration in milliseconds. This duration is based on the path length of the curved trajectory through *x,y* space. If you want to a slower or faster transition, multiply this by an arbitrary scale factor (<i>V</i> as described in the original paper).

### Sampling

<a name="quantize" href="#quantize">#</a> d3.<b>quantize</b>(<i>interpolator</i>, <i>n</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/quantize.js "Source")

Returns *n* uniformly-spaced samples from the specified *interpolator*, where *n* is an integer greater than one. The first sample is always at *t* = 0, and the last sample is always at *t* = 1. This can be useful in generating a fixed number of samples from a given interpolator, such as to derive the range of a [quantize scale](https://github.com/d3/d3-scale#quantize-scales) from a [continuous interpolator](https://github.com/d3/d3-scale#interpolateWarm).

Caution: this method will not work with interpolators that do not return defensive copies of their output, such as [d3.interpolateArray](#interpolateArray), [d3.interpolateDate](#interpolateDate) and [d3.interpolateObject](#interpolateObject). For those interpolators, you must wrap the interpolator and create a copy for each returned value.

### Color Spaces

<a name="interpolateRgb" href="#interpolateRgb">#</a> d3.<b>interpolateRgb</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/rgb.png" width="100%" height="40" alt="rgb">

Or, with a corrected [gamma](#interpolate_gamma) of 2.2:

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/rgbGamma.png" width="100%" height="40" alt="rgbGamma">

Returns an RGB color space interpolator between the two colors *a* and *b* with a configurable [gamma](#interpolate_gamma). If the gamma is not specified, it defaults to 1.0. The colors *a* and *b* need not be in RGB; they will be converted to RGB using [d3.rgb](https://github.com/d3/d3-color#rgb). The return value of the interpolator is an RGB string.

<a href="#interpolateRgbBasis" name="interpolateRgbBasis">#</a> d3.<b>interpolateRgbBasis</b>(<i>colors</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js#L54 "Source")

Returns a uniform nonrational B-spline interpolator through the specified array of *colors*, which are converted to [RGB color space](https://github.com/d3/d3-color#rgb). Implicit control points are generated such that the interpolator returns *colors*[0] at *t* = 0 and *colors*[*colors*.length - 1] at *t* = 1. Opacity interpolation is not currently supported. See also [d3.interpolateBasis](#interpolateBasis), and see [d3-scale-chromatic](https://github.com/d3/d3-scale-chromatic) for examples.

<a href="#interpolateRgbBasisClosed" name="interpolateRgbBasisClosed">#</a> d3.<b>interpolateRgbBasisClosed</b>(<i>colors</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js#L55 "Source")

Returns a uniform nonrational B-spline interpolator through the specified array of *colors*, which are converted to [RGB color space](https://github.com/d3/d3-color#rgb). The control points are implicitly repeated such that the resulting spline has cyclical C² continuity when repeated around *t* in [0,1]; this is useful, for example, to create cyclical color scales. Opacity interpolation is not currently supported. See also [d3.interpolateBasisClosed](#interpolateBasisClosed), and see [d3-scale-chromatic](https://github.com/d3/d3-scale-chromatic) for examples.

<a name="interpolateHsl" href="#interpolateHsl">#</a> d3.<b>interpolateHsl</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hsl.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hsl.png" width="100%" height="40" alt="hsl">

Returns an HSL color space interpolator between the two colors *a* and *b*. The colors *a* and *b* need not be in HSL; they will be converted to HSL using [d3.hsl](https://github.com/d3/d3-color#hsl). If either color’s hue or saturation is NaN, the opposing color’s channel value is used. The shortest path between hues is used. The return value of the interpolator is an RGB string.

<a name="interpolateHslLong" href="#interpolateHslLong">#</a> d3.<b>interpolateHslLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hsl.js#L21 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hslLong.png" width="100%" height="40" alt="hslLong">

Like [interpolateHsl](#interpolateHsl), but does not use the shortest path between hues.

<a name="interpolateLab" href="#interpolateLab">#</a> d3.<b>interpolateLab</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/lab.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/lab.png" width="100%" height="40" alt="lab">

Returns a Lab color space interpolator between the two colors *a* and *b*. The colors *a* and *b* need not be in Lab; they will be converted to Lab using [d3.lab](https://github.com/d3/d3-color#lab). The return value of the interpolator is an RGB string.

<a name="interpolateHcl" href="#interpolateHcl">#</a> d3.<b>interpolateHcl</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hcl.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hcl.png" width="100%" height="40" alt="hcl">

Returns an HCL color space interpolator between the two colors *a* and *b*. The colors *a* and *b* need not be in HCL; they will be converted to HCL using [d3.hcl](https://github.com/d3/d3-color#hcl). If either color’s hue or chroma is NaN, the opposing color’s channel value is used. The shortest path between hues is used. The return value of the interpolator is an RGB string.

<a name="interpolateHclLong" href="#interpolateHclLong">#</a> d3.<b>interpolateHclLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hcl.js#L21 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hclLong.png" width="100%" height="40" alt="hclLong">

Like [interpolateHcl](#interpolateHcl), but does not use the shortest path between hues.

<a name="interpolateCubehelix" href="#interpolateCubehelix">#</a> d3.<b>interpolateCubehelix</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/cubehelix.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelix.png" width="100%" height="40" alt="cubehelix">

Or, with a [gamma](#interpolate_gamma) of 3.0 to emphasize high-intensity values:

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixGamma.png" width="100%" height="40" alt="cubehelixGamma">

Returns a Cubehelix color space interpolator between the two colors *a* and *b* using a configurable [gamma](#interpolate_gamma). If the gamma is not specified, it defaults to 1.0. The colors *a* and *b* need not be in Cubehelix; they will be converted to Cubehelix using [d3.cubehelix](https://github.com/d3/d3-color#cubehelix). If either color’s hue or saturation is NaN, the opposing color’s channel value is used. The shortest path between hues is used. The return value of the interpolator is an RGB string.

<a name="interpolateCubehelixLong" href="#interpolateCubehelixLong">#</a> d3.<b>interpolateCubehelixLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/cubehelix.js#L29 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixLong.png" width="100%" height="40" alt="cubehelixLong">

Or, with a [gamma](#interpolate_gamma) of 3.0 to emphasize high-intensity values:

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixGammaLong.png" width="100%" height="40" alt="cubehelixGammaLong">

Like [interpolateCubehelix](#interpolateCubehelix), but does not use the shortest path between hues.

<a name="interpolate_gamma" href="#interpolate_gamma">#</a> <i>interpolate</i>.<b>gamma</b>(<i>gamma</i>)

Given that *interpolate* is one of [interpolateRgb](#interpolateRgb), [interpolateCubehelix](#interpolateCubehelix) or [interpolateCubehelixLong](#interpolateCubehelixLong), returns a new interpolator factory of the same type using the specified *gamma*. For example, to interpolate from purple to orange with a gamma of 2.2 in RGB space:

```js
var interpolator = d3.interpolateRgb.gamma(2.2)("purple", "orange");
```

See Eric Brasseur’s article, [Gamma error in picture scaling](https://web.archive.org/web/20160112115812/http://www.4p8.com/eric.brasseur/gamma.html), for more on gamma correction.

### Splines

Whereas standard interpolators blend from a starting value *a* at *t* = 0 to an ending value *b* at *t* = 1, spline interpolators smoothly blend multiple input values for *t* in [0,1] using piecewise polynomial functions. Only cubic uniform nonrational [B-splines](https://en.wikipedia.org/wiki/B-spline) are currently supported, also known as basis splines.

<a href="#interpolateBasis" name="interpolateBasis">#</a> d3.<b>interpolateBasis</b>(<i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/basis.js "Source")

Returns a uniform nonrational B-spline interpolator through the specified array of *values*, which must be numbers. Implicit control points are generated such that the interpolator returns *values*[0] at *t* = 0 and *values*[*values*.length - 1] at *t* = 1. See also [d3.curveBasis](https://github.com/d3/d3-shape#curveBasis).

<a href="#interpolateBasisClosed" name="interpolateBasisClosed">#</a> d3.<b>interpolateBasisClosed</b>(<i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/basisClosed.js "Source")

Returns a uniform nonrational B-spline interpolator through the specified array of *values*, which must be numbers. The control points are implicitly repeated such that the resulting one-dimensional spline has cyclical C² continuity when repeated around *t* in [0,1]. See also [d3.curveBasisClosed](https://github.com/d3/d3-shape#curveBasisClosed).

### Piecewise

<a name="piecewise" href="#piecewise">#</a> d3.<b>piecewise</b>(<i>interpolate</i>, <i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/piecewise.js "Source")

Returns a piecewise interpolator, composing interpolators for each adjacent pair of *values*. The returned interpolator maps *t* in [0, 1 / (*n* - 1)] to *interpolate*(*values*[0], *values*[1]), *t* in [1 / (*n* - 1), 2 / (*n* - 1)] to *interpolate*(*values*[1], *values*[2]), and so on, where *n* = *values*.length. In effect, this is a lightweight [linear scale](https://github.com/d3/d3-scale/blob/master/README.md#linear-scales). For example, to blend through red, green and blue:

```js
var interpolate = d3.piecewise(d3.interpolateRgb.gamma(2.2), ["red", "green", "blue"]);
```
