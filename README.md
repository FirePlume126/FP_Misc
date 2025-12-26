
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
	- [FPFeatures](#fpfeatures)：包含简单的游戏功能模块：[交互](#fpfeatures-interaction)和[库存](#fpfeatures-inventory)
	- [FPEditorTools](#fpeditortools)：编辑器工具
		
<a name="fpfeatures"></a>
## FPFeatures

包含简单的游戏功能模块：[交互](#fpfeatures-interaction)和[库存](#fpfeatures-inventory)

<a name="fpfeatures-interaction"></a>
### 交互

|类名|描述|
|:-:|:-:|
|FPInteractionComponent|互动组件，添加此组件的`APawn`或`APlayerController`可使用互动功能|
|FPInteractionInterface|互动接口，互动的目标必须继承此接口|
|FPInteractionWidget|互动控件，显示互动提示的控件继承此类，并添加给[项目设置](#fpfeatures-settings)的`DefaultInteractWidgetClass`|

* **调试指令**

|指令|描述|
|:-:|:-:|
|FP.Interaction.Debug.Interact (bool)|启用交互组件的调试|

<a name="fpfeatures-inventory"></a>
### 库存

|类名|描述|
|:-:|:-:|
|FPInventoryComponent|库存组件：添加此组件的`AActor`可存放物品|
|FPInventoryBackpackComponent|背包组件，玩家使用的库存组件，继承`FPInventoryComponent`，主要用来同步操作到服务器，玩家可以通过组件操作任何背包，建议添加给玩家的`APawn`或`APlayerState`|
|FPInventoryItemInterface|库存物品接口(可选)，调用函数`UFPInventoryComponent::DropItemToWorld()`放置物品时建议添加此接口获取实体信息|

* **使用指南**

1、给玩家的`APawn`或`APlayerState`添加`FPInventoryBackpackComponent`组件

2、给物品箱等需要库存的实体添加`FPInventoryComponent`组件

3、根据项目需求继承`FFPInventoryItemTableRowBase`添加更多物品属性，然后创建物品的数据表格。并添加给[项目设置](#fpfeatures-settings)的`ItemDataTable`
```c++
// 物品表行基类，用于创建数据表格，可以继承此结构体添加更多属性
USTRUCT(BlueprintType)
struct FFPInventoryItemTableRowBase : public FTableRowBase
{
	GENERATED_BODY()

public:

	// 物品名称
	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta = (DisplayPriority = 0), Category = "Inventory")
	FText Name = FText::GetEmpty();

	// 物品类
	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta = (DisplayPriority = 1), Category = "Inventory")
	TSoftClassPtr<AActor> Class = nullptr;

	// 物品重量
	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta = (DisplayPriority = 2), Category = "Inventory")
	float Weight = 0.0f;

	// 物品堆叠数量
	UPROPERTY(EditAnywhere, BlueprintReadWrite, meta = (DisplayPriority = 3), Category = "Inventory")
	int32 StackSize = 1;
};
```

4、调用函数`UFPInventoryComponent::InitInventory()`初始化库存组件，调用`UFPInventoryComponent`的函数使用库存系统，绑定委托`OnUpdateInventory`更新库存UI
```c++
// 库存更新委托
UPROPERTY(BlueprintAssignable, Category = "Inventory")
FFPInventoryUpdateDelegate OnUpdateInventory;

// 初始化库存
// @param NewItems 物品ID和数量
// @param bSort 为true时，会自动整理库存
UFUNCTION(BlueprintCallable, Category = "Inventory")
void InitInventory(const TArray<FFPInventoryItem>& NewItems, bool bSort = false);

// 添加物品
// @param InItem 物品ID和数量
// @return 返回0代表全部加入库存，返回大于0的值代表库存已满多出的数量
UFUNCTION(BlueprintCallable, Category = "Inventory")
int32 AddItems(const FFPInventoryItem& InItem);

// 移除物品
// @param InIndex 物品位置索引
// @param InQuantity 物品数量，输入的值小于1时，全部移除
UFUNCTION(BlueprintCallable, Category = "Inventory")
void RemoveItems(int32 InIndex, int32 InQuantity = -1);

// 清空库存
UFUNCTION(BlueprintCallable, Category = "Inventory")
void ClearInventory();

// 移动物品
// @param InSourceComp 源库存组件
// @param InSourceIndex 源库存物品索引
// @param InTargetIndex 此库存物品索引，输入的值小于0时，自动分配位置
UFUNCTION(BlueprintCallable, Category = "Inventory")
void TransferSlots(UFPInventoryComponent* InSourceComp, int32 InSourceIndex, int32 InTargetIndex = -1);

// 拆分物品
// @param InIndex 物品位置索引
// @param InQuantity 拆分数量
UFUNCTION(BlueprintCallable, Category = "Inventory")
void SplitStack(int32 InIndex, int32 InQuantity);

// 整理物品
UFUNCTION(BlueprintCallable, Category = "Inventory")
void SortItems();

// 丢弃物品到世界
// @param InIndex 物品位置索引
// @param InSpawnTransform 在世界中生成的变换
// @param InQuantity 输入的值小于1时，全部丢弃
UFUNCTION(BlueprintCallable, Category = "Inventory")
AActor* DropItemToWorld(int32 InIndex, const FTransform& InSpawnTransform, int32 InQuantity = -1);

// 设置库存总负重
UFUNCTION(BlueprintCallable, Category = "Inventory")
void SetTotalWeight(float InlWeight);

// 获取库存所有物品
UFUNCTION(BlueprintCallable, Category = "Inventory")
TArray<FFPInventoryItem> GetInventoryItems() const;

// 获取库存插槽的物品
// @param InIndex 库存插槽
// @param OutItem 输出物品信息
UFUNCTION(BlueprintCallable, Category = "Inventory")
bool GetItemByIndex(int32 InIndex, FFPInventoryItem& OutItem) const;

// 获取当前重量
UFUNCTION(BlueprintCallable, Category = "Inventory")
float GetCurrentWeight() const;

// 获取库存总负重
UFUNCTION(BlueprintCallable, Category = "Inventory")
float GetTotalWeight() const;
```

5、联机环境下，在客户端调用`FPInventoryBackpackComponent`的函数使用库存系统，已完成网络同步
```c++
// 添加物品，在本地调用
// @param InItem 物品ID和数量
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_AddItems(const FFPInventoryItem& InItem, UFPInventoryComponent* InTargetComp = nullptr);

// 移除物品，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 物品数量，输入的值小于1时，全部移除
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_RemoveItems(int32 InIndex, int32 InQuantity = -1, UFPInventoryComponent* InTargetComp = nullptr);

// 清空库存，在本地调用
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_ClearInventory(UFPInventoryComponent* InTargetComp = nullptr);

// 移动物品，在本地调用
// @param InSourceComp 源库存组件
// @param InSourceIndex 源库存物品索引
// @param InTargetIndex 此库存物品索引，输入的值小于0时，自动分配位置
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_TransferSlots(UFPInventoryComponent* InSourceComp, int32 InSourceIndex, int32 InTargetIndex = -1, UFPInventoryComponent* InTargetComp = nullptr);

// 拆分物品，在本地调用
// @param InIndex 物品位置索引
// @param InQuantity 拆分数量
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_SplitStack(int32 InIndex, int32 InQuantity, UFPInventoryComponent* InTargetComp = nullptr);

// 整理物品，在本地调用
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_SortItems(UFPInventoryComponent* InTargetComp = nullptr);

// 丢弃物品到世界，在本地调用
// @param InIndex 物品位置索引
// @param InSpawnTransform 在世界中生成的变换
// @param InQuantity 输入的值小于1时，全部丢弃
// @param InTargetComp 要操作的库存组件，输入nullptr时，默认为玩家背包
UFUNCTION(BlueprintCallable, Category = "Inventory")
void Local_DropItemToWorld(int32 InIndex, const FTransform& InSpawnTransform, int32 InQuantity = -1, UFPInventoryComponent* InTargetComp = nullptr);
```

<a name="fpfeatures-settings"></a>
### 项目设置

![FPFeatures_Settings](https://github.com/FirePlume126/FP_Misc/blob/main/Images/FPFeatures_Settings.png)

```c++
// 默认互动控件类
UPROPERTY(config, EditAnywhere, Category = "Config|Interaction")
TSoftClassPtr<UFPInteractionWidget> DefaultInteractWidgetClass = nullptr;

// 物品数据表格
UPROPERTY(config, EditAnywhere, Category = "Config|Inventory")
TSoftObjectPtr<UDataTable> ItemDataTable = nullptr;

// 掉落物品生命周期，小于等于0表示永久存在
UPROPERTY(config, EditAnywhere, Category = "Config|Inventory")
float DroppedItemLifeSpan = 300.0f;

// 掉落物品类，为空则使用物品数据表中配置的默认类
UPROPERTY(config, EditAnywhere, Category = "Config|Inventory")
TSoftClassPtr<AActor> DroppedItemClass = nullptr;
```

<a name="fpeditortools"></a>
## FPEditorTools

编辑器工具

|类名|描述|
|:-:|:-:|
|FPEditorToolsAnimModifier_CalculateRotation|计算旋转的动画修改器|
|FPEditorToolsAnimModifier_CopyCurves|复制曲线的动画修改器|
|UFPEditorToolsAnimModifier_CreateCurves|创建曲线的动画修改器|
|UFPEditorToolsAnimModifier_CreateLayeCurves|创建图层曲线的动画修改器|
