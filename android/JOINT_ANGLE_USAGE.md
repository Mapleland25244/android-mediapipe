# 關節角度計算功能使用說明

## 功能概述

已成功在 `GestureRecognizerHelper.kt` 中添加了計算手部關節彎曲角度的功能。

## 新增的函數

### 1. `calculateJointAngle(joint1, joint2, joint3): Double`
計算三個關節點之間的角度（joint2 是頂點）。

**參數：**
- `joint1`: 第一個關節點 (NormalizedLandmark)
- `joint2`: 中間關節點/頂點 (NormalizedLandmark)
- `joint3`: 第三個關節點 (NormalizedLandmark)

**返回：** 角度（度數）

**示例：**
```kotlin
val landmarks = result.landmarks()[0] // 獲取第一隻手的關節點
val angle = gestureRecognizerHelper.calculateJointAngle(
    landmarks[5],  // 食指 MCP
    landmarks[6],  // 食指 PIP
    landmarks[7]   // 食指 DIP
)
Log.d(TAG, "食指 PIP 關節角度: $angle 度")
```

### 2. `calculateFingerAngles(landmarks): Map<String, List<Double>>`
計算所有手指的所有關節角度。

**參數：**
- `landmarks`: 手部的 21 個關節點列表

**返回：** Map，鍵為手指名稱，值為該手指的關節角度列表

**手指關節對應：**
- **Thumb (拇指)**: [CMC-MCP-IP 角度, MCP-IP-TIP 角度]
- **Index (食指)**: [WRIST-MCP-PIP 角度, MCP-PIP-DIP 角度, PIP-DIP-TIP 角度]
- **Middle (中指)**: [WRIST-MCP-PIP 角度, MCP-PIP-DIP 角度, PIP-DIP-TIP 角度]
- **Ring (無名指)**: [WRIST-MCP-PIP 角度, MCP-PIP-DIP 角度, PIP-DIP-TIP 角度]
- **Pinky (小指)**: [WRIST-MCP-PIP 角度, MCP-PIP-DIP 角度, PIP-DIP-TIP 角度]

**示例：**
```kotlin
val landmarks = result.landmarks()[0]
val allAngles = gestureRecognizerHelper.calculateFingerAngles(landmarks)

allAngles.forEach { (finger, angles) ->
    Log.d(TAG, "$finger: ${angles.joinToString(", ") { "%.2f°".format(it) }}")
}

// 輸出示例:
// Thumb: 125.34°, 156.78°
// Index: 167.23°, 178.45°, 172.90°
// Middle: 165.12°, 175.34°, 170.56°
// Ring: 162.45°, 173.12°, 168.90°
// Pinky: 159.78°, 170.23°, 165.45°
```

### 3. `calculateSpecificFingerAngle(landmarks, fingerName, jointIndex): Double?`
計算特定手指的特定關節角度。

**參數：**
- `landmarks`: 手部的 21 個關節點列表
- `fingerName`: 手指名稱 ("thumb", "index", "middle", "ring", "pinky")
- `jointIndex`: 關節索引（0, 1, 2）

**返回：** 角度（度數）或 null（如果參數無效）

**示例：**
```kotlin
val landmarks = result.landmarks()[0]

// 獲取食指的第二個關節（PIP-DIP-TIP）角度
val indexAngle = gestureRecognizerHelper.calculateSpecificFingerAngle(
    landmarks, 
    "index", 
    2
)

if (indexAngle != null) {
    Log.d(TAG, "食指末端關節角度: %.2f°".format(indexAngle))
}
```

## 完整使用示例

### 在 Fragment 或 Activity 中使用

```kotlin
// 在 GestureRecognizerListener 的 onResults 回調中
override fun onResults(resultBundle: GestureRecognizerHelper.ResultBundle) {
    val result = resultBundle.results.firstOrNull() ?: return
    
    // 檢查是否檢測到手
    if (result.landmarks().isNotEmpty()) {
        val landmarks = result.landmarks()[0] // 第一隻手
        
        // 方法 1: 計算所有手指角度
        val allAngles = gestureRecognizerHelper.calculateFingerAngles(landmarks)
        
        // 顯示所有角度
        val angleText = buildString {
            allAngles.forEach { (finger, angles) ->
                append("$finger: ")
                append(angles.joinToString(", ") { "%.1f°".format(it) })
                append("\n")
            }
        }
        
        // 更新 UI
        activity?.runOnUiThread {
            binding.angleTextView.text = angleText
        }
        
        // 方法 2: 檢測特定手勢（例如：食指是否伸直）
        val indexAngles = allAngles["Index"] ?: return
        val isIndexStraight = indexAngles.all { it > 160 } // 所有關節角度大於 160 度
        
        if (isIndexStraight) {
            Log.d(TAG, "食指伸直！")
        }
        
        // 方法 3: 計算特定關節角度
        val thumbAngle = gestureRecognizerHelper.calculateSpecificFingerAngle(
            landmarks, 
            "thumb", 
            1
        )
        
        if (thumbAngle != null && thumbAngle < 90) {
            Log.d(TAG, "拇指彎曲: %.1f°".format(thumbAngle))
        }
    }
}
```

### 在 OverlayView 中顯示角度

```kotlin
// 在 OverlayView.kt 中添加
fun drawAngles(
    landmarks: List<NormalizedLandmark>,
    angles: Map<String, List<Double>>,
    canvas: Canvas
) {
    val paint = Paint().apply {
        color = Color.WHITE
        textSize = 30f
        style = Paint.Style.FILL
    }
    
    // 在食指 PIP 關節旁顯示角度
    val indexAngles = angles["Index"]
    if (indexAngles != null && indexAngles.size > 1) {
        val x = landmarks[6].x() * imageWidth
        val y = landmarks[6].y() * imageHeight
        canvas.drawText(
            "%.1f°".format(indexAngles[1]),
            x + 20,
            y,
            paint
        )
    }
}
```

## MediaPipe 手部關節點索引

```
0:  WRIST (手腕)
1:  THUMB_CMC (拇指腕掌關節)
2:  THUMB_MCP (拇指掌指關節)
3:  THUMB_IP (拇指指間關節)
4:  THUMB_TIP (拇指指尖)
5:  INDEX_FINGER_MCP (食指掌指關節)
6:  INDEX_FINGER_PIP (食指近端指間關節)
7:  INDEX_FINGER_DIP (食指遠端指間關節)
8:  INDEX_FINGER_TIP (食指指尖)
9:  MIDDLE_FINGER_MCP (中指掌指關節)
10: MIDDLE_FINGER_PIP (中指近端指間關節)
11: MIDDLE_FINGER_DIP (中指遠端指間關節)
12: MIDDLE_FINGER_TIP (中指指尖)
13: RING_FINGER_MCP (無名指掌指關節)
14: RING_FINGER_PIP (無名指近端指間關節)
15: RING_FINGER_DIP (無名指遠端指間關節)
16: RING_FINGER_TIP (無名指指尖)
17: PINKY_MCP (小指掌指關節)
18: PINKY_PIP (小指近端指間關節)
19: PINKY_DIP (小指遠端指間關節)
20: PINKY_TIP (小指指尖)
```

## 應用場景

1. **手勢識別增強**: 結合角度信息進行更精確的手勢分類
2. **康復訓練**: 監測手指彎曲角度，評估康復進度
3. **手語識別**: 分析手指姿態進行手語翻譯
4. **遊戲控制**: 基於手指彎曲度進行遊戲交互
5. **虛擬鍵盤**: 檢測手指彎曲來模擬按鍵輸入

## 注意事項

- 角度計算基於 3D 座標 (x, y, z)，更加準確
- 返回的角度範圍為 0° 到 180°
- 如果 landmarks 少於 21 個點，函數會返回空 Map 或 null
- 建議在生產環境中添加異常處理

