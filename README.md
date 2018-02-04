# Interactive Content with ARKit
# 用ARKit与内容交互

This sample demonstrates using SceneKit content with ARKit together to achieve an interactive AR experience.  
本示例程序使用了SceneKit和ARkit来共同实现交互式的AR体验.

It uses ARKit to display an animated chameleon in a camera view. You can interact with the chameleon by touching it, and you can move it to new locations by tapping at a place or dragging the chameleon. The model also reacts to the user's movement based on the relative position and proximity of the camera.
它使用ARKit在摄像机视图上显示出一个动态的变色龙.你可以通过触摸与它交互,并可通过点击或拖拽来移动变色龙.同时,模型还会根据用户的相对位置的移动不断变化.

## What this sample demonstrates
## 本示例演示了什么

This sample demonstrates the following concepts:  
本示例演示了以下概念:

* How to place an interactive animated SceneKit managed CG object (a chameleon) that you can interact with into a scene viewed through the device's camera.
* 如何通过设备的摄像头放置一个有交互动画的SceneKit版CG物体(一个变色龙),并与其中的场景视图产生交互.
* How to trigger and control animations of the object based on the user's movement and proximity.
* 如何根据用户的移动和接近来触发并控制物体的动画.
* How to use shaders to adjust the appearance of that object based on what the camera is seeing (see the `ARSCNView` extension in the file `Extensions.swift`).
* 如何使用着色器来根据相机中看到的来调整虚拟物体的外观(见项目中`ARSCNView`的扩展`Extensions.swift`)


## Implementation Details
## 实现细节

### Reacting on events in the renderer loop
### 渲染循环中响应事件
The interactive chameleon in this sample reacts to various events based on the rendering loop and interactions by the user. These actions are triggered in the following methods in `Chameleon.swift`:
本示例中的变色龙会对渲染循环及用户交互产生反应.这些动作是在`Chameleon.swift`中以下方法中触发的:

* `reactToInitialPlacement(in:)`: Called when a plane was detected and the chameleon is initially placed in the scene.
* `reactToInitialPlacement(in:)`: 当检测到平面,并且变色龙初次放置在场景中时调用.
* `reactToPositionChange(in:)`: Called when the user moved the chameleon to a new location.
* `reactToPositionChange(in:)`: 当用户将变色龙移动到新位置时调用.
* `reactToTap(in:)`: Called when the user touched the chameleon.
* `reactToTap(in:)`: 当用户触摸变色龙时调用.
* `reactToRendering(in:)`: Called at every frame at the beginning of a new rendering cycle. Used to control head and body turn animations based on the camera pose.
* `reactToRendering(in:)`: 每一帧的渲染循环开始时调用.用来根据摄像头的位置来控制头部与身体的转运动画.
* `reactToDidApplyConstraints(in:)`: Called at every frame after all constraints have been applied. Used to update the position of the tongue.
* `reactToDidApplyConstraints(in:)`: 每一帧的所有约束应用完成后调用.用来更新舌头的位置.

### Placing the object on a horizontal plane
### 将物体放置在水平面上
Plane detection is used to identify a horizontal surface on which the chameleon can be placed.  
平面检测是用来探测放置变色龙的平面的.

Once a plane has been found, the chameleon's transform is set to the plane anchor's transform in the `renderer(_:didAdd:for:)` method.  
当发现一个平面时,变色龙的变换矩阵就会在`renderer(_:didAdd:for:)`方法中被设置为等于平面锚点的变换矩阵.

For reasons of simplicity in this sample, the model has already been invisibly loaded into the scene since the beginning, and the node's `hidden` property is set to `false` to display it. In a more complex scene, you could asynchronously load the content when needed.  
为了简化代码,模型一开始就被加载到场景中了,只是不可见状态,将节点的`hidden`属性设置为`false`就可以显示出来.在更复杂的场景中,你可以根据需要来异步加载内容.

### Looking at the user
### 始终注视用户
The chameleon's eyes focus on the camera by a `SCNLookAtConstraint`. For a more natural saccadic eye movement, a random offset is additionally applied to the each eye's pose.  
通过使用`SCNLookAtConstraint`来让变色龙的眼睛始终对着摄像机.为了让眼睛动作更自然,给每只眼睛的位置添加了一个随机值.

### Moving the head and body to face the user
### 移动头部和身体来面对用户
In each frame, the chameleon's position in relation to the camera is computed (see `reactToRendering(in:)`) to determine whether the user is within the chameleon's field of view. In that case, the chameleon moves its head (with some delay) to look at the user (see `handleWithinFieldOfView(localTarget:distance:)`). In this method, it is also checked whether  
每一帧中,计算变色龙与摄像机之间的相对位置(见`reactToRendering(in:)`)来确定用户是否在变色龙的视场中.如果在,就移动头部(稍有延迟)来面对用户(见`handleWithinFieldOfView(localTarget:distance:)`).在这个方法中,还需要检测是否

* the user has reached a certain threshold distance. In that case, the chameleon's head movement follows the user closely without a delay ("target lock").
* 用户接近了某个距离阈值.如果接近了,变色龙的头部运动紧紧跟随用户,没有任何延迟("锁定目标").
* the user is within reach of the tongue. In that case, the chameleon opens the mouth and prepares for shooting the tongue.
* 用户进入了舌头的射程.进入后,变色龙张开嘴并准备射出舌头.

The head movement is realized by a `SCNLookAtConstraint`.  
头部运动是用`SCNLookAtConstraint`实现的.

If the camera pose is such that the chameleon cannot turn its head to face the user, a turn animation is triggered to obtain a better position (see `playTurnAnimation(_:)`).  
如果摄像机的位置太偏,变色龙的头转不到,会触发一个转身的动画来获得更好的位置(见`playTurnAnimation(_:)`).

### Shooting the tongue based on proximity
### 根据接近程度射出舌头
When shooting the tongue, it has to be ensured that it moves towards to user and sticks to the screen even when the camera moves. For that reason, the tongue's position must be updated each frame. This is done in `reactToDidApplyConstraints(in:)` to ensure that this happens after other animations, like head rotation, have already been applied.  
当射出舌头时,必须确保是朝向用户方向射出的,并且当摄像机移动时舌尖仍是粘在屏幕上.因此,舌头的位置必须每帧都更新.这些是在`reactToDidApplyConstraints(in:)`中完成的,以确保其他动画,如头部旋转,已经应用了.

### Adjusting the appearance based on what the camera is seeing
## 根据摄像机看到的场景调整外观
Chameleons can change color to adapt to the environment. This is done upon initial placement and when the chameleon is moved (see `activateCamouflage(_:)` and `updateCamouflage(_:)`).  
变色龙可以根据环境来改变颜色.这是在初次放置或位置被移动后完成的(见`activateCamouflage(_:)` 和 `updateCamouflage(_:)`)

The camouflage color is determined by retrieving an average color in a patch taken from the center of the current camera image (see `averageColorFromEnvironment(at:)` in `Extensions.swift`).  
伪装色是根据当前摄像头画面中间部分截取后,求取平均值来得到的(见`Extensions.swift` 中的 `averageColorFromEnvironment(at:)`).

The camouflage is then applied by modifying two variables in a Metal shader:  
随后,修改Metal着色器的两个变量,得到伪装色:

* `blendFactor` allows to blend between an opaque colorful texture, and a semitransparent texture which can be combined with a uniform color.
* `blendFactor` 允许混合一个透明的彩色纹理,和一个结合了全局颜色的半透明纹理.
* `skinColorFromEnvironment` sets the base color that shines through the transparent parts of the texture, creating a skin tone that is dominated by this color.
* `skinColorFromEnvironment` 设置透明纹理下的基础颜色,这个基础颜色构成皮肤的主导色.


## Useful Resources
## 有用的资源

* [ARKit Framework](https://developer.apple.com/documentation/arkit)

* [WWDC 2017 - Session 602, Introducing ARKit: Augmented Reality for iOS ](https://developer.apple.com/videos/play/wwdc2017/602/)

## Requirements
## 要求

### Running the sample
### 运行示例代码

* For plane detection to work, you will need to view a flat and sufficiently textured surface through the on device camera while this sample is running.
* 要让平面检测正常工作,你需要在本示例运行时,通过摄像头找到一块平整的,带有明显纹理的表面.
* Try out differently colored surfaces to see the chameleon change its color.
* 试试不同颜色的表面,看变色龙会改变颜色.

### Build
### 构建

Xcode 9 and iOS 11 SDK

### Runtime
### 运行时

iOS 11 or later

ARKit requires an iOS device with an A9 or later processor.

Copyright (C) 2017 Apple Inc. All rights reserved.
