---
title: VRChat Expression 系统设计经验分享
date: 2022-04-11 15:17:10
tags:
    - VRChat
---

# 前言

写这个的原因是看过很多中文系的Expression系统讲解，大部分都是在讲如何实现某个功能，对于不甚了解Unity或者Expression菜单的读者来说，唯一能做的就是照葫芦画瓢，就算做完之后也不明白其工作原理。

所以我想着随便写点自己这两天改模（其实也就刚开始改了几天模）过程中从不少其他人的讲解中归纳，以及自己悟到的东西，尝试系统性地讲解自己对这个系统工作原理的理解，以及一些生活小妙招（？）。

虽然不一定有人看，但是过一遍流程也算是让自己对所学到的知识的反刍，让自己能够加深理解并且悟出新的东西的话就更好了。

<!-- more -->
# Expression Menu 的工作原理简述

[![LZ3ik9.jpg](https://s1.ax1x.com/2022/04/11/LZ3ik9.jpg)](https://imgtu.com/i/LZ3ik9)

整个Expression Menu的工作原理还是比较简单易懂的。

每个Avatar的Avatar Descriptor里面会各需求一个Expression Menu（后简称Menu）和一个Expression Parameter（后简称Parameter）。Parameter中可以添加总计占用128个**记忆槽**（自译）的参数，其中每个bool占**1**槽，每个int或float占**8**槽。被添加到Avatar Descriptor的Menu作为VRChat内菜单的根菜单，子菜单可以根据需要添加，个人暂时还没有测试出最多可容纳的子菜单数量。

Parameter中的参数名称必须和动画控制器（Animator）中的参数（Parameter）名称以及类型一致，才能通过Menu对其进行操作。在动画控制器中的参数被玩家通过Menu进行的操作所更改后，动画控制器将参数的修改反映到动画（Animation）状态的转换（transition）上，从而实现在游戏中Avatar的换装等动作。

请保证完全理解这一段之后再继续阅读，不然有可能卡壳。

# Animator

Animator是一种为动画服务的状态机，每一段动画就是一个状态（State）。两个存在过渡（transition）之间的状态在满足了过渡的条件后便执行所指向的状态。

举一个简单的例子，两个状态【吃饭】和【没在吃饭】之间的过渡可以是是否【饱了】。当处于【没在吃饭】的状态中，且满足**并非**【饱了】的条件，便会执行【吃饭】的状态。当然这个【饱了】也可以是【到饭点了】或者什么其他的条件，具体的设置根据需要来就好。

[![Lwe6aQ.png](https://s1.ax1x.com/2022/04/18/Lwe6aQ.png)](https://imgtu.com/i/Lwe6aQ)

在Animator中还有三种自动生成的状态，这三种状态只影响状态机本身的循环。

* Entry（进入）

放入当前Layer的第一个动画或者新建的状态会自动生成一条由Entry指出的过渡，所指的状态将会作为默认状态播放。

* Any State（任意状态）

由Any State指出的过渡代表不论当前Layer在哪一个状态，只要满足过渡的条件，就会执行该过渡所指向的状态。

* Exit（退出）

指向Exit的过渡在满足条件后便会退出当前状态，并从Entry重新进入状态机。

# 数据类型

在Parameter里面可以添加的变量类型有三种：

* 布尔型/bool
* 浮点数/float
* 整数型/int

其中，布尔型可能的值只有两个，是（true）和否（false）；浮点数可能的值是从0到1的任何小数（比如0.1，0.57）；整数型则是任何整数（比如0，5）。

以上提到的三种数据类型在Animator的Parameters里面都有对应，除此外，Animator的Parameters里面还有一种叫做Trigger的参数类型，根据[Unity官方文档](https://docs.unity3d.com/Manual/AnimationParameters.html)的说明：

> Trigger是一种在被动画过渡消耗后被重置的布尔型。（A boolean parameter that is reset by the controller when consumed by a transition）

个人暂时还没有在VRChat的模型编辑修改中用到过Trigger类型，也没有看到别人用过，所以在这里仅做说明，如果有想法的话可以自己去尝试。

# Menu操作

在Menu中可以对变量进行的操作一共有六种：

* 按钮/Button
* 开关/Toogle
* 子菜单/Sub-Menu
* 二轴控制盘/Two Axis Puppet
* 四轴控制盘/Four Axis Puppet
* 百分比控制盘/Radial Puppet

按钮和开关的操作类似，但是按钮只有在菜单中按下时才会对变量进行控制，而开关则在下次操作钱会一直保持对变量的修改。

二轴和四轴的控制盘可以同时对多个变量进行控制。二轴控制盘控制两个变量，分别是水平和垂直，变量范围是-1到1的浮点数；四轴则控制上下左右四个变量，范围是0到1的浮点数。

百分比控制盘只能控制一个浮点数，范围是0到1。

# 上面几章意味着什么？

Menu是你在游戏中所互动的操作界面，总共提供六种类型的互动；而Parameter负责与Animator交流。你在游戏中与Menu的互动会修改Parameter中的值，进而控制Animator中的动画状态。

了解这些值和互动类型能够如何组合以达到一些常见的效果就是本文的目的之一，接下来将会举几个例子来运用上面所提到的变量以及互动类型。

## 例1：换装

换装是角色常见的，比较频繁的一种需求。类似的还有换发型，换瞳色等等，总结起来是在同类物品中选择其一。这种需求我们一般用整数型以及开关就可以简单的做到。

假设有A和B两套衣服，我们首先需要录制两套衣服的动画【A启用B禁用】和【A禁用B启用】（关于Write Default的讲解在[下面](#Write Default)）。在Animator中创建新的Layer和整数型Parameter，并将Layer权重设置为1。

将动画两段拖入Layer中，并创建由Any State指出的过渡到两段动画。设置过渡条件分别为前面创建的整数型等于0和1.