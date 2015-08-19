# 状态机

在几乎所有的游戏中，游戏相关的代码逻辑往往高度依赖于当前游戏所处的状态。比方说，一个玩家角色的动画代码将随着他当时处于行走，跳跃还是站立状态而改变；一个敌方角色的移动代码也将随着他被赋予的 AI 智能的判断而改变，这个智能决定了他当时会去追逐一个弱小的玩家，还是逃离一个强大的玩家；甚至你的游戏在任何时候的每一帧里哪部分代码将被执行也随着游戏是否在运行，是否被暂定又或者是否处在菜单场景或者一些剪辑场景等等这些状态而改变。

当你开始编写一个游戏时，你可以简单的将所有的依赖状态变化的代码放在一个地方——比如说，放在一个 SpriteKit 游戏的每一帧的更新方法中。然而，当你的游戏不断地变大变复杂，这个方法将变得难以维护，也更难扩展。

更好的是，你可以有条理定义游戏中不同的状态，并且进一步规定状态间的切换规则。而这些定义被我们称之为状态机（State Machines）。这样，你可以将不同状态和相应的代码关联起来，那样在一个特定状态下，你就知道游戏的每一帧应该做什么，什么时候应该转化到另一个状态，以及转换状态过程中应该做什么动作。通过使用状态机来组织你的代码，你可以更加简单的区分游戏的复杂行为。

## 状态机的设计

状态机可以被应用在游戏中任何一个依赖状态行为的部分。下面的例子将会告诉你一些不同的状态机设计。

### 为动画设计的状态机

我们设想一个不断奔跑的游戏，在这类游戏中，玩家的角色将自动奔跑，玩家需要点击跳跃的按钮来跳过障碍。奔跑和跳跃有着不一样的动画，而不管是奔跑还是跳跃，角色的位置也需要不断更新。我们可以用一个包含三个状态的状态机来表示这个设计，如图 4-1 所示。

**图 4-1** 为动画设计的状态机

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/state_machine_1_2x.png)

- **奔跑** 在这个状态下，我们要循环播放奔跑动画。如果玩家按下了跳跃按钮，切换到跳跃状态，如果玩家越过了障碍，切换到落下状态。
- **跳跃** 开始进入这个状态时，开始播放一个声音特效，同时播放一次跳跃动画。在这个状态下，每一帧会根据重力将角色向上方移动一小段距离，当向上的速度变为零时，切换到落下状态。
- **落下** 开始进入这个状态时，开始播放一次下落动画。在这个状态下，每一帧会将角色向下移动一小段距离。玩家落地时，切换到奔跑状态，同时播放一次着陆动画和声音特效。

### 为敌方角色行为设计的状态机

再考虑另一个迷宫类游戏，游戏中有敌方角色追逐玩家角色。玩家角色有机会获得一个能力提升，期间玩家可以攻击敌方角色，被击杀的敌方角色将消失一段时间后重新刷新在迷宫中。每一个敌方角色将包含如图 4-2 所示的状态机实例：

**图 4-2** 为敌方角色行为设计的状态机

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/state_machine_2_2x.png)

- **追逐** 开始进入这个状态时，显示出正常的敌方角色。在这个状态下时，每一帧不断向玩家的位置更新自身的位置，如果玩家获得了一个能力提升，切换到逃跑状态。
- **逃跑** 开始进入这个状态时，敌方角色变得易于受到攻击。一段时间后，切换回追逐状态，然而如果被玩家击杀，则切换到击杀状态。
- **击杀** 开始进入这个状态时，显示一个动画特效并且增加玩家的得分。在这个状态下时，将所有的剩余敌方角色移动到迷宫中心位置，一旦到达，切换到重生状态。
- **重生** 在这个状态下，简单的记录一段时间，时间到达后，切换到追逐状态。

这个状态机在 [Building a Game with State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW3) 章节和*迷宫*游戏的示例项目中有更详细的说明

### 为游戏界面设计的状态机

在几乎所有的游戏中，游戏界面受游戏状态的影响，反之亦然。比方说，暂停一个游戏将显示出一个菜单，并且当游戏被暂停后，一些普通的游戏操作就不再起作用了。你可以利用图 4-3 所示的状态机来控制游戏中的界面变化。

**图 4-3** 为游戏界面设计的状态机

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/state_machine_3_2x.png)

- **游戏名界面** 应用打开后处于该状态，一旦进入该状态，游戏将显示一个游戏标题。在这个状态下，如果用户点击开始，将切换到游戏中状态。
- **游戏中** 在这个状态下，每一帧会调用负责游戏运行的更新方法。如果用户点击暂停，将切换到暂停状态。
- **暂停** 开始进入这个状态时，显示一个视觉效果提示玩家已经进入暂停状态，退出这个状态时将移除这个效果。（在这个状态下，我们没有必要暂停游戏运行逻辑，而是在游戏中状态下唤醒这部分逻辑。）如果用户点击继续，将切换到游戏中状态。
- **游戏结束** 开始进入这个状态时，显示一个界面来总结玩家的动作和得分，并且向这个界面中任何互动元素发送事件。如果用户点击重新游戏，将切换到游戏中状态。

这些状态机的设计可以轻易地被很多游戏扩展并使用。比如，额外菜单状态可以显示一些复杂的菜单；场景切换状态将播放场景动画而不再处理用户输入。

## 状态机的使用

在 GameplayKit 中，一个`GKStateMachine`类的实例就是一个状态机。我们为每一个状态创建一个`GKState`子类，用来定义状态所对应的动作逻辑，以及何时切入或者切出该状态。在某个特定时刻，一个状态机只会有一个特定状态。如果你需要对你的游戏物体的每一帧更新逻辑时（比如调用 SpriteKit 场景的`update:`方法，或者 SceneKit 的渲染代理的`renderer:updateAtTime:`方法），你可以调用状态机的`updateWithDeltaTime:`方法，这时状态机会调用当前状态下状态对象的相同方法。当你的游戏逻辑需要切换一个状态时，调用状态机的`enterState:`方法来选择一个新的状态。

**迷宫（Maze）**项目（已经在 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节被提及的）是基于一些经典解密游戏实现的。这个游戏分别使用不同的状态机实例来整体驱动每一个游戏中的敌方角色。一般情况下，怪物（敌方角色）追杀玩家，但是玩家一旦获得能力提升就可以击杀怪物，而这时怪物就会逃跑。而怪物被击杀后，它们会重新回到一个刷新点，并在一定时间后刷新。

> 这个章节讨论的实例代码：[Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip)。请下载，并在Xcode中查看。

### 定义状态和行为

这个状态机（指迷宫游戏）中的每一个状态都是`GKState`的一个子类，这个子类实现这个状态下的具体行为。该状态机包含四个状态子类：`AAPLEnemyChaseState`，`AAPLEnemyFleeState`，`AAPLEnemyDefeatedState`，和`AAPLEnemyRespawnState`。所有的四个子类都需要确保获得整个游戏世界的基本信息，所以这些子类都继承于一个`AAPLEnemyState`类，这个类定义了供状态类使用的属性和公用的初始化方法。代码 4-1 是这些类的定义：

**代码 4-1** 敌方角色状态定义

```oc

@interface AAPLEnemyState : GKState
@property (weak) AAPLGame *game;
@property AAPLEntity *entity;
- (instancetype)initWithGame:(AAPLGame *)game entity:(AAPLEntity *)entity;
// ...
@end
 
@interface AAPLEnemyChaseState : AAPLEnemyState
@end
 
@interface AAPLEnemyFleeState : AAPLEnemyState
@end
 
@interface AAPLEnemyDefeatedState : AAPLEnemyState
@property GKGridGraphNode *respawnPosition;
@end
 
@interface AAPLEnemyRespawnState : AAPLEnemyState
@end

```

这些状态类大多不需要额外的公共属性——他们需要的关于游戏世界的所有信息来自他们对主`AAPLGame`对象的引用。这个引用是一个弱（weak）引用，因为这些状态对象属于状态机，而这些状态机属于怪物，这些怪物又属于`Game`对象。

之后，每一个状态类通过重写`GKState`的`enter`，`exit`和`update`方法来定义该状态下的具体行为。下面的代码 4-2举例说明了敌方角色追逐状态的实现。

**代码 4-2** 追逐状态的实现

```oc

- (BOOL)isValidNextState:(Class __nonnull)stateClass {
    return stateClass == [AAPLEnemyChaseState class] ||
        	stateClass == [AAPLEnemyDefeatedState class];
}
- (void)didEnterWithPreviousState:(__nullable GKState *)previousState {
    AAPLSpriteComponent *component = (AAPLSpriteComponent *)[self.entity componentForClass:[AAPLSpriteComponent class]];
    [component useFleeAppearance];
 
    // 选择一个目标的坐标，并追赶。
    // ...
}
 
- (void)updateWithDeltaTime:(NSTimeInterval)seconds {
    // 如果怪物已经干掉当前目标，则寻找下一个目标。
    // ...
 
    // 向当前目标的位置追赶。
    [self startFollowingPath:[self pathToNode:self.target]];
}

```

当状态机切换到某个状态时，状态机将调用该状态对象的`didEnterWithPreviousState`方法。在逃跑状态中，这个方法使用游戏中的`SpriteComponent`类改变怪物的显示效果。（请查看 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节关于该类的讨论。）同时，这个方法也为怪物选择了一个迷宫中的随机位置用来逃跑。

动画每一帧都会调用`updateWithDeltaTime`方法（本质上，来自于 SpriteKit 场景中的`update:`方法）。在这个方法中，逃跑状态对象将持续的重新计算它和目标角色坐标的路径，并且控制怪物沿着该路径移动。（请查看 [Pathfinding](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Pathfinding.html#//apple_ref/doc/uid/TP40015172-CH3-SW1) 章节来深入讨论这个方法的实现。）

你可以通过重写每个状态类的`isValidNextState:`方法来执行当前状态的先决条件和不变因素。在迷宫游戏中，重生状态只会是由击杀状态切换而来，不会从追逐或者逃跑状态切换而来。因此，`EnemyRespawnState`类中的代码可以安全的假定所有击杀状态相关的效果都已经发生过了。

### 创建并驱动一个状态机

在你定义了`GKState`子类之后，你可以使用他们来创建状态机。在迷宫游戏中，`AAPLIntelligenceComponent`组件对象为每个怪物管理一个状态机。（请查看 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节了解更多关于基于组件设计的游戏。）建立一个状态机是简单地创建并配置每个状态类实例的问题，如代码 4-3 所示，建一个`GKStateMachine`实例包含这些状态对象：

**代码 4-3** 定义一个状态机

```oc

@implementation AAPLIntelligenceComponent
 
- (instancetype)initWithGame:(AAPLGame *)game enemy:(AAPLEntity *)enemy startingPosition:(GKGridGraphNode *)origin {
    self = [super init];
    if (self) {
        AAPLEnemyChaseState *chase = [[AAPLEnemyChaseState alloc] initWithGame:game entity:enemy];
        AAPLEnemyFleeState *flee = [[AAPLEnemyFleeState alloc] initWithGame:game entity:enemy];
        AAPLEnemyDefeatedState *defeated = [[AAPLEnemyDefeatedState alloc] initWithGame:game entity:enemy];
        defeated.respawnPosition = origin;
        AAPLEnemyRespawnState *respawn = [[AAPLEnemyRespawnState alloc] initWithGame:game entity:enemy];
 
        _stateMachine = [GKStateMachine stateMachineWithStates:@[chase, flee, defeated, respawn]];
        [_stateMachine enterState:[AAPLEnemyChaseState class]];
    }
    return self;
}
 
- (void)updateWithDeltaTime:(NSTimeInterval)seconds {
    [self.stateMachine updateWithDeltaTime:seconds];
}
 
@end

```

之后，游戏为每一个怪物（敌方角色）创建了`AAPLIntelligenceComponent`类的实例。为了支持每一帧调用每个状态类的更新逻辑，`AAPLIntelligenceComponent`类使用它的`updateWithDeltaTime:`方法来调用状态机上相应的方法。反之，状态机也会调用当前状态的更新方法——所以，当追逐状态被激活时，追逐状态的更新逻辑才会被执行。

每当游戏中的状态需要发生变化，状态机需要调用`enterState:`方法在状态之间切换。在迷宫游戏中，每个怪物的状态机从追逐状态开始，在一些特殊时候发生改变：

- 游戏中`AAPLGame`对象始终追踪玩家是否获得能力提升。当玩家角色获得或者失去能力提升时，`setHasPowerup:`方法将把所有怪物的当前状态改变为逃跑或者追逐状态，分别如下面的代码所示：

```oc

- (void)setHasPowerup:(BOOL)hasPowerup {
    static const NSTimeInterval powerupDuration = 10;
    if (hasPowerup != _hasPowerup) {
        // 根据玩家是否获得能力提升来选择切换到追逐还是逃跑状态
        Class nextState;
        if (!_hasPowerup) {
            nextState = [AAPLEnemyFleeState class];
        } else {
            nextState = [AAPLEnemyChaseState class];
        }
        // 使得每个怪物都切换到目标状态
        for (AAPLIntelligenceComponent *component in self.intelligenceSystem) {
            [component.stateMachine enterState:nextState];
        }
        self.powerupTimeRemaining = powerupDuration;
    }
    _hasPowerup = hasPowerup;
}

``` 

- 不但如此，`AAPLGame`对象仍然负责处理 SpriteKit 提供的物理碰撞。当玩家角色碰上怪物之后，游戏会首先根据怪物的当前状态来决定他们的碰撞的结果。如果怪物处于逃跑状态，玩家将杀掉怪物，于是怪物进入到击杀状态。

- `AAPLEnemyDefeatedState`类将一个被击杀的怪物移动到初始点，之后切换到重生状态。为了完成这个任务，该状态计算出到达初始点的路径，之后使用游戏中的`AAPLSpriteComponent`类生成一组`SKAction`动作序列。这组序列将怪物沿着路径移动，在所有的动作完成之后，怪物移动到路径的结尾，之后怪物切换到重生状态。

  请参考 [Pathfinding](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Pathfinding.html#//apple_ref/doc/uid/TP40015172-CH3-SW1) 和 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节来了解更多游戏中`AAPLSpriteComponent`类型的细节。

- `AAPLEnemyRespawnState`类将计算一段等待时间，让怪物重新回到迷宫之中。这个状态类使用`updateWithDeltaTime:`方法来检测是否经历一段时间并切换到追逐状态：

```oc

- (void)updateWithDeltaTime:(NSTimeInterval)seconds {
    self.timeRemaining -= seconds;
    if (self.timeRemaining < 0) {
        [self.stateMachine enterState:[AAPLEnemyChaseState class]];
    }
}

```