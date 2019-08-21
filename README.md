# d3-contour

这个库将 [marching squares](https://en.wikipedia.org/wiki/Marching_squares) 算法应用到数值的矩形数组来计算等值线多边形. 例如, 这是 `Maungawhau` 的拓扑结构(经典的 `volcano` 数据集和来自 `R` 语言的 `terrain.colors`):

[<img alt="Contour Plot" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/volcano.gif" width="420" height="295">](https://bl.ocks.org/mbostock/4241134)

对于每个 [threshold value(阈值)](#contours_thresholds), [contour generator(等值线生成器)](#_contours) 构造一个 `GeoJSON` 多边形几何对象来表示输入值大于或等于阈值的区域. 几何是平面坐标, 其中 ⟨<i>i</i> + 0.5, <i>j</i> + 0.5⟩ 对应于输入值数组中的元素 <i>i</i> + <i>jn</i>. 下面是一个加载地球表面温度数据 `GeoTIFF` 例子, 另一个模糊了嘈杂的单色 `PNG`, 以生成平滑的云量轮廓:

[<img alt="GeoTiff Contours" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/temperature.png" width="420" height="219">](https://bl.ocks.org/mbostock/4886c227038510f1c103ce305bef6fcc)
[<img alt="Cloud Contours" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/clouds.png" width="420" height="219">](https://bl.ocks.org/mbostock/818053c76d79d4841790c332656bf9da)

因为等值线多边形是 `GeoJSON`, 你可以使用标准工具去转换和显示它们; 参考 [d3.geoPath](https://github.com/d3/d3-geo/blob/master/README.md#geoPath), [d3.geoProject](https://github.com/d3/d3-geo-projection/blob/master/README.md#geoProject) 和 [d3.geoStitch](https://github.com/d3/d3-geo-projection/blob/master/README.md#geoStitch), 例如. 上面的地表温度等值线显示在自然地球投影中:

[<img alt="GeoTiff Contours II" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/reprojection.png" width="420" height="219">](https://bl.ocks.org/mbostock/83c0be21dba7602ee14982b020b12f51)

等值线图也可以通过采样实现连续函数的可视化. 下面是 `Goldstein–Price` 函数(一个用于全局优化的测试函数)和 `sin(x + y)sin(x - y)` 的动画:

[<img alt="Contour Plot II" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/goldstein-price.png" width="420" height="219">](https://bl.ocks.org/mbostock/f48ff9c1af4d637c9a518727f5fdfef5)
[<img alt="Contour Plot III" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/sin-cos.png" width="420" height="219">](https://bl.ocks.org/mbostock/bf2f5f02b62b5b3bb92ae1b59b53da36)

等值线还可以显示点云的[estimated density(估计密度)](#density-estimation), 对于避免在大数据集中重叠绘制特别有用. 这个库实现了快速的二维核密度估计; 参考 [d3.contourDensity](#contourDensity). 下面是一张散点图, 显示了 `Old Faithful(老忠实泉, 位于黄石公园)` 的停顿时间和喷发时间之间的关系:

[<img alt="Density Contours" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/faithful.png" width="420" height="219">](https://bl.ocks.org/mbostock/e3f4376d54e02d5d43ae32a7cf0e6aa9)

下面是一张密度等高值图, 统计了 `53,940` 颗钻石的重量和价格之间的关系:

[<img alt="Density Contours" src="https://raw.githubusercontent.com/d3/d3-contour/master/img/diamonds.png" width="420" height="420">](https://bl.ocks.org/mbostock/7f5f22524bd1d824dd53c535eda0187f)

## Installing

`NPM` 安装: `npm install d3-contour`. 此外还可以下载 [最新发行版](https://github.com/d3/d3-contour/releases/latest). 可以直接从 [d3js.org](https://d3js.org) 以 [标准独立库](https://d3js.org/d3-contour.v1.min.js) 或作为 [D3](https://github.com/d3/d3) 的一部分直接引入. 支持 `AMD`, `CommonJS` 以及基础的标签引入形式. 如果使用标签引入, 则会暴露全局 `d3`:

```html
<script src="https://d3js.org/d3-contour.v1.min.js"></script>
<script>

// Populate a grid of n×m values where -2 ≤ x ≤ 2 and -2 ≤ y ≤ 1.
var n = 256, m = 256, values = new Array(n * m);
for (var j = 0.5, k = 0; j < m; ++j) {
  for (var i = 0.5; i < n; ++i, ++k) {
    values[k] = goldsteinPrice(i / n * 4 - 2, 1 - j / m * 3);
  }
}

// Compute the contour polygons at log-spaced intervals; returns an array of MultiPolygon.
var contours = d3.contours()
    .size([n, m])
    .thresholds(d3.range(2, 21).map(p => Math.pow(2, p)))
    (values);

// See https://en.wikipedia.org/wiki/Test_functions_for_optimization
function goldsteinPrice(x, y) {
  return (1 + Math.pow(x + y + 1, 2) * (19 - 14 * x + 3 * x * x - 14 * y + 6 * x * x + 3 * y * y))
      * (30 + Math.pow(2 * x - 3 * y, 2) * (18 - 32 * x + 12 * x * x + 48 * y - 36 * x * y + 27 * y * y));
}

</script>
```

[在浏览器中测试 `d3-contour`](https://tonicdev.com/npm/d3-contour)

## API Reference

<a name="contours" href="#contours">#</a> d3.<b>contours</b>() [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

使用默认的设置构建一个新的等值线生成器.

<a name="_contours" href="#_contours">#</a> <i>contours</i>(<i>values</i>) [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

计算给定 *values* 数组的等值线, 返回 [GeoJSON](http://geojson.org/geojson-spec.html) [MultiPolygon](http://geojson.org/geojson-spec.html#multipolygon) [几何对象](http://geojson.org/geojson-spec.html#geometry-objects) 数组. 每个几何对象代表一个区域, 这个区域中的 <i>values</i> 大于等于对应的 [阈值](#contours_thresholds); 每个几何对象对应的阈值以 <i>geometry</i>.value 的形式暴露.

输入的 *values* 必须以长度为 <i>n</i>×<i>m</i> 的数组形式给出, 其中 [<i>n</i>, <i>m</i>] 为等值线生成器的 [size](#contours_size); 此外, 每个 <i>values</i>[<i>i</i> + <i>jn</i>] 必须表示位于 ⟨<i>i</i>, <i>j</i>⟩ 处的值. 例如, 构造一个 `256 x 256` 的网格的 [Goldstein–Price](https://en.wikipedia.org/wiki/Test_functions_for_optimization) 函数, 其中 -2 ≤ <i>x</i> ≤ 2 and -2 ≤ <i>y</i> ≤ 1:

```js
var n = 256, m = 256, values = new Array(n * m);
for (var j = 0.5, k = 0; j < m; ++j) {
  for (var i = 0.5; i < n; ++i, ++k) {
    values[k] = goldsteinPrice(i / n * 4 - 2, 1 - j / m * 3);
  }
}

function goldsteinPrice(x, y) {
  return (1 + Math.pow(x + y + 1, 2) * (19 - 14 * x + 3 * x * x - 14 * y + 6 * x * x + 3 * y * y))
      * (30 + Math.pow(2 * x - 3 * y, 2) * (18 - 32 * x + 12 * x * x + 48 * y - 36 * x * y + 27 * y * y));
}
```

返回的几何对象通常被传递给 [d3.geoPath](https://github.com/d3/d3-geo/blob/master/README.md#geoPath) 去显示, 不使用或者使用 [d3.geoIdentity](https://github.com/d3/d3-geo/blob/master/README.md#geoIdentity) 作为相关投影.

<a name="contours_contour" href="#contours_contour">#</a> <i>contours</i>.<b>contour</b>(<i>values</i>, <i>threshold</i>) [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

计算一个单个的等值线, 返回一个 [GeoJSON](http://geojson.org/geojson-spec.html) [MultiPolygon](http://geojson.org/geojson-spec.html#multipolygon) [几何对象](http://geojson.org/geojson-spec.html#geometry-objects) 表示输入 <i>values</i> 大于或等于给定的 [*阈值*](#contours_thresholds); 阈值以 <i>geometry</i>.value 的形式暴露.

输入的 *values* 必须以长度为 <i>n</i>×<i>m</i> 的数组形式给出, 其中 [<i>n</i>, <i>m</i>] 为等值线生成器的 [size](#contours_size); 此外, 每个 <i>values</i>[<i>i</i> + <i>jn</i>] 必须表示位于 ⟨<i>i</i>, <i>j</i>⟩ 处的值. 参考 [*contours*](#_contours) 的示例.

<a name="contours_size" href="#contours_size">#</a> <i>contours</i>.<b>size</b>([<i>size</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

如果指定了 *size* 则将当前输入 *values* 的期望值设置为指定的大小并返回等值线生成器. *size* 以 [<i>n</i>, <i>m</i>\] 的形式指定, 其中 <i>n</i> 为网格的列数, <i>m</i> 为行数; *n* 和 *m* 必须为正整数. 如果没有指定 *size* 则返回当前默认大小, 默认为 [1, 1].

<a name="contours_smooth" href="#contours_smooth">#</a> <i>contours</i>.<b>smooth</b>([<i>smooth</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

如果指定了 *smooth* 则设置是否在生成等值线时使用线性插值. 如果没有指定 *smooth* 则返回当前的平滑标识, 默认为 `true`.

<a name="contours_thresholds" href="#contours_thresholds">#</a> <i>contours</i>.<b>thresholds</b>([<i>thresholds</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/contours.js "Source")

如果指定了 *thresholds* 则将阈值生成器设置为指定的函数或数组并返回当前等值线生成器. 如果没有指定 *thresholds* 则返回当前阈值生成器, 默认情况下阈值生成器的实现方法为 [Sturges’ formula](https://github.com/d3/d3-array/blob/master/README.md#thresholdSturges).

阈值以 [*x0*, *x1*, …] 数组的形式定义. 第一个 [等值线](#_contour) 表示输入值大于或等于 *x0* 的部分, 第二个表示输入值大于或等于 *x1* 的部分, 以此类推. 这样, 对于每个指定的阈值, 都对应一个生成的 `MultiPolygon` 几何对象, 阈值以 <i>geometry</i>.value 的形式暴露.

如果使用 *count* 代表阈值数组, 则输入值的 [extent(范围)](https://github.com/d3/d3-array/blob/master/README.md#extent) 会被均匀的分为 *count* 个分箱; 可以参考 [d3.ticks](https://github.com/d3/d3-array/blob/master/README.md#ticks).

## Density Estimation

<a name="contourDensity" href="#contourDensity">#</a> d3.<b>contourDensity</b>() [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

使用默认的设置构造一个新的密度估计函数.

<a name="_density" href="#_density">#</a> <i>density</i>(<i>data</i>) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

估计给定数据数组的密度轮廓, 返回 [GeoJSON](http://geojson.org/geojson-spec.html) [MultiPolygon](http://geojson.org/geojson-spec.html#multipolygon) [几何对象](http://geojson.org/geojson-spec.html#geometry-objects) 数组. 每个几何对象表示每平方像素估计点数大于或等于相应阈值的区域; 每个几何对象对应的阈值以 <i>geometry</i>.value 的形式暴露. 返回的几何对象通常被传递给 [d3.geoPath](https://github.com/d3/d3-geo/blob/master/README.md#geoPath) 去显示, 不使用或者使用 [d3.geoIdentity](https://github.com/d3/d3-geo/blob/master/README.md#geoIdentity) 作为相关投影. 参考 [d3.contours](#contours).

使用 [*density*.x](#density_x) 和 [*density*.y](#density_y) 计算每个数据点的 *x*- 和 *y*- 坐标. 此外, [*density*.weight](#density_weight) 表示每个数据点的权重(默认为 1). 生成的等值线轮廓只在估计函数的 [defined size](#density_size) 范围内是精确的.

<a name="density_x" href="#density_x">#</a> <i>density</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *x* 则设置 *x*-坐标访问器. 如果没有指定 *x* 则返回当前的 *x*-坐标访问器, 默认为:

```js
function x(d) {
  return d[0];
}
```

<a name="density_y" href="#density_y">#</a> <i>density</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *y* 则设置 *y*-坐标访问器. 如果没有指定 *y* 则返回当前的 *y*-坐标访问器, 默认为:

```js
function y(d) {
  return d[1];
}
```

<a name="density_weight" href="#density_weight">#</a> <i>density</i>.<b>weight</b>([<i>weight</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *weight* 则设置点的权重访问器. 如果没有指定 *weight* 则返回默认的点的权重访问器, 默认为:

```js
function weight() {
  return 1;
}
```

<a name="density_size" href="#density_size">#</a> <i>density</i>.<b>size</b>([<i>size</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *size* 则将当前密度估计函数的大小设置为指定的界限并返回估计函数. *size* 以 [<i>width</i>, <i>height</i>\] 的形式指定, 其中 <i>width</i> 为最大 *x*-值, <i>height</i> 为最大 *y*-值. 如果没有指定 *size* 则返回当前默认大小, 默认为 [960, 500]. [estimated density contours(估计的密度轮廓)](#_density) 只在定义的大小范围内准确.

<a name="density_cellSize" href="#density_cellSize">#</a> <i>density</i>.<b>cellSize</b>([<i>cellSize</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *cellSize* (单元格大小), 则将底层分箱网格中单个单元格的大小设置为指定的正整数并返回估计函数. 如果没有指定 *cellSize* 则返回当前的单元格大小, 默认为 `4`. 单元格大小四舍五入到最近的 `2` 的幂次方. 单元格越小轮廓越精细, 但是计算代价也越大.

<a name="density_thresholds" href="#density_thresholds">#</a> <i>density</i>.<b>thresholds</b>([<i>thresholds</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *thresholds* 则将阈值生成器设置为指定的函数或数组并返回等值线生成器. 如果 *thresholds* 没有指定则返回当前的阈值生成器, 默认会生成 `20` 个四舍五入的密度阈值.

阈值以 [*x0*, *x1*, …] 数组的形式定义. 第一个 [密度等值线](#_density) 表示估计值大于或等于 *x0* 的部分, 第二个表示估计值大于或等于 *x1* 的部分, 以此类推. 这样, 对于每个指定的阈值, 都对应一个生成的 `MultiPolygon` 几何对象, 阈值以 <i>geometry</i>.value 的形式暴露. 第一个阈值应该大于 `0`.

如果使用 *count* 代表阈值数组, 则会近似生成 *count* 个阈值; 可以参考 [d3.ticks](https://github.com/d3/d3-array/blob/master/README.md#ticks).

<a name="density_bandwidth" href="#density_bandwidth">#</a> <i>density</i>.<b>bandwidth</b>([<i>bandwidth</i>]) [<>](https://github.com/d3/d3-contour/blob/master/src/density.js "Source")

如果指定了 *bandwidth*, 则设置 `Gaussian` 核密度估计的宽带(标准差)并返回估计函数. 如果 *bandwidth* 没有指定则返回当前的带宽, 默认为 `20.4939…`, 这个实现目前将指定的带宽四舍五入到最近的支持值, 并且必须是非负的.
