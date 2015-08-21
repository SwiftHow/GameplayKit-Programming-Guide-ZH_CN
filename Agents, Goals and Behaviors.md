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
