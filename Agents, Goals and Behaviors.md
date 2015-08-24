# 代理、目标和行为

在很多游戏中，实体移动具有实时性和一定程度的自主性，例如，在一个游戏中，玩家可以通过操作（*触摸或点击*）让自己游戏中的人物移动到想要的位置上，然后人物就可以自动的向目标位置移动，并且会绕过障碍物，最终到达目标位置。敌对单位则想要将玩家的角色拦截并进行攻击。友方单位可能与玩家并列飞行并保持一定的距离。在上述的这些情况中，每一个实体的运动过程可能会受到现实中的一些情况的约束，所以游戏中的角色在移动时需要根据实际的情况来改变移动方向。

GameplayKit的代理系统提供了一个高效的方法来实现自主移动。

`代理（*Agent*）`表示的是游戏实体，它在一个二维空间中运动的时候，可能会受到一些实际情况的限制，比如它的大小，位置，速度以及受到阻碍时的运动速度。（注意：虽然代理的运动是基于物理模型的，但是它不受SpriteKit和SceneKit的物理模拟的约束。）代理的行为是由促使它移动的目标确定的。

`目标（Goals）`表示随着时间的推进能够影响`代理（Agent）`的运动的一个独特的因素。下面是目标（Goals）的示例图：

图7-1：朝一个特定方向运动

![图7-1](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/agent_seek_2x.png)


图7-2：避开障碍物

![图7-2](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/agent_avoid_2x.png)


图7-3：和其他代理聚集在一起

![图7-3](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/agent_flock_2x.png)


图7-4：沿着一条路径移动

![图7-4](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/agent_follow_path_2x.png)


`行为（Behavior）`是将一个或者多个`目标（Goal）`组合在一起，以推动代理的运动。例如，一个行为可以将一个移动到指定位置的目标和一个绕过障碍物的目标组合在一起。当你在一个行为中将多个目标组合在一起时，你可以针对每个目标的权重进行分配，来决定他们的影响程度。


### 设计你的游戏代理

GameplayKit中的代理架构扩展了之前[实体和组件]()中讨论的实体-组件架构，即一个`GKAgent`对象是一个组件。当你在设计游戏时为你的每一个运动的角色都使用了实体，你可以简单的通过为每一个相关的实体添加`GKAgent`组件来创建机遇代理的行为，并且可以通过已经指定了一系列`GKGoal`对象的`GKBehavior`对象来配置它。对于任何其他的组件，可以在游戏循环中的每一个模拟步骤或者每一帧动画上调用`updateWithDeltaTime:`方法（直接调用，或者通过一个在游戏中管理所有代理的`GKComponentSystem`对象）。

>注意
>
>在使用代理时，也可以不使用实体组件架构。但是在这种情况下，你必须管理自己的一套`GKAgent`实例，并且为每个代理调用`updateWithDeltaTime:`方法。


每个`GKGoal`对象代表一个基础的、能重复使用的行为单位。当一个代理的`updateWithDeltaTime:`方法被调用时，它会评估每个和GKBehavior相关的目标，得到一个代表代理的旋转和速度改变需要的载体，以满足目标，换句话说，要满足在时间差和代理的最大速度限制内的目标。然后代理综合这些得到的载体，其中每一个目标按照`GKBehavior`对象中定义的权重比例进行调整后，应用到代理的位置、方向和速度上。

`GKGoal`对象可以在多个行为中被重复使用。例如，两个在游戏中敌对角色的代理可以共享一个相同的目标，比如一个拦截玩家代理的目标，但是他们其中之一可能会为了避开第三个代理而抵消这个目标。同样的，`GKBehavior`对象可以在多个代理中被重复使用。例如，几个在游戏中敌对角色的代理可能共享相同的行为，比如一个追逐玩家同时避开障碍物的行为，而且每个都可以无视当前的状态而独立移动以满足组成行为的目标。


### 在游戏中使用代理

在AgentsCatalog示例代码项目中，包含了几个SpriteKit的场景，这几个场景演示了有一个或者多个目标的代理的基本用法。

>注意
>
>这部分内容讨论了实例代码项目：[AgentsCatalog: Using the Agents System in GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/AgentsCatalog/Introduction/Intro.html#//apple_ref/doc/uid/TP40016141)，可以下载并在Xcode中查看。

例如，`Fleeing`场景（可以在`FleeingScene`类中找到）演示了如何使用代理和目标在一个角色试图躲避时，使另一个游戏人物跟随鼠标或者触屏位置移动。这个效果中使用了三个代理：

- `跟踪代理`，它没有在现实上的交互，但是其位置实在不断更新的，以匹配鼠标的点击/拖动事件（在OS X中）或者触摸开始/移动事件（在IOS中），如下代码所示：

	```swift
	- (void)touchesMoved:(nonnull NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event {
	    UITouch *touch = [touches anyObject];
	    CGPoint position = [touch locationInNode:self];
	    self.trackingAgent.position = (vector_float2){position.x, position.y};
	}
	```
	
- `玩家代理`，其唯一的目标就是追踪代理的当前位置：

	```swift
	self.seekGoal = [GKGoal goalToSeekAgent:self.trackingAgent];
	[self.player.agent.behavior setWeight:1 forGoal:self.seekGoal];
	```
	
- `敌对代理`，其唯一的目标就是逃离玩家代理的位置：

	```swift
	self.fleeGoal = [GKGoal goalToFleeAgent:self.player.agent];
	[self.enemy.agent.behavior setWeight:1 forGoal:self.fleeGoal];
	```
	

SpriteKit场景的`update:`方法调用了每个代理的`updateWithDeltaTime:`方法。每个代理更新它的位置和速度以实现其目标，并调用`agentDidUpdate:`方法，依次更新这些充当代理视觉交互的SpriteKit节点的位置和旋转。

>注意
>
>在这个例子中，跟踪代理的`updateWithDeltaTime:`方法永远不会被调用。因为它的位置总是在被鼠标或者触摸所控制，而GameplayKit代理模式永远不需要移动它。而且这个代理没有视觉交互，所以它并不需要委托。


在这个项目中还有其他的场景来演示其他种类的代理行为。例如，在`FlockingSence`类中演示了如何将分离，对齐和内聚目标组成一组代理来一起移动。



### 代理和寻路

GameplayKit包括两个系统用来帮助你管理游戏角色的移动：其中之一是代理，在这个章节我们已经讨论过了，另一个是寻路，我们在前一个章节也讨论过了（详情参见[寻路]()）。每个系统使用不同的方法来管理游戏中的移动。寻路需要在一个大规模的游戏世界，这样就可以提前很长时间并且很详细的规划移动的路线，然而代理模式注重的是在短时间内和小空间的游戏中的动态反应。根据你想实现的游戏功能，在这两种模式中，有可能其中一种比另一中更为合适，或者你也可以同时将两种模式结合在一起使用。

一种常见的模式是使用寻路来规划游戏角色的移动路线，然后使用代理模式来使角色按照规划的路线移动，可能同时还需要考虑其他的目标。`goalToFollowPath:maxPredictionTime:forward:`目标使代理从一个点沿着`GKPath`对象描述的路线向另一个点移动，并且`initWithGraphNodes:radius:`可以方便的从寻路操作的结果中创建一个`GKPath`对象。

DemoBots实例项目对这种模式进行了演示。为了使用寻路，游戏首先创建了一个`GKObstacleGraph`对象来表示游戏世界中的边界区域（不能到达的地方），如在前一章节的6-3代码片中。然后，当一个对敌的电脑角色需要追赶另一个角色的时候，游戏的`TaskBotBehavior`类会通过Graph找出移动的路线（在前一章节的6-4代码片有介绍）


>注意
>
>这部分内容讨论了实例代码项目：[DemoBots: Building a Cross Platform Game with SpriteKit and GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/DemoBots/Introduction/Intro.html#//apple_ref/doc/uid/TP40015179)，可以下载并在Xcode中查看。

寻路的结果在一个Graph节点数组中，在7-1代码片中的`addFollowAndStayOnPathGoalsForPath`方法从这些节点中创建了`GKGoal`对象，因此代理可以自动的沿着路线移动。

代码片7-1：使用寻路创建目标

```swift
private func addFollowAndStayOnPathGoalsForPath(path: GKPath) {
    // The "follow path" goal tries to keep the agent facing in a forward direction when it is on this path.
    setWeight(1.0, forGoal: GKGoal(toFollowPath: path, maxPredictionTime: GameplayConfiguration.TaskBot.maxPredictionTimeWhenFollowingPath, forward: true))
    
    // The "stay on path" goal tries to keep the agent on the path within the path's radius.
    setWeight(1.0, forGoal: GKGoal(toStayOnPath: path, maxPredictionTime: GameplayConfiguration.TaskBot.maxPredictionTimeWhenFollowingPath))
}
```

>注意
>
>在DemoBots项目中，`GameplayConfiguration`对象存放了影响游戏的全局常量

除了寻路行为，`TaskBotBehavior`类创建其他的目标并且将他们添加到敌对电脑角色的代理中。

- `behaviorAndPathPointsForAgent`方法将分离，对齐和连贯性的目标组合作为一个不重叠的单位，让一组机器人一起移动。

- `addAvoidObstaclesGoalForScene`方法添加一个目标，这个目标让一个机器人持续尝试穿过不可越过的区域。这个目标是十分必要的，因为代理通常无法精确的沿着多边形的路线移动，它的最大加速度（maxAcceleration），最大速度（maxSpeed），和其他的属性，以及其他的目标，这些都可能导致代理沿着一条大致接近自然光滑的路线移动，避开障碍的目标也会导致代理远离既定的路线。
