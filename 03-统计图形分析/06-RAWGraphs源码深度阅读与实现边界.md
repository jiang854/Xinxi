# 06 RAWGraphs 源码深度阅读与实现边界

## 1. 研究对象与证据范围

- 仓库：[rawgraphs/rawgraphs-charts](https://github.com/rawgraphs/rawgraphs-charts)
- 阅读分支：`master`
- `package.json` 版本：`v1.1.0`
- 本次读取的 Git tree SHA：`cade3b6e1b4d35d365440b1365d51b0dd48c9bce`
- 核心入口：`src/index.js`
- 主要证据文件：每个图表目录中的 `metadata.js`、`dimensions.js`、`mapping.js`、`render.js`、`visualOptions.js`；以及 `package.json`、`CHANGELOG.md`、`example/configurations`、`example/datasets`。

本次核对到：
- `src/index.js` 导出 32 个模块：31 个正式图表模块 + 1 个 Color test 模块。
- `src/empty_chart` 和 `src/empty_chart_series` 是开发新图表的脚手架，不计入正式图表。
- 示例目录包含 39 个配置文件和 30 个数据文件。
- 项目 README 明确说明：此仓库只收录 RAWGraphs 公共应用选用的图表，是经过筛选的集合，不是所有可能图表的全集；项目也提示图表仍处于 beta 阶段，可能发生破坏性变更。

因此，本文件分析的是**特定版本的软件实现**，不是对所有统计学教材、软件或图表传统的最终定义。

## 2. 从源码看一张图表如何工作

一个图表目录通常由以下文件组成：

| 文件 | 源码职责 | 对应研究问题 |
|---|---|---|
| `metadata.js` | 图表名称、ID、介绍、应用内分类标签、教程和图标 | 软件如何介绍和检索该图表？ |
| `dimensions.js` | 可映射字段、字段类型、必选项、多字段映射、聚合规则 | 输入数据需要什么结构？ |
| `mapping.js` | 直接取值、分组、汇总、字段展开和派生对象 | 原始记录如何转成图表数据？ |
| `render.js` | 比例尺、统计变换、布局算法、SVG 几何对象 | 数据怎样变成图形？ |
| `visualOptions.js` | 排序、尺度、颜色、间距、标签、布局等可调参数 | 哪些视觉参数可以改变？ |
| `*.svg` / `*_thumb.svg` | 图标和缩略图 | 软件界面如何呈现图表入口？ |

### 2.1 数据输入层：dimensions.js

字段定义中的常见属性：

- `id`：程序内部使用的维度名称。
- `name`：界面中显示的字段名称。
- `validTypes`：接受 number、string、date 中的哪些数据类型。
- `required`：是否必填；脚手架注释说明默认不是必填。
- `multiple`：是否允许一个维度映射多个原始字段。
- `aggregation`：是否允许聚合。
- `aggregationDefault`：不同字段类型对应的默认聚合函数。
- `operation`：常见为 `get`，表示直接读取字段值。

脚手架注释列出的聚合选项为 `csv`、`csvDistinct`、`sum`、`mean`、`max`、`min`。实际某个图表能否使用某种聚合，仍需查看该图表自己的维度定义和核心 API 行为。

### 2.2 数据映射层：mapping.js

源码主要有两种路径：

1. **直接映射**：如部分散点、平行坐标、甘特图和 Voronoi 图，输入记录中的字段直接成为绘图对象的属性。
2. **分组与聚合**：使用 D3 `rollup` / `rollups`，按类别、系列、时间点、层级路径或 Source–Target 关系分组，再调用聚合函数。

同样的图形外观不意味着数据处理相同。例如：
- 条形图可将记录按类别分组后计算数量或数值总和。
- 蜂群图通常保留每条记录为一个点。
- 箱线图保留组内观测值，渲染时再计算分位数。
- 网络图先把记录变成关系边，再计算网络布局。
- 层级图先生成嵌套分组结构，再计算树或面积布局。

### 2.3 绘制层：render.js

渲染层既包含统计变换，也包含纯布局运算。常见比例尺和图形生成方法：

- `scaleLinear`、`scaleTime`：连续数值或时间映射到坐标。
- `scaleBand`、`scalePoint`：类别映射到离散位置。
- `scaleSqrt`：常用于圆形半径，使圆面积而非半径与数值近似成比例。
- `line`、`area`、`arc`：生成折线、面积和弧形路径。
- `stack`：把多个系列变成堆叠区间。
- `hierarchy`、`tree`、`cluster`、`pack`、`treemap`、`partition`：把层级数据转换为空间结构。
- `Delaunay` / `voronoi`、`polygonHull`：计算几何邻域和点集包络。
- `forceSimulation`、`forceCollide`：用于点或标签的空间避让。

**必须区分统计计算和布局算法。** 例如，Voronoi 单元、树形分支坐标、蜂群点的避让位移并不自动代表新的统计变量。

## 3. 31 种图表的源码实现要点

以下按图表主要分析任务组织。分组是便于研究的工作分类，不是 RAWGraphs 的唯一官方分类。

### A. 类别比较、数量和构成

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Bar chart 条形图 | 类别字段；可选数值 size、color、series | 按 series 和类别分组；映射 size 时聚合，未映射时以组内记录数 `v.length` 为值 | 类别位置 + 条形长度；可横向/纵向。长度表达数值，不能把 metadata 中的 width 一词理解为只能用宽度编码 |
| Multi-set bar chart 多组条形图 | 一个类别组字段 + 多个数值字段；可选 series | 对每个数值字段分别应用聚合函数，生成并列条形 | 组内并列比较多个指标；不同数值字段必须有可比单位或清楚说明尺度 |
| Stacked bar chart 堆叠条形图 | 一个类别字段 + 多个数值字段；可选 series | 每个数值字段独立聚合，再进行 stack 变换 | 堆叠段表达组成，总长度表达总量；中间段不共享统一基线，精确比较不如并列条形 |
| Pie chart 饼图/环形图 | 多个数值字段 arcs；可选 series | 每个 series 形成一个饼；`d3.pie().sort(null)` 按映射字段顺序计算扇区角度；总值用于计算饼的半径 | 扇区角度表达组成比例；饼的整体半径也随总值变化。比较不同饼时要同时注意角度和整体面积，不应只看单个扇区大小 |
| Matrix plot 矩阵图 | X、Y 两个类别字段；可选 size、color、label | 按 X/Y 组合分组；size 映射数值聚合，未映射时以记录数作为默认 size | 格子位置表达两个分类维度；圆/方形大小或颜色编码数值。格子中的数值口径必须写清楚 |

### B. 时间变化、排名和区间

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Line chart 折线图 | X、Y；可选 lines、series、color | 按 series、线组和 X 分组；Y 按聚合规则汇总，再按 X 排序 | 位置表达连续变量或时间和值；连线只表示顺序/趋势，不证明因果。源码中颜色聚合在外层线组记录上进行，若期待颜色随每个 X 点变化，应核验映射粒度 |
| Horizon graph 地平线图 | X；可选 Y 和 group | 按组与 X 聚合 Y；把数值范围切成 bands，并用正负色彩/位置压缩纵向空间 | 颜色和带区表达正负及幅度；读者不能把折叠后的纵向位置直接当作普通零基线 |
| Streamgraph 河流图 | X、size；可选 streams、series | D3 stack；可选择 streams 排序和 offset；源码明确拒绝负数 | 多系列堆叠面积表达连续变化。Wiggle、Silhouette 等偏移改变基线；不能把偏移后的上下位置解释成原始零点 |
| Bump chart 排名变化图 | X；size；可选 rank、streams、series | 先对类别与 X 聚合；可使用显式 rank，否则根据流的 size 计算排序位置；源码拒绝负 size | 纵向位置表示名次/排序，而非原始数值大小。显式 rank 和按 size 自动排序是两种不同口径 |
| Gantt chart 甘特图 | startDate、endDate；可选 group、color | 要求开始和结束字段为同一数据类型；组内按开始时间排序，通过区间重叠检查分配不同 level/lane | 横向区间长度表达持续跨度，位置表达开始/结束；垂直 lane 是防止区间重叠的布局结果，不是额外统计量 |

### C. 分布、相关和多变量

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Beeswarm plot 蜂群图 | X 连续值/日期；可选 size、color、label、series | 每条记录保留为点；D3 force simulation 与碰撞力调整点位以减少重叠 | X 是数据值；Y 方向主要是避让和分组位置，不是第二个测量变量 |
| Box plot 箱线图 | 数值 value；可选 group、color | 排序后计算 Q1、median、Q3；按 IQR multiplier 计算上下界，并将界内实际极值作为须端点，界外值作为离群点。默认 multiplier 为 1.5 | 箱体表达四分位区间，中线表达中位数，须和离群点表达分布范围。须端点不是直接画在理论围栏上，而是组内实际观测值 |
| Violin plot 小提琴图 | 数值 value；可选 group、color | 使用 `d3.bin()` 对每组数值分箱；轮廓宽度由各 bin 的记录数生成，并可显示原始点 | 本仓库实现的是分箱计数形成的镜像分布轮廓；不要未经核验就称为核密度估计（KDE） |
| Bubble chart 气泡图 | X、Y 连续变量；可选 size、color、connectedBy、label、series | 每条输入记录作为点；连续坐标映射；size 使用平方根尺度控制圆半径；可按字段连接点 | X/Y 是两个测量维度，圆面积编码 size，颜色编码额外变量；相关不等于因果 |
| Contour plot 等高线图 | X、Y 连续变量 | 调用 `d3-contour` 的 `contourDensity` 对二维点云做密度估计；带宽影响平滑 | 等值轮廓表达估计密度，不是地形海拔，也不是原始点之间的真实边界。当前 dimensions 中 weight 字段仍为注释代码，不应写成已开放的正式输入 |
| Hexagonal binning 六边形分箱图 | X、Y 连续变量 | 使用 `d3-hexbin` 将点分入六边形 bin；bin 中的记录数用于颜色或尺寸 | 六边形格内计数是分箱计数，不是连续核密度估计；网格大小会改变聚集形态 |
| Convex hull 凸包图 | X、Y；可选 group、label | 按组收集点，调用 `d3.polygonHull` 计算最小凸包 | 包络外边界表达点集的凸外包，不表达密度、真实行政边界或点之间的连通路径 |
| Voronoi diagram 泰森多边形 | X、Y 点位；可选 color、label | `d3.Delaunay.from(points)` 建立 Delaunay 结构，再生成 Voronoi 单元 | 每个单元内的空间位置离其生成点最近；单元面积由点位和绘图边界决定，不代表该点权重 |
| Parallel coordinates 平行坐标图 | 多个 number/date 字段；可选 color | 每条记录保留为一条多维折线；每个维度建立自己的线性尺度 | 折线穿过各轴上的对应值；各轴独立映射范围，不能把不同轴间的线段斜率直接解释为原始单位差值 |

### D. 网络关系和流向

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Arc diagram 弧线关系图 | Source、Target；可选 size | 将关系记录汇总为边，建立节点、入/出边和度数；可用 Louvain 社群结果辅助节点排序 | 节点沿基线排列，弧线连接节点；节点排序会显著改变交叉。Louvain 在这里服务于布局排序，不是因果分类 |
| Chord diagram 弦图 | Source、Target；可选 size | 关系边转为有向邻接矩阵，再用 `d3.chordDirected()` 计算节点弧段和弦带 | 圆周弧段对应节点，弦带宽度表达有向关系量；节点次序和边数量影响可读性 |
| Sankey diagram 桑基图 | Source、Target；可选 size | 关系边转为 nodes/links，使用 `d3-sankey-circular` 的 `sankeyCircular()` 计算节点位置和流带；支持循环链接和多种节点对齐方式 | 节点矩形与连线宽度表达流量结构，方向表达流向；流量单位与边权重必须明确 |
| Alluvial diagram 冲积图 | 至少两个有序 steps 字段；可选 size | 对每一对相邻阶段字段分别分组；size 映射数值聚合，未映射时用该阶段组合的记录数 | 流带连接相邻阶段中的类别；字段顺序由用户映射决定，不必然等于真实时间。记录必须能在多个阶段之间正确对应 |

### E. 层级与空间组成

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Linear dendrogram 线性树状图 | 多个 hierarchy 字段；可选 size、color、label | 依层级字段嵌套分组并构造 D3 hierarchy；使用 `d3.tree()` 或 `d3.cluster()` | 连线表达父子关系；节点大小/颜色可增加属性，但树枝位置本身不表示数值大小 |
| Circular dendrogram 圆形树状图 | 多个 hierarchy 字段；可选 size、color、label | hierarchy + radial tree/cluster 布局 | 由中心向外展开；径向路径表达层级。外圈标签拥挤，角度和路径长度不宜随意比较 |
| Circle packing 圆形打包图 | 多个 hierarchy 字段；可选 size、color、label | hierarchy 汇总后使用 `d3.pack()` 计算嵌套圆的位置和半径 | 嵌套圆表达包含关系；圆面积与叶节点权重相关，但圆间留白使精确比较不如矩形面积直观 |
| Treemap 矩形树图 | 多个 hierarchy 字段；可选 size、color、label | hierarchy 汇总后使用 `d3.treemap()`；可切换 Squarify、Binary、Dice、Slice 等 tiling | 嵌套矩形面积表达权重；矩形长宽比由布局算法决定，不是额外数据 |
| Sunburst diagram 旭日图 | 多个 hierarchy 字段；可选 size、color、label | hierarchy + D3 partition 布局，将层级映射为角度和径向深度 | 同一层级扇区角度表示分支权重；不同深度扇区的面积还受半径影响，不能只凭面积跨层级比较 |
| Voronoi Treemap Voronoi 树图 | 多个 hierarchy 字段；可选 size、color、label | hierarchy 后调用 `d3-voronoi-treemap` 迭代剖分；支持随机 seed、裁剪形状和多边形边数 | 不规则单元表达层级组成；项目 metadata 明确警告，迭代计算可能导致单元面积不能完全代表映射值，随机种子影响复现 |

### F. 多指标轮廓和两端比较

| 图表 | 输入与映射 | 核心计算 / 布局 | 主要编码与边界 |
|---|---|---|---|
| Radar chart 雷达图 | 至少 3 个数值 axes；可选 color、series | 每条记录展开为多个轴值；径向角度由轴顺序确定；源码使用统一的 `[0, maxValue]` 数值域 | 多指标轮廓用于同组指标对照；轴顺序会改变形状，面积不等于综合得分 |
| Slope chart 斜率图 | Source、Target 两个数值/日期字段；可选 name、color、series | 每条记录保留两端值，使用共同数值尺度；标签可用力模拟避让 | 两端位置和连线方向表达变化；只观察两个端点，不展示中间过程 |

## 4. 需要特别区分的默认值与聚合口径

**未映射 size 不代表所有图表都自动按原始记录数计数。** 源码存在多种不同默认行为：

| 图表/家族 | 未映射 size 时的实现行为 | 解释时的注意事项 |
|---|---|---|
| Bar chart | 分组后的 `v.length` | 该类别组内的原始记录数 |
| Alluvial diagram | 相邻阶段组合的 `v.length` | 该相邻阶段流带对应的记录数 |
| Matrix plot | X/Y 组合的 `v.length` | 该矩阵格子的记录数 |
| 层级图 | 叶节点组的 `v.length` | 该层级叶节点下的记录数 |
| Arc / Sankey / Chord | 每条已聚合关系边赋值 1 | 是每条关系边的单位权重，不是重复原始记录的条数 |
| Beeswarm / Bubble / Parallel coordinates / Slope 等直接映射图 | 多数保留原始行作为点或线 | 不应误称为先按类别汇总 |

此外，`csv`、`csvDistinct` 是拼接类聚合，不是数值统计量；`sum`、`mean`、`max`、`min` 也只有在数据单位和研究问题允许时才有意义。

## 5. 源码阅读发现的实现风险与核验提醒

以下是代码审读中发现的具体注意事项。它们不意味着所有图表都会出错，而是提醒后续案例分析不要只看图形外观。

### 5.1 Source–Target 关系的分组键

Arc、Sankey、Chord 的映射代码使用 Source 与 Target 字段拼接成字符串作为分组键（例如 `source + target`），而不是使用结构化二元组键。若类别值存在拼接歧义，例如 `AB + C` 与 `A + BC`，可能形成相同键。Alluvial 的相邻阶段键使用下划线拼接，也应留意类别值中含有下划线的情况。

**研究建议：** 对关系数据先检查 Source/Target 唯一组合；重要案例最好保留原始关系表或预先生成无歧义的组合 ID。

### 5.2 关系图无 size 的默认权重

Arc、Sankey、Chord 在未映射 size 时，将聚合后的每条关系边设为 1。若原始数据中同一 Source–Target 组合出现多行，不能直接把图中边宽解释为该组合的原始记录频次。

**研究建议：** 需要频次时，先按关系组合计算 count 字段，再映射为 size；需要强度时，应使用有单位、有定义的权重。

### 5.3 Violin plot 不是核密度估计

本仓库小提琴图使用 `d3.bin()` 进行数值分箱，轮廓宽度来自每个 bin 的计数。它与通过核函数和平滑带宽估计概率密度的 KDE 实现不同。

**研究建议：** 案例中若使用“小提琴图”一词，必须进一步核对其计算方式，不能仅凭轮廓形状认定使用了 KDE。

### 5.4 Contour plot 当前没有开放 weight 映射

`contourPlot/dimensions.js` 中 weight 维度处于注释状态；映射对象中虽出现 weight 的 get 入口，但这不等于 UI 已提供可配置的权重输入。

**研究建议：** 当前实现应按未加权二维点云密度理解；不要把它描述成加权密度估计，除非有额外代码或数据证据。

### 5.5 Sunburst、Treemap 与 Voronoi Treemap 的面积含义不同

- Treemap：矩形面积依权重分配。
- Sunburst：角度/分区布局依层级权重分配；不同半径下的面积不能直接跨层级比较。
- Voronoi Treemap：项目自身说明迭代剖分可能使面积不能完全代表输入权重。

**研究建议：** 任何面积编码都应检查数据权重、布局算法与图例口径，不可一概而论。

### 5.6 布局位置不一定是统计变量

- Beeswarm 的竖直避让位置不是第二个测量维度。
- Gantt 的 lane/level 是区间防重叠布局。
- Arc Diagram 的社群排序可用于减少交叉，不等于关系的因果分类。
- Voronoi 单元面积不是点的权重。
- 凸包边界不是数据分布密度。
- 树状图节点坐标由层级布局决定，不是数值坐标。

### 5.7 需要进一步核验的映射粒度

折线图 `mapping.js` 在内层 X 分组生成点时，颜色聚合器接收的是外层线组数据 `v`，而非内层 X 组数据 `vv`。因此，颜色聚合更接近线组级别，而非每个 X 点分别计算。

**研究建议：** 若案例颜色意图表达逐点变化，应检查具体版本代码和数据映射，不要仅从图面颜色推断。

## 6. 版本与开发状态信息

- `package.json` 声明版本 `v1.1.0`，主要运行依赖包括 D3 及 contour、hexbin、sankey、voronoi-treemap、gridding、Louvain 等扩展。
- `CHANGELOG.md` 中 v1.1.0 记录了共享坐标轴与刻度控制的整合，以及甘特图轴位置、蜂群图仅沿 Y 轴移动等调整。
- README 声明图表仍处于 beta，可能有 breaking changes。
- `package.json` 的 test 脚本为占位脚本（`echo "Error: no test specified" && exit 1`），不能把仓库现状描述为已具备完整自动化测试覆盖。

因此，本文记录的是可复核的源码行为，不对项目未声明的统计准确性、算法性能或测试覆盖作额外保证。

## 7. 对本知识库的使用方式

分析一张真实信息图时，建议按以下顺序判断：

1. **先判断数据逻辑**：信息对象之间是编年、层级、编目、字母表还是地理关系？
2. **再判断作品表现类型**：整张作品是混合型、列表型、时间轴型、流程图型或其他类型？
3. **逐个识别局部统计图形**：是否真的存在数值变量、统计量或可量化的视觉编码？
4. **反推输入结构**：一行数据代表什么？有哪些字段？单位和时间范围是什么？
5. **还原计算口径**：如何分组、聚合、计数、排序、分箱或估计密度？
6. **解释视觉编码**：位置、长度、面积、角度、颜色、线宽和连通性分别代表什么？
7. **区分统计与布局**：哪些属性来自计算结果，哪些只是排版或避让？
8. **标注证据等级**：图面可确认 / 根据结构推断 / 需原始数据或制作文件核验。

允许得出“无独立统计图形”的结论。不能因为作品出现图标、人物、箭头、节点或复杂几何形态，就强行将其归为统计图表。

## 8. 核心源码入口

- [图表导出入口 src/index.js](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/index.js)
- [图表输入维度示例：Bar chart](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/barchart/dimensions.js)
- [图表数据映射示例：Bar chart](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/barchart/mapping.js)
- [箱线图分位数与离群值：Box plot](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/boxplot/render.js)
- [分箱轮廓：Violin plot](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/violinplot/render.js)
- [二维密度：Contour plot](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/contourPlot/render.js)
- [六边形分箱：Hexagonal binning](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/hexagonalBinning/render.js)
- [流向网络：Sankey diagram](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/sankeydiagram/render.js)
- [层级面积布局：Treemap](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/treemap/render.js)
- [Voronoi 层级剖分：Voronoi Treemap](https://github.com/rawgraphs/rawgraphs-charts/blob/master/src/voronoitreemap/render.js)
- [项目版本变更记录 CHANGELOG](https://github.com/rawgraphs/rawgraphs-charts/blob/master/CHANGELOG.md)
