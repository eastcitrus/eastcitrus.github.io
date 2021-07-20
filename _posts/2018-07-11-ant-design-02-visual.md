---
layout: post
title:  Ant Design-02-Visual
date:  2018-07-11 10:17:10 +0800
categories: [Design]
tags: [web, design, sh]
published: true
---

# 色彩

Ant Design 将色彩体系解读成两个层面：系统级色彩体系和产品级色彩体系。

系统级色彩体系主要定义了蚂蚁中台设计中的基础色板、中性色板和数据可视化色板。产品级色彩体系则是在具体设计过程中，基于系统色彩进一步定义符合产品调性以及功能诉求的颜色。

## 色彩模型

Ant Design 的设计团队倾向于采用 HSB 色彩模型进行设计，该模型更便于设计师在调整色彩时对于颜色有明确的心理预期，同时也方便团队间的沟通。

## 系统级色彩体系

Ant Design 系统级色彩体系同样源于『自然』的设计价值观。

设计师通过对自然场景的抽象捕捉，结合蚂蚁的技术基因，形成了特有的 12 色。进一步又通过大量的观察，捕捉不同色彩在自然光下的变化规律，借助美术中素描的思路，对 12 个颜色进行了衍生。在中性色板的定义上，则是平衡了可读性、美感以及可用性得出的。

### 基础色板

[基础色板](https://ant.design/docs/spec/colors-cn#%E5%9F%BA%E7%A1%80%E8%89%B2%E6%9D%BF)

### 中性色板

中性色包含了黑、白、灰。在蚂蚁中后台的网页设计中被大量使用到，合理的选择中性色能够令页面信息具备良好的主次关系，助力阅读体验。Ant Design 的中性色板一共包含了从白到黑的 10 个颜色。

[中性色板](https://ant.design/docs/spec/colors-cn#%E4%B8%AD%E6%80%A7%E8%89%B2%E6%9D%BF)

# 产品级色彩体系

## 品牌色的应用

品牌色是体现产品特性和传播理念最直观的视觉元素之一。在色彩选取时，需要先明确品牌色在界面中的使用场景及范围。在基础色板中选择主色，我们建议选择色板从浅自深的第六个颜色作为主色。 

Ant Design 的品牌色取自基础色板的蓝色，Hex 值为 1890FF，应用场景包括：关键行动点，操作状态、重要信息高亮，图形化等场景。

## 功能色

功能色代表了明确的信息以及状态，比如成功、出错、失败、提醒、链接等。功能色的选取需要遵守用户对色彩的基本认知。
我们建议在一套产品体系下，功能色尽量保持一致，不要有过多的自定义干扰用户的认知体验。

## 中性色

Ant Design 的中性色主要被大量的应用在界面的文字部分，此外背景、边框、分割线、等场景中也非常常见。产品中性色的定义需要考虑深色背景以及浅色背景的差异，同时结合 WCAG 2.0 标准。Ant Design 的中性色在落地的时候是按照透明度的方式实现的。

# 布局

空间布局是体系化视觉设计的起点，和传统的平面设计的不同之处在于，UI界面的布局空间要基于『动态、体系化』的角度出发展开。我们受到建筑界大师柯布西耶的模度思想的启发，基于『秩序之美』的原则，探索 UI 设计中的动态空间秩序，形成了 Ant Design 的界面布局方式，为设计者构筑具备理性之美的布局空间创造了条件。

在中后台视觉体系中定义布局系统，我们建议从 5 个方面出发：

- 统一的画板尺寸

- 适配方案

- 网格单位

- 栅格

- 常用模度

## 统一的画板尺寸

为了尽可能减少沟通与理解的成本，有必要在组织内部统一设计板的尺寸。

蚂蚁中台设计团队统一的画板尺寸为 1440px。

## 适配方案

在设计过程中，设计师还需要建立适配的概念，根据具体情况判断系统是否需要进行适配，以及哪些区块需要考虑动态布局。据统计，使用中台系统的用户的主流分辨率主要为 1920、1440 和 1366，个别系统还存在 1280 的显示设备。

Ant Design 两种较为典型的适配方案：

### 左右布局的适配方案

常被用于左右布局的设计方案中，常见的做法是将左边的导航栏固定，对右边的工作区域进行动态缩放。

### 上下布局的适配方案

常被用于上下布局的设计方案中，做法是对两边留白区域进行最小值的定义，当留白区域到达限定值之后再对中间的主内容区域进行动态缩放。

### 网格单位

Ant Design 通过网格体系来实现视觉体系的秩序。网格的基数为 8，不仅符合偶数的思路同时能够匹配多数主流的显示设备。
通过建立网格的思考方式，还能帮助设计者快速实现布局空间的设计决策同时也能简化设计到开发的沟通损耗。

### 关于栅格

Ant Design 采用 24 栅格体系。
以 1440 上下布局的结构为例，对宽度为 1168 的内容区域 进行 24 栅格的划分设置，如下图所示。
我们为页面中栅格的 Gutter 设定了定值，即浏览器在一定范围扩大或缩小，栅格的 Column 宽度会随之扩大或缩小，但 Gutter 的宽度值固定不变。

对开发者而言栅格是实现动态布局的手段，而设计师对于栅格的理解源自平面设计中的栅格。在具体落地中视角的不同就容易造成偏差，最终影响还原度，继而增加沟通成本。

Ant Design 的设计师通过 4 点来实现设计过程中和工程师的沟通：

1. 清晰的定义动态布局范围

2. 尽量保持偶数思维

3. 关键数据的交付（Gutter、Column）

4. 区块的定义要从 column 开始到 column 结束

### 常用模度

蚂蚁中后台涵盖了大量的不同类型和量级的产品，为了帮助不同设计能力的设计者们在界面布局上的一致性和韵律感，统一设计到开发的布局语言，减少还原损耗，Ant Design 提出了 UI 模度的概念。
在大量的实践中，我们提取了一组可以用于UI布局空间决策的数组，他们都保持了 8 倍数的原则、具备动态的韵律感。
经过验证，可以在一定程度上帮助我们更快更好的实现布局空间上的设计决策。

### 是启发，而非限制

Ant Design 在布局空间上的成果并非要限制设计产出，更多的在于引导设计者如何做到『更好』。
8 倍数的双数组通过排列组合的方式可以形成千变万化种可能性，但在无限的可能性之中依然存在着『只是简单的套用数据组合』同『看起来很精妙』的差别。实现合理优雅的界面布局，在对美感的追求之上，还应当结合可用性来看待，对于企业级应用界面布局的探索，我们依然在路上。

# 字体

字体是体系化界面设计中最基本的构成之一。

我们的用户通过文本来理解内容和完成工作，科学的字体系统将大大提升用户的阅读体验及工作效率。

Ant Design 字体方案，是基于『动态秩序』的设计原则，结合了自然对数以及音律的规则得出的，再经过了大量的蚂蚁中后台产品验证之后，推荐给大家。 

在中后台视觉体系中定义字体系统，我们建议从下面五个方面出发：

1. 字体家族

2. 主字体

3. 字阶与行高

4. 字重

5. 字体颜色


## 字体家族

优秀的字体系统首先是要选择合适的字体家族。

Ant Design 的字体家族中优先使用系统默认的界面字体，同时提供了一套利于屏显的备用字体库，来维护在不同平台以及浏览器的显示下，字体始终保持良好的易读性和可读性，体现了友好、稳定和专业的特性。

```css
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
             "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei",
             SimSun, sans-serif;
```

> [using-system-ui-fonts-practical-guide](https://www.smashingmagazine.com/2015/11/using-system-ui-fonts-practical-guide/)

另外，在中后台系统中，数字经常需要进行纵向对比展示，我们单独将数字的字体设置为 Tahoma，使其为等宽字体。

> [数字对齐问题](http://stackoverflow.com/questions/13611420/set-a-font-specifically-for-all-numbers-on-the-page)

## 主字体

我们基于电脑显示器阅读距离（50 cm）以及最佳阅读角度（0.3）对 Ant Design 的主字体进行了一次升级，
从原先的 12 上升至 `14`，以保证在多数常用显示器上的用户阅读效率最佳。

## 字阶与行高

字阶和行高决定着一套字体系统的动态与秩序之美。字阶是指一系列有规律的不同尺寸的字体。行高可以理解为一个包裹在字体外面的无形的盒子。

Ant Design 受到 5 音阶以及自然律的启发定义了 10 个不同尺寸的字体以及与之相对应的行高。

在 Ant Design 的视觉体系中，我们建议的主要字体为 14，与之对应的行高为 22。其余的字阶的选择可根据具体情况进行自由的定义。建议在一个系统设计中（展示型页面除外），字阶的选择尽量控制在 3-5 种之间，保持克制的原则。

| Font Size   | 12 | 14 | 16 | 20 | 24 | ... |
| Font Weight | 20 | 22 | 24 | 28 | 32 | ... |

## 字重

字重的选择同样基于秩序、稳定、克制的原则。

多数情况下，只出现 regular 以及 medium 的两种字体重量，分别对应代码中的 400 和 500。在英文字体加粗的情况下会采用 semibold 的字体重量，对应代码中的 600。

## 字体颜色

文本颜色如果和背景颜色太接近就会难以阅读。考虑到无障碍设计的需求，我们参考了 WCAG 的标准，将正文文本、标题和背景色之间保持在了 7:1 以上的 AAA 级对比度。

[字体颜色](https://ant.design/docs/spec/font-cn#%E5%AD%97%E4%BD%93%E9%A2%9C%E8%89%B2)

## 建议

字体系统的构建，是『动态秩序之美』的第一步。在实际的设计中，我们还有三点建议：

- 建立体系化的设计思路：在同一个系统的 UI 设计中先建立体系化的设计思路，对主、次、辅助、标题、展示等类别的字体做统一的规划，再落地到具体场景中进行微调。建立体系化的设计思路有助于强化横向字体落地的一致性，提高字体应用的性价比，减少不必要的样式浪费。

- 少即是多：在视觉展现上能够用尽量少的样式去实现设计目的。避免毫无意义的使用大量字阶、颜色、字重强调视觉重点或对比关系。

- 尝试让字体像音符一样跳跃：在需要拉开差距的时候可以尝试在字阶表中跳跃的选择字体大小，会令字阶之间产生一种微妙的韵律感。

# 图标

图标是具有指代意义的图形，也是一种标识。通过使用图标表达命令，强调状态，表示产品或类别。
为了系统及跨平台之间图形认知保持一致，Ant Design 的图标在设计和使用时有以下两个原则点需要注意：

- 简单的图形语言以及高辨识度。清晰、直观的图标更能明确指代含义便于识别记忆；

- 保持图标之间一致的风格和表现方式。界面中的所有图标都应该在细节设计、透视和笔画权重上保持一致。

## 系统图标

系统图标通常用来表示常用的操作，比如：删除、保存、编辑。也可以用来表示一个文件或者状态。

> [ICON 库](https://ant.design/components/icon-cn/)

## 关键轮廓线

根据不同的图标形状类型使用不同的轮廓线，可以使图标之间保持一致的视觉效果。

请将所有图标在 1024×1024（16×16 的 64 倍）的画板中制作。

> [制作技巧](https://zos.alipayobjects.com/rmsportal/hmNuLjCkBssupcZgYAde.png)

## 笔画

一致的笔画权重是保持整个图标系统视觉统一的关键，Ant Design 系统图标的线条统一为 72px 宽度。

## 边角

一致的角度半径也是保持整个图标系统视觉统一的重要元素。

Ant Design 的图标设计中，外轮廓线统一半径为 72px 的圆角，icon 内部空间的边角保持直角，笔画的终端为圆角。


## 视觉修正

在一些特殊情况下（比如，icon 的形状比较复杂紧凑），可以通过调整线条的粗细或圆角的大小等微妙的变化来增加图标的易读性。

## 透明角度

始终保持简洁、平面的风格,不要让图标具有多维度空间感，或者充满了细节。

## 命名规则

统一的命名方式有助于管理图标，也能更快速的找到需要的图标。例如，环绕型的图标统一以「-o」后缀。

## 图标尺寸

应用于页面时请使用 Ant Design 的规范尺寸，与字体搭配时和字体的尺寸保持一致。

例如：和 12pt 字体搭配时，图标使用 12px，图标与文字的间距为 8px。

## 颜色

图标的颜色需要与搭配文案的色值保持一致（表示状态的除外）



* any list
{:toc}