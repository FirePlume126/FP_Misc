
# 杂项插件

* **插件未开源**

## 作者信息

Copyright FirePlume, All Rights Reserved.

Email: fireplume@126.com<br>
GitHub: [FirePlume126](https://www.github.com/FirePlume126)<br>
Bilibili: [火羽FP](https://space.bilibili.com/395084718)<br>
YouTube: [FirePlume126](https://www.youtube.com/@FirePlume126)

**[返回目录](https://www.github.com/FirePlume126/FP_Readme#Directory)**

* Misc：此类包含杂项插件
	- [FPFeatures](#fpfeatures)：包含简单的游戏功能模块
		- [FPInteraction](#fpfeatures-fpinteraction)：交互
		- [FPInventory](#fpfeatures-fpinventory)：库存
	- [FPEditorTools](#fpeditortools)：编辑器工具
		
<a name="fpfeatures"></a>
## FPFeatures

包含简单的游戏功能模块

<a name="fpfeatures-fpinteraction"></a>
### FPInteraction

交互

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPInteractionComponent|互动组件，添加此组件的APawn或APlayerController可使用互动功能|
|FPInteractionInterface|互动的目标必须继承此接口|
|FPInteractionWidget|显示互动提示的控件继承此类，并添加给`FPInteraction`项目设置|
|FPInteractionActor|互动的目标Actor继承此类，或者添加`FPInteractionInterface`接口|

* **调试指令**

|指令|描述|
|:-:|:-:|
|FP.Interaction.Debug.Interact (bool)|启用交互组件的调试|

<a name="fpfeatures-fpinventory"></a>
### FPInventory

库存系统

* **此模块的主要类**

|类名|描述|
|:-:|:-:|
|FPInventoryComponent|库存组件：添加此组件的`AActor`可存放物品|
|FPInventoryBackpackComponent|背包组件：主要用来同步操作到服务器，可以操作自己的背包和场景中的物品箱，添加建议给`APlayerState`|
|FPInventoryItemBase|物品基类|
|FPInventoryItemChest|物品箱|

* **使用指南**

1、给`APlayerState`添加`FPInventoryBackpackComponent`组件

2、物品箱继承`FPInventoryItemChest`或者给`AActor`添加`FPInventoryComponent`组件

3、继承`FFPInventoryItemInfo`创建物品的数据表格
![FPInventory_DataTable](https://github.com/FirePlume126/FP_Misc/blob/main/Images/FPInventory_DataTable.png)

4、将创建好的数据表格添加到`FPInventory`项目设置
![FPInventorySettings](https://github.com/FirePlume126/FP_Misc/blob/main/Images/FPInventorySettings.png)

5、调用`FPInventoryBackpackComponent`的函数使用库存系统
```c++
// 移动物品，在本地调用
// @param InSourceInventoryComp 源库存组件
// @param InSourceIndex 源库存物品索引
// @param InTargetIndex 此库存物品索引，InTargetIndex=-1时，自动分配位置
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "FPInventory")
void LocalTransferSlots(UFPInventoryComponent* InSourceInventoryComp, int32 InSourceIndex, int32 InTargetIndex = -1, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 拆分物品，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 拆分数量
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "FPInventory")
void LocalSplitStack(int32 InIndex, int32 InQuantity, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 丢弃物品到世界，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 输入-1时，全部丢弃
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "FPInventory")
void LocalDropItemToWorld(int32 InIndex, int32 InQuantity = -1, UFPInventoryComponent* InTargetInventoryComp = nullptr);

// 整理物品
// @param InTargetInventoryComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "FPInventory")
void LocalSortItems(UFPInventoryComponent* InTargetInventoryComp = nullptr);
```

<a name="fpeditortools"></a>
## FPEditorTools

编辑器工具

* 动画修改器
