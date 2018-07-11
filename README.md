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

返回一个在两个字符串 *a* 和 *b* 之间插值的插值器。字符串插值器会分别在字符串 *a* 和 *b* 之间找出内嵌的数字，其中每个数字都是可以被 `JavaScript` 识别的。在字符串中可以被检测到的数字示例: `-1`, `42`, `3.14159` 和 `6.0221413e+23`.

对于每个嵌入到 *b* 中的数字，插值器都会尝试在 *a* 中找出相对应的数字。如果能找到则对两者使用 [interpolateNumber](#interpolateNumber)。字符串 *b* 的其他部分被当做模板: 字符串 *b* 的静态部分会保持不变，而数字部分会进行插值计算。

例如，如果 *a* 为 `"300 12px sans-serif"`, 并且 *b* 为 `"500 36px Comic-Sans"`, 则会找出两个内嵌的数值. 剩余的静态部分 (对于字符串 *b*) 是两个数值之间的空格 (`" "`), 以及后缀 (`"px Comic-Sans"`). 在 *t* = 0.5 时返回的结果为 `"400 24px Comic-Sans"`.

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

返回一个在 2D `CSS` 变换 *a* 和 *b* 之间插值的插值器. 每个变换都会被分解为 `translate`, `rotate` *x*-skew 以及 `scale`; 这些组成变换的部分会被分别插值。这个行为是一个标准的 `CSS` 动画: 参考 [matrix decomposition for animation](http://www.w3.org/TR/css3-2d-transforms/#matrix-decomposition).

<a name="interpolateTransformSvg" href="#interpolateTransformSvg">#</a> d3.<b>interpolateTransformSvg</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/transform/index.js#L63 "Source")

返回一个在 2D `SVG` 变换 *a* 和 *b* 之间插值的插值器. 每个变换都会被分解为 `translate`, `rotate` *x*-skew 以及 `scale`; 这些组成变换的部分会被分别插值。这个行为是一个标准的 `CSS` 动画: 参考 [matrix decomposition for animation](http://www.w3.org/TR/css3-2d-transforms/#matrix-decomposition)

<a name="interpolateZoom" href="#interpolateZoom">#</a> d3.<b>interpolateZoom</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/zoom.js "Source")

返回一个在二维平面两个视图 *a* 和 *b* 之间的插值器，基于 `Jarke J. van Wijk` 和 `Wim A.A. Nuij` 的 [“Smooth and efficient zooming and panning”](http://www.win.tue.nl/~vanwijk/zoompan.pdf). 每个视图被定义为三个元素的数组: *cx*, *cy* and *width*。其中 *cx*, *cy* 表示视图的中心，*width* 表示视图的大小。

返回的插值器暴露一个 *duration* 属性用来表示建议的过渡时间(毫秒)。过渡时间基于 `x`, `y` 空间的曲线轨迹的路径长度得到的。如果你想更快或更慢的过渡，可以将它乘以一个比例系数。

### Sampling

<a name="quantize" href="#quantize">#</a> d3.<b>quantize</b>(<i>interpolator</i>, <i>n</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/quantize.js "Source")

根据指定的 *interpolator* 返回 *n* 个等间隔的均匀采样, 其中 *n* 是一个大于 `1` 的整数。第一个采样点总是取 *t* = 0 时的值而最后一个值总是取 *t* = 1 处的值。这个方法可以从给定的插值器中取固定数量的等间隔的值, 比如从 [continuous interpolator](https://github.com/d3/d3-scale#interpolateWarm) 中推导 [quantize scale](https://github.com/d3/d3-scale#quantize-scales)。

警告: 这种方法不适用于不返回副本的插值器, 比如 [d3.interpolateArray](#interpolateArray), [d3.interpolateDate](#interpolateDate) 和 [d3.interpolateObject](#interpolateObject)。对于这些插值器，你必须对其包装，每次插值时返回一个副本。

### Color Spaces

<a name="interpolateRgb" href="#interpolateRgb">#</a> d3.<b>interpolateRgb</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/rgb.png" width="100%" height="40" alt="rgb">

或者, 使用 `2.2` 修正的 [gamma](#interpolate_gamma):

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/rgbGamma.png" width="100%" height="40" alt="rgbGamma">

返回一个在两个颜色 *a* 和 *b* 之间插值的 `RGB` 颜色空间插值器, 其中包含可配置参数 [gamma](#interpolate_gamma)。如果没有指定 `gamma` 则默认为 `1.0`。颜色 *a* 和 *b* 不需要非要是 `RGB` 空间；可以使用 [d3.rgb](https://github.com/xswei/d3-color#rgb) 转为 `RGB` 颜色即可。插值器返回的结果使用 `RGB` 字符串表示.

Returns an RGB color space interpolator between the two colors *a* and *b* with a configurable [gamma](#interpolate_gamma). If the gamma is not specified, it defaults to 1.0. The colors *a* and *b* need not be in RGB; they will be converted to RGB using [d3.rgb](https://github.com/d3/d3-color#rgb). The return value of the interpolator is an RGB string.

<a href="#interpolateRgbBasis" name="interpolateRgbBasis">#</a> d3.<b>interpolateRgbBasis</b>(<i>colors</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js#L54 "Source")

根据指定的颜色数组返回一个 `uniform nonrational` B-样条插值器, 这些颜色都会被转为 [RGB color space](https://github.com/d3/d3-color#rgb)。隐式控制点的生成会使得 *t* = 0 时返回 *colors*[0] 并且在 *t* = 1 时返回 [*colors*.length - 1]。目前支持不透明度的插值.参考 [d3.interpolateBasis](#interpolateBasis) 和 [d3-scale-chromatic](https://github.com/xswei/d3-scale-chromatic) 获取更多例子。

<a href="#interpolateRgbBasisClosed" name="interpolateRgbBasisClosed">#</a> d3.<b>interpolateRgbBasisClosed</b>(<i>colors</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/rgb.js#L55 "Source")

根据指定的颜色数组返回一个 `uniform nonrational` B-样条插值器, 这些颜色都会被转为 [RGB color space](https://github.com/d3/d3-color#rgb)。隐式的控制点是重复的，这样在 *t* 处于 [0, 1] 时返回的结果是循环重复的。创建一个循环颜色比例尺时是有用的。目前支持不透明度的插值.参考 [d3.interpolateBasis](#interpolateBasis) 和 [d3-scale-chromatic](https://github.com/xswei/d3-scale-chromatic) 获取更多例子。

<a name="interpolateHsl" href="#interpolateHsl">#</a> d3.<b>interpolateHsl</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hsl.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hsl.png" width="100%" height="40" alt="hsl">

在两个颜色 *a* 和 *b* 之间创建一个 `HSL` 颜色空间的插值器。*a* 和 *b* 不一定使用 `HSL` 表示，它们将会适用 [d3.hsl](https://github.com/xswei/d3-color#hsl) 转为 `HSL` 表示. 如果其中一个颜色的 `hue` 或 `saturation` 为 `NaN` 则使用相反的颜色通道值。最短路径的 `hues` 将会被使用。返回的值使用 `RGB` 字符串表示.

<a name="interpolateHslLong" href="#interpolateHslLong">#</a> d3.<b>interpolateHslLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hsl.js#L21 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hslLong.png" width="100%" height="40" alt="hslLong">

与 [interpolateHsl](#interpolateHsl) 类似, 但是不使用 `hues` 之间的最短路径.

<a name="interpolateLab" href="#interpolateLab">#</a> d3.<b>interpolateLab</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/lab.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/lab.png" width="100%" height="40" alt="lab">

在两个颜色 *a* 和 *b* 之间创建一个 `Lab` 颜色空间的插值器。*a* 和 *b* 不一定使用 `Lab` 表示，它们将会适用 [d3.lab](https://github.com/xswei/d3-color#lab) 转为 `Lab` 表示。返回的值使用 `RGB` 字符串表示.

<a name="interpolateHcl" href="#interpolateHcl">#</a> d3.<b>interpolateHcl</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hcl.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hcl.png" width="100%" height="40" alt="hcl">

在两个颜色 *a* 和 *b* 之间创建一个 `HCL` 颜色空间的插值器。*a* 和 *b* 不一定使用 `HCL` 表示，它们将会适用 [d3.hcl](https://github.com/xswei/d3-color#hcl) 转为 `HCL` 表示. 如果其中一个颜色的 `hue` 或 `chroma` 为 `NaN` 则使用相反的颜色通道值。最短路径的 `hues` 将会被使用。返回的值使用 `RGB` 字符串表示.

<a name="interpolateHclLong" href="#interpolateHclLong">#</a> d3.<b>interpolateHclLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/hcl.js#L21 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/hclLong.png" width="100%" height="40" alt="hclLong">

与 [interpolateHcl](#interpolateHcl) 类似, 但是不使用 `hues` 之间的最短路径.

<a name="interpolateCubehelix" href="#interpolateCubehelix">#</a> d3.<b>interpolateCubehelix</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/cubehelix.js "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelix.png" width="100%" height="40" alt="cubehelix">

或者使用 [gamma](#interpolate_gamma) = 3.0 来强调高亮度的值:

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixGamma.png" width="100%" height="40" alt="cubehelixGamma">

返回一个在两个颜色 *a* 和 *b* 之间可配置 [gamma](#interpolate_gamma) 的 `Cubehelix` 颜色空间插值器。如果没有指定 `gamma` 则默认为 1.0. 颜色值 *a* 和*b* 不一定必须为 `Cubehelix`；在内部会使用 [d3.cubehelix](https://github.com/xswei/d3-color#cubehelix) 将其转换为 `Cubehelix` 表示。如果其中一个颜色值得 `hue` 或 `saturation` 为 `NaN`, 则相反的颜色通道值会使用。会在两个颜色值之间最短的 `hue` 路径之间插值，返回值是一个 `RGB` 字符串。

<a name="interpolateCubehelixLong" href="#interpolateCubehelixLong">#</a> d3.<b>interpolateCubehelixLong</b>(<i>a</i>, <i>b</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/cubehelix.js#L29 "Source")

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixLong.png" width="100%" height="40" alt="cubehelixLong">

或者使用 [gamma](#interpolate_gamma) = 3.0 来强调高亮度的值:

<img src="https://raw.githubusercontent.com/d3/d3-interpolate/master/img/cubehelixGammaLong.png" width="100%" height="40" alt="cubehelixGammaLong">

与 [interpolateCubehelix](#interpolateCubehelix) 类似, 但是不是使用两个 `hues` 值之间的最短路径.

<a name="interpolate_gamma" href="#interpolate_gamma">#</a> <i>interpolate</i>.<b>gamma</b>(<i>gamma</i>)

给定的 *interpolate* 为 [interpolateRgb](#interpolateRgb), [interpolateCubehelix](#interpolateCubehelix) 或 [interpolateCubehelixLong](#interpolateCubehelixLong) 中的一种, 使用指定的 *gamma* 值返回一个新的同类型的插值器. 例如使用 *gamma* 为 2.2 且在 `purple` 和 `orange` 之间进行 `RGB` 颜色插值的插值器:

```js
var interpolator = d3.interpolateRgb.gamma(2.2)("purple", "orange");
```

参考 `Eric Brasseur` 的文章 [Gamma error in picture scaling](https://web.archive.org/web/20160112115812/http://www.4p8.com/eric.brasseur/gamma.html) 获取更多关于 *gamma* 修正的资料.

### Splines

标准的插值器会在起始值 *t* = 0 时的 *a* 和 *t* = 1 时的 *b* 之间计算得出一个适当的值。而样条曲线插值器可以使用分段多项式函数为多个介于 [0,1] 之间的输入值进行光滑插值。目前只支持三次无理 [B-splines](https://en.wikipedia.org/wiki/B-spline)，也就是基本样条曲线。

<a href="#interpolateBasis" name="interpolateBasis">#</a> d3.<b>interpolateBasis</b>(<i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/basis.js "Source")

根据给定的一组 *values* 返回一个 `B-splibe` 插值器, 输入值必须为数值类型，隐式控制点的生成会使得在 *t* = 0 时返回 *values*[0] 并且 *t* = 1 时返回 *values*[*values*.length - 1], 参考 [d3.curveBasis](https://github.com/d3/d3-shape#curveBasis)。

<a href="#interpolateBasisClosed" name="interpolateBasisClosed">#</a> d3.<b>interpolateBasisClosed</b>(<i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/basisClosed.js "Source")

根据给定的一组 *values* 返回一个 `B-splibe` 插值器, 输入值必须为数值类型, 控制点是隐式重复的，这样得到的一维样条在 [0, 1] 中重复出现时会保持周期连贯性, 参考 [d3.curveBasisClosed](https://github.com/d3/d3-shape#curveBasisClosed)。

### Piecewise

<a name="piecewise" href="#piecewise">#</a> d3.<b>piecewise</b>(<i>interpolate</i>, <i>values</i>) [<>](https://github.com/d3/d3-interpolate/blob/master/src/piecewise.js "Source")

根据指定的 *values* 数组返回一个分段的插值器, 其中数组中每两个相邻的值之间会创建一个单独的插值器。返回的插值器在 *t* 处于[0, 1 / (*n* - 1)] 时通过 *interpolate*(*values*[0], *values*[1]) 事件, 在 *t* 处于 [1 / (*n* - 1), 2 / (*n* - 1)] 时使用 *interpolate*(*values*[1], *values*[2]) 计算，其中 *n* 等于 *values*.length，以此类推。从结果来看，这是一个轻量级的 [linear scale](https://github.com/d3/d3-scale/blob/master/README.md#linear-scales)。例如创建一个在 `red`, `green` 和 `blue` 之间创建一个插值器:

```js
var interpolate = d3.piecewise(d3.interpolateRgb.gamma(2.2), ["red", "green", "blue"]);
```
