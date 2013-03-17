---
layout: post
title: "关于设计的一点思考"
date: 2013-03-17 15:37
comments: true
categories: [Thoughts, Design, Closure]
---
现在游戏里有类似于红警的建筑列表面板的那种需求，简单说就是有一个面板，里面可以显示若干个按钮，每个按钮有一个图标，鼠标指上去后会显示tip，左键点击按钮就会灰化同时进入建造该建筑的状态（此时当鼠标在场景内移动时就会有一个虚的建筑跟着移动）。UI同学三下五除二给了套这样的API：
{% codeblock lang:python %}
class BuildingPanel(Panel):
	def setIconList(self, iconList)
	def setBtnEnable(self, index, enabled)
	def asOnGetTip(self, index)  ＃ 鼠标悬停时的回调，请求tip内容
	def asOnBtnClicked(self, index)  # 鼠标点击的回调
{% endcodeblock %}

当然，这套API也能够完成任务，但是确提供了一套很别扭的语义(先不说命名问题)。一般我们会按照一个面板包含很多按钮，每个按钮可以有icon有tip这样层次化的去理解这个东西，但是，这套API带来的语义确是这个面板有一列icon，你可以设某个位置的按钮的使能。如果用数据库表来类比的话，那么这套API就是以表的每一列来向你讲述这个表而不是表有很多行，每行表示一个按钮。例如，如果初始化这个面板的时候要有一些按钮默认置灰，那么代码可能就得这么写
{% codeblock lang:python %}
allBuildingTypes = ..
icons = getAllBuildingIcons(allBuildingTypes)
enableStates = calcEnableStates(allBuildingTypes, otherArgs)

\# assume buildingPanel is the panel
buildingPanel.setIconList(icons)

for index, enabled in enumerate(enableStates):
	buildingPanel.setBtnEnable(index, enabled)
	
{% endcodeblock %}
这样的代码就很难是自解释的了，它淹没在具体的实现细节里了。另外，为实现asOnGetTip/asOnBtnClicked，BuildingPanel需要保存一个管理器类的对象，并调用其相关的函数，这引入了一些依赖。而管理器类也需要自己维护一个按钮索引到建筑类型的一个映射，以在处理tip或者点击时知道是哪个建筑，多了一些负担。

如果我们直接将文章开头的描述转换成代码，那首先需要抽象一个按钮：
{% codeblock lang:python %}
class ButtonData(object):
	def __init__(self, icon, callback, enabled, tip)
		self.icon = icon
		self.callback = callback  # called when clicked
		self.enabled = enabled
		self.tip = tip
{% endcodeblock lang:python %}

这样BuildingPanel就管理一些ButtonData就可以了：
{% codeblock lang:python %}
class BuildingPanel(Panel):
	def __init__(self):
		self.buttons = []  # [ButtonData]
{% endcodeblock lang:python %}

之前提到的初始化代码会变成这样子：
{% codeblock lang:python %}
allBuildingTypes = ..

def onClicked(buildingType, index)
	doSomething(buildingType)
	buildingPanel.setBtnEnable(index, False)

buttons = []
for buildingType in allBuildingTypes:
	buttons.append(ButtonData(
		buildingType.icon,
		functools.partial(onClicked, buildingType),
		buildingType.enabled,
		buildingType.tip
	))
	
buildingPanel.setButtons(buttons)
{% endcodeblock lang:python %}
（一些命名还比较蛋疼，先忍着）这样子，代码的实现算是层次比较分明了，也更具有自解释性。所以，API的设计，或者代码的每个层次，应该尽量提供一个比较自然的语义，而不是一些看起来能完成功能的随机产生物。同时，借助于直接的回调机制，管理类现在不需要去维护一个按钮索引和实际建筑的关系，BuildingPanel也不需要依赖管理类来处理鼠标点击或者tip显示问题，完全通过self.buttons自给自足，大家各自独立，不需要多知道彼此一点点。

我觉得，以callable（functions/callable class instance/closure/partial(bind)）为基础建立起来的回调机制，是使得模块划分清晰的有力武器，主动通知方不需要了解被通知方的任何性质，被通知方也不需要按照某种硬性interface规格来使自己适应，而且callable本身就包含了足够的信息进行处理（closure或者bind或者callable class instance的形式保存），各自可以说井水不犯河水。用duck type的意思来说，在主动通知方看来，你看起来是我要的callback，那你就是我要的callback，不管你是function还是closure还是instance还是什么。在Python里，我们可以轻松的用lambda或者functools.partial，或者复杂点自己写个定义了__call__的类。在最新的CPP11里，也融合了大量的函数式编程的内容，包括lambda/std::function/std::bind，让我们活得轻松点。据说java8里也有lambda什么的了，但是看起来笨笨的，没那么duck typing。。