---
title: 【个人翻译】VRChat文档——PhysBones
date: 2022-04-22 21:43:49
tags:
    - VRChat
    - 翻译
---

# PhysBones

---

PhysBones是一套能够为avatar的头发，尾巴，耳朵，衣服等部位添加次要动作的组件。合理的使用这套组件能够让你的avatar看起来更加生动且逼真。

PhysBones是Dynamic Bones的一套替代方案。虽然两者在概念上大同小异，但PhysBones还是与Dynamic Bones有较大的差别。因此，并非所有avatar都能直接适配VRChat的系统。

你可以在`VRCSDK\Examples3\Dynamics\Robot Avatar`目录下找到一个关于如何使用Avatar Dynamics的例子。

# VRCPhysBone
该组件设定了一条骨骼链，其可以被用于模拟软体以及次要动作。例如：头发，尾巴，软踏踏的耳朵等。其包含许多可供配置的属性，且它们都能以多种方式进行设置。

此外，任何人都可以与PhysBones互动。如果你允许，其他玩家能够抓取你的avatar上的PhysBones，并在保持抓取时通过扣下扳机键来对其调整且保持其位置。你也能通过禁用此选项来禁止对你的PhysBone造型，抓取或者完全禁用。

尽管并非设计的本意，在我们实现我们的布料组件前，PhysBones也可以作为其替代品使用。

图片

## Transforms

`Root Transform` - 此组件的开始位置。如果该属性留空，则默认从此game object开始。
`Ignore Transforms` - 不受此组件影响的transform列表。此属性将默认包含
`Endpoint Position` - 用于在每条骨骼链末端创建额外骨骼的向量。在该值非零时才会启用。通常，你会将其朝+Y方向（上方）设置。

注意：

如果你在为单个骨骼，或者父级与其的几个子级（但不包含孙级）骨骼添加此组件，你**必须**设定一个endpoint position！

举个例子：假设你为以下任意`RootBone`添加了PhysBone组件，为保证PhysBone正常运作，你**必须**为其设定一个**Endpoint Position**。这是与Dynamic Bones不同的一点，请注意。

单个骨骼
* RootBone

多个子级与一个父级
* RootBone
    * ChildBone1
    * ChildBone2
    * ChildBone3
    * ChildBone4

你也能通过往每个`ChildBone`后添加“末端骨骼”来解决这个问题，但这会牵涉到编辑骨骼。

## Forces

**Integration Type**决定了受此组件影响的所有transform要如何模拟其动作。
取决于你的选项，Forces栏目中的其他选项也会有所不同，你可以在以下两个选项中选择：

* `Simplified`对外界的力的反应更加缓慢且微小，但相比更加稳定，且配置起来较为简单。
* `Advanced`对外界的力反应更加敏感，但相比没那么稳定。你可以对其进行更加复杂细致的配置。

在默认设置下，两者的表现大致相当，但在调整并测试这些设定之后你能更加直观的了解其区别。

你可以通过点击大多数属性条旁的C按钮来为其启用曲线。通过调整曲线，你可以adjust the value over the length of the bone chain，且可以对骨骼链进行**非常**细致的设置。

事实上，绝大多数PhysBones的设定支持你使用曲线进行调整。学会如何使用他们能让你的PhysBones格外出彩。

图片

`Pull` - 决定用于将骨骼回归默认位置的力。
`Spring` - 决定在骨骼回到其默认时其抖动程度。仅在选择`Simplified`时启用。
`Momentun` - 决定在骨骼回到其默认时其抖动程度。仅在选择`Advanced`时启用。尽管此处描述相同，其效果与Spring的效果还是有些细微的差别。
`Stiffness` - 决定骨骼尝试保持其默认的强度。仅在选择`Advanced`时启用。
`Immobile` - 决定运动对骨骼的影响程度。该值为1时，骨骼在没有与碰撞器互动或被抓取时，将永远保持其默认位置。
`Gravity` - 决定重力对骨骼的影响程度。正数值会让骨骼受到向下的力，反之亦然。
`Gravity Falloff` - 仅当Gravity不为零时启用。其决定了骨骼在默认位置时被移除的重力影响程度。该值为1时，当骨骼位于默认位置时其将永远不会受到重力的影响。这让你可以使骨骼在离开默认位置时才受到重力影响，而在静止时不受影响。

此处介绍Gravity Falloff的一个用法：如果你的avatar的头发已经建模成了默认站立时的形态，你可以将Gravity Falloff设置为1，这样在你站立时你的头发将不会受重力影响。但如果avatar的头发建模是45度斜向的话，你可以将该值调整到例如0.5到0.8，这样能在你站立时让头发受到部分重力的影响，从而呈现出一个优雅的曲线。

## Limits

设置PhysBones的Limits可以让你限制骨骼链的移动，可以避免出现如头发与头部的穿模。且这比设置一个碰撞器要更加节省性能。

此外，当你为PhysBones设置Limits时，在你的Scene窗口中会出现已选中的骨骼链限制范围的可视化视图。这在你调整其限制范围时十分有用。

#### None

`None`表示此骨骼的运动没有被限制，所以也没有任何属性可以设置。

#### Angle（角度）

`Angle`表示此骨骼链的运动会被限制以`Rotation`（旋转）所设定的轴为中轴的某个`Max Angle`（最大角度）内。在Scene窗口中，其限制范围会被表示为一个圆锥形。

#### Hinge（铰链）

`Hinge`表示表示此骨骼链的运动会被限制以`Rotation`（旋转）所设定的平面的某个`Max Angle`（最大角度）内。在Scene窗口中，其限制范围会被表示为一个扇形。

#### Polar（极坐标）

`Polar`稍微有点复杂。如果你用一个`Hinge`横着扫过`Yaw`值的范围，你就得到