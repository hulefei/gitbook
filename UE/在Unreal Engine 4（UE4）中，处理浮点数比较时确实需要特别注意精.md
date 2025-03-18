# UE 浮点数比较

在 Unreal Engine 4（UE4）中，处理浮点数比较时确实需要特别注意精度问题。以下是针对你提出的问题的分步解答和总结：

---

## **1. 浮点数比较的通用原则**

浮点数因精度限制无法直接使用 `==` 或 `!=` 进行精确比较，应采用**误差范围（Tolerance）**。UE4 提供了以下常用方法：

---

## **2. UE4 中的工具函数**

### **(1) `FMath::IsNearlyEqual`**

• **作用**：判断两个浮点数是否在指定误差范围内接近。
• **函数原型**：

```cpp
static bool FMath::IsNearlyEqual(float A, float B, float Tolerance = KINDA_SMALL_NUMBER);
```

• **默认误差**：`KINDA_SMALL_NUMBER`（约 `1e-4`）。
• **示例**：

```cpp
if (FMath::IsNearlyEqual(A, B)) {
  // A和B在默认误差范围内相等
}
```

### **(2) `FMath::IsNearlyZero`**

• **作用**：判断浮点数是否接近零。
• **函数原型**：

```cpp
static bool FMath::IsNearlyZero(float Value, float Tolerance = KINDA_SMALL_NUMBER);
```

• **示例**：

```cpp
if (FMath::IsNearlyZero(Vector.Size())) {
    // 向量的长度接近零
}
```

---

## **3. 蓝图中的等效节点**

在蓝图中，可通过以下节点实现类似功能：
• **"Nearly Equal (Float)"**：判断两个浮点数是否接近，默认误差为 `1e-4`。
• **"Nearly Equal (Vector)"**：判断向量各分量是否接近。
• **误差调整**：通过 **"Tolerance"** 引脚自定义误差范围。

---

## **4. 比较操作的注意事项**

### **(1) 大小比较（`>` 或 `<`）**

• 直接使用 `>` 或 `<` 可能因微小误差导致误判。建议先检查是否接近：

```cpp
bool IsAGreaterThanB(float A, float B, float Tolerance = KINDA_SMALL_NUMBER) {
      return (A - B) > Tolerance; // 仅当A明显大于B时返回true
}
```

### **(2) 特殊值处理**

• **NaN/Infinity**：UE4 的函数（如 `FMath::IsNaN`）可检测特殊值：

```cpp
if (FMath::IsNaN(A) || FMath::IsInf(A)) {
    // 处理异常值
}
```

---

## **5. 误差范围的选择**

• **默认值**：`KINDA_SMALL_NUMBER`（`1e-4`）适用于大多数情况。
• **动态调整**：对于大范围数值（如坐标比较），可能需要放大误差，例如使用 `1e-3`。
• **物理模拟**：涉及速度或力的计算可能需要更小的误差（如 `1e-6`）。

---

## **6. 示例代码**

• **默认值**：`KINDA_SMALL_NUMBER`（`1e-4`）适用于大多数情况。

```cpp
// 判断位置是否到达目标点
bool IsAtTargetPosition(FVector CurrentPos, FVector TargetPos) {
    return FMath::IsNearlyEqual(CurrentPos.X, TargetPos.X, 0.1f) &&
           FMath::IsNearlyEqual(CurrentPos.Y, TargetPos.Y, 0.1f);
}

// 判断浮点数是否明显大于
bool IsSignificantlyGreater(float A, float B, float Tolerance = 1e-4) {
    return (A - B) > Tolerance;
}
```

---

## **7. 文档参考**

• **官方文档**：[FMath 类](https://docs.unrealengine.com/5.0/en-US/API/Runtime/Core/Math/FMath/)
• **最佳实践**：避免在关键逻辑中依赖绝对浮点相等，优先使用误差范围。

通过合理使用 `FMath` 函数和蓝图节点，可有效规避浮点数精度问题，确保逻辑的健壮性。
