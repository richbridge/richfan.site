---
title: PowerBI设置单位为万
toc: true
date: 2024-08-14
categories:
  - 可视化
---

## 方法一：

> PowerBI默认的格式化字符串的方式为“千”、“百万”与“十亿”，不符合国人使用习惯，今天给大家介绍如何利用计算组结合动态字符串可以修改字符串格式为“万”与“亿“。

在最新版的PowerBI中已支持计算组与动态字符串设置，如果你发现的的操作界面没有动态字符串功能及模型视图，请至官网下载最新版PowerBI Desktop并在预览功能中打开

度量值的动态格式字符串

模型资源管理器和计算组创作

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/1-1.avif#center)

### 步骤分解

#### 切换模型视图

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/1-2.avif#center)

#### 新建计算组

右键语义模型中的计算组，选择新建计算组

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/1-3.avif#center)

#### 设置动态格式

建立计算组后，会自动建立一个计算项，选中计算项，在“属性”面板中打开动态格式字符串

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/1-4.avif#center)

将以下代码复制到动态格式窗格中,并设置一个合适的计算项名称(比如“格式化字符串”)

```dax
VAR CurrentValue = abs(SELECTEDMEASURE())
RETURN
SWITCH (
    TRUE (),
    CurrentValue < 1E4, "#,0" ,
    CurrentValue <= 1E8, ".'" 
         & FORMAT ( CurrentValue/1E4, "0.00 万" ),
    CurrentValue <= 1E80,".'" 
         & FORMAT ( CurrentValue/1E8, "0.00 亿" ),
         SELECTEDMEASUREFORMATSTRING()
)
```

#### 运用计算组

选中你想格式化字符串的视觉对象，将“计算列”拖至筛选面板的“此视觉对象上的筛选器”，选中刚刚构建的计算项“格式化字符串”，表格中的数值就切换成以“万”和“亿”为单位的显示格式了。

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/1-5.avif#center)

### 思考

更进一步，如果再一个表格中有多个度量值，我们只想要其中一部分度量值的单位修改为“万”，另一部分保持在PowerBI中设置度量值格式，改怎么处理呢？

修改计算项中的动态格式字符串表达式如下内容即可，以下表达式指定了只对”交易金额”,“订单数”,“交易用户数”这三个度量值生效动态格式字符串，大家可以根据实际需求修改为自己需要的内容

```dax
VAR CurrentValue = abs(SELECTEDMEASURE())

VAR __custFormat = SWITCH (
    TRUE (),
    CurrentValue < 1E4, "#,0" ,
    CurrentValue <= 1E8, ".'" 
         & FORMAT ( CurrentValue/1E4, "0.00 万" ),
    CurrentValue <= 1E80,".'" 
         & FORMAT ( CurrentValue/1E8, "0.00 亿" ),
         SELECTEDMEASUREFORMATSTRING()
)
RETURN
IF(SELECTEDMEASURENAME() in {"交易金额","订单数","交易用户数"},__custFormat,SELECTEDMEASUREFORMATSTRING())
```

## 方法二：

### 1：第一步，BI中新建单位表

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-1.avif#center)

这是dax代码

```dax
单位 =
SELECTCOLUMNS(
{
( "元", 1),
("万",10000),
("千万元", 10000000),
("亿元", 100000000)
} , "单位" , [Value1] , "单位值" , [Value2] )
```

建表后

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-2.avif#center)

###  2：第二步，准备度量值

我们假设现在要计算一个流量_曝光人数的求和

> 流量_曝光人数 = CALCULATE(SUM('流量数据'[曝光人数]))

然后新建一个度量值用于测试，这里使用了SELECTEDVALUE函数，会去获取当前的筛选条件，我们利用了这个函数特性，例如我筛选器选择"万",那么这个函数会返回10000，配合这个除法公式，就会在原有的值上去除10000。

> 流量_曝光人数_测试 = [流量_曝光人数] / SELECTEDVALUE ( '单位'[单位值], 1 )

假设我们现在有一个值是"230000"，那么现在我们得到的值应该就是"23"，按照我们之前的预期，我们想要的到的是"23万"，离目标还差一个单位值的拼接。

点选度量值以后，先设置度量值格式为动态格式

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-3.avif#center)

然后进入度量值格式设置

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-4.avif#center)

在格式栏，设置以下参数，这里的代码意思是，这个度量值保留一位小数，并且用连接符号"&"拼接单位这张表里的单位名称，我们在上一步已经得到了"23"如果再拼接上，目前被选择的"万"，是不是就能得到"23万"了呢?

> "0.0"&SELECTEDVALUE ( '单位'[单位] )

### 第三步：验证和测试

我们新建一个卡片图，

将单位拉入我们的卡片图筛选器，勾选单选，并勾选"万"

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-5.avif#center)

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-6.avif#center)

看一下最后效果，已经是我们的预期了，同样的道理，也适用与表格或者其他形式的视觉对象

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-7.avif#center)

并且我们可以通过切换切片器的单位，去切换我们的展示单位。如下面我们切换到"亿元"，其他的单位都是支持的。

![](https://jsd.cdn.zzko.cn/gh/richbridge/picx-images-hosting@master/powerbi/PBI_dynamicformat/2-8.avif#center)