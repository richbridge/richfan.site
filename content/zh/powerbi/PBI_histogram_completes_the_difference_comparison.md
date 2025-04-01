---
title: PowerBI柱状图完成差异对比
titleIcon: "fa-solid fa-newspaper"
page-layout: full
toc: true
comments: false
author: richfan
date: '2024-08-14'
categories: PowerBI
---

在PowerBI里面，默认自带的原生视觉对象，是可以自由设置功能最丰富了。比如今天要演示的柱状图，就可以通过对视觉对象误差线的设置，来实现同比变化值突出显示的效果。不仅可以标记变化长度，还能够使用颜色标记增长为绿色，下降为红色，同时标记出增长变化率。这些都是在同一个视觉对象上完成的，效果如下↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/1.gif)

这种可视化方式最常用的就是同期对比，或者是和目标值的对比。我们这里就使用两种常用的方法来进行制作演示，了解了思路后，可以根据实际情况做出更多的效果。

## 【同比变化演示】

首先是同比变化情况，我们的数据是2023和2024两年产品每月的销售金额，要在图形上直观的展示2024年对比2023年的增长变化情况，首先准备数据，格式很简单，就是每月每个产品的销售金额，如下↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/2.avif)

然后把数据导入到PowerBI里面，先创建两个最基本的度量值，销售金额和上年同期的销售金额，并计算出金额的差值↓

```dax
销售金额 = SUM([GMV])
上年销售金额 = 
CALCULATE(
    [销售金额],
    DATEADD('_date'[Date],-1,YEAR)
)
同比差值 = [销售金额] - [上年销售金额]
```

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/3.avif)

这三个字段计算好了之后，就可以在PowerBI里面进行可视化操作了，使用的视觉对象是默认的簇状柱形图，演示一下过程↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/4.gif)

然后简单设置一下本年和上年的颜色。上一年设置成白色，黑色边框填充；今年的值设置成黑色，黑色边框填充↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/5.gif)

然后再设置一下布局效果，把重叠打开，设置一下类别间距和系列间距，两个柱状图就有了一定的融合效果，就基本上有了我们需要的效果了↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/6.gif)

接下来新建两个度量值，用来分别展示比上期高和比上期低的值↓

```dax
同比增长 = IF([同比差值] > 0,[最大值])
同比下降 = IF([同比差值] < 0,[最大值])
把度量值分别放入更多设置的误差项里面。需要设置一下颜色，宽度根据实际情况设置，并且把边框颜色设置为0。
```

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/7.gif)

这是增长的绿色部分，下降的红色部分同理。基本上就得到了我们想要的效果了。最后就是简单的装饰部分，把增减的部分放入值进行显示，并自定义一个颜色度量值用来自动变化颜色就可以了↓

```dax
color差值 = 
IF([销售金额]>[上年销售金额],
"#32a11b","#c90032")
```

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/8.avif)

如果不想展示差异的绝对值，也可以使用变化百分比来进行性展示，度量值和效果如下↓

同比百分比 = DIVIDE([销售金额] - [上年销售金额],[上年销售金额])

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/9.avif)

## 【目标完成率演示】

目标完成率思路是完全一样的，只是对比的内容是目标值。数据字段如下，多了一列目标值，其他都是一样的↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/10.avif)

导入PowerBI，然后使用日期维度表建立关系↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/11.avif)

下面按照同样的思路，进行度量值的建立和绘图↓

```dax
_销售金额 = SUM([GMV])
_目标 = SUM([GMV目标])
_差值 = [_销售金额]-[_目标]
_最大值 = MAX([_目标],[_销售金额])
_目标完成 = IF([_销售金额] > 0,[_最大值])
_目标未完成 = IF([_销售金额] < 0,[_最大值])
_color差值 = 
IF([_销售金额]>[_目标],
"#32a11b","#c90032")
```

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/12.avif)

也可以按照目标完成率来进行展示↓

```dax
完成率 = DIVIDE([_销售金额],[_目标])
```

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/13.avif)

如果为了更方便二者进行筛选，可以做一个书签，根据需求进行选择↓

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_histogram_completes_the_difference_comparison/14.gif)

## 【本文使用的数据Excel文件和PowerBI文件↓】

```dax
链接：https://pan.baidu.com/s/1IUb6NSbKnpEoOksvCm3y5Q?pwd=3zzl 
提取码：3zzl
链接：https://pan.baidu.com/s/18ep-lrLsyj5opUEzYKGDWQ?pwd=5868 
提取码：5868
```