# 实体与组件

众所周知，设计一个复杂的游戏软件需要对项目架构有良好的规划。我们会发现有一些架构设计的并不合理，随着你的游戏功能和内容越来越多，原本简单的游戏 Demo 却会变得难以维护。GameplayKit 提供给你一个架构方式让你一开始就能获得更好的可重用性，并且帮你分解了一些游戏开发中可能遇到的各种各样的问题。

在**实体与组件**这个结构中，**实体**可以是你的游戏相关的各类物体，它可以代表游戏中至关重要的角色，比如玩家或者敌人；可以代表一些仅仅存在于游戏世界中而不和玩家产生交互的物体，比如一个动态的装饰特效；它甚至可以是你游戏中的抽象概念或者 UI 元素，比如一个控制何时向游戏世界中添加新的敌人的管理器，或者一个管理玩家装备的系统。

一个实体通过成为一批**组件**的容器来获得所需的功能。而一个**组件**则负责处理一个实体的显示状态或者行为状态中的具体的一小部分功能。由于组件的功能本身是具体的，但组件并不和某个具体的实体进行绑定，所以你可以在不同的实体中重复使用某个相同的组件。

## 实体与组件模式的设计

实体与组件设计模式是一个支持使用组合方式多过使用继承方式的设计模式。为了说明这一点，我们假设要设计一款塔防游戏，包含以下功能：

- 玩家在游戏地图的某一边要保卫一个基地。
- 数波敌人周期性的从游戏地图的另一边出现，并向玩家基地按既定路线进发。如果有较多敌人到达基地，基地将被破坏，游戏就结束了。
- 为了保卫基地，玩家将在游戏地图的合理位置码放防御塔，这些塔将会自动向射程内的敌人射击。

这个设计可以利用继承方式简单实现。

- 创建一个`Base`类表示基地，显示基地状态（像是 SpriteKit 游戏中 SKSpriteNode 一样），并且记录承受了多少敌人的进攻。
- 创建`Enemy`类用来显示敌人，并控制敌人的移动。
- 创建`Tower`类来显示防御塔，并且添加缩地敌人进行射击等逻辑。

所有的这些类都需要处理当前对象的显示状态，所以这部分功能可以用一个父类`GameObject`实现，最终使用继承方式实现整个游戏结构。如图 3-1 所示。

**图 3-1** 基于继承的设计

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/entity_component_1_2x.png)

### 基于继承的架构阻碍了游戏的扩展和改进

如果你想要改进这个游戏，你可能会添加更多种类的敌人和不同的防御塔或者向游戏中的物体增加功能。比方说，你可能想要增加一些向防御塔反击的敌人，那么现在问题来了——这个新的`ShootingEnemy`类由于不是继承于`Tower`类，所以无法重用之前`Tower`类中实现的锁定目标并射击的代码逻辑。

**图 3-2** 基于继承的设计遇到的阻碍

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/entity_component_3_2x.png)

如果你继续这样改进游戏，最终你的基类将会包含几乎它子类的所需的所有功能，而子类自身却为自己提供了很少的功能。这将导致代码变得越来越复杂而且难以维护，因为父类的逻辑中需要先判断子类的的 ID，再决定是否可以继续做接下来的动作，同时我们向父类添加新的功能时需要尤其小心因为此时的父类包含的太多的状态和行为。

### 基于组合的架构将使游戏改进变得简单

实体与组件设计模式期望你去考虑游戏世界中每一个游戏元素做什么，而不是考虑它们是什么来组织游戏里的元素。在塔防游戏的例子中，你可以为基地，敌人或者防御塔设计不同的组件。这些组件包含不同的功能，有的可能负责处理是否显示；有的负责锁定目标和射击；有的负责移动；有的负责记录承受的伤害并且更新相应的 UI，等等。所以，首先我们需要基于`GKComponent`类为独立的小功能创建子类（组件）。

接着，我们可以基于`GKEntity`类创建组建的容器作为不同的游戏实体，而不是为这些游戏实体创建各自的类。如图 3-3 所示，一个`GKEntity`实例可以被当做一个防御塔，它将包含*显示*和*射击*组件。另一个`GKEntity`实例可以被认为是敌人，它将包含*显示*和*移动*组件。如果我们想要添加一个可以射击的敌人，只需要向一个`GKEntity`中添加*移动*和*射击*的组件就可以了。

**图 3-3** 实体与组件模式组合功能

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/entity_component_4_2x.png)

## 实体与组件模式的使用

就其本身而言，实体与组件设计模式将成为你组织代码的一个简单方法。而 GameplayKit 在这一设计模式基础上为游戏开发者们添加了另一个常用设计概念——周期性更新（periodic updates），这将为你的游戏构建提供更加有用的基础架构。

### 利用周期性更新驱动游戏

游戏引擎（如 SpriteKit，SceneKit，基于 Metal 或者 OpenGL 技术构建的其他自定义引擎）通常会对游戏运行的相关的代码始终保持一个循环，我们称之为*更新/渲染 周期*。在*更新*阶段，游戏将更新所有游戏逻辑相关的内部状态，而在*渲染*阶段，游戏引擎自动处理了一些诸如动画和物理的功能，而后基于当前的内部状态绘制并展现场景。通常，游戏引擎就是被设计为让开发者负责控制更新逻辑，让引擎本身来负责渲染部分——SceneKit 和 SpriteKit 就是这样设计的。

GameplayKit 中的 `GKEntity` 和 `GKComponent` 类利用了这一观念。每次当你游戏逻辑中的更新方法执行时（比如，SpriteKit 中的`update:`方法，或者 SceneKit 渲染代理其中的`renderer:updateAtTime:`方法），你可以通知那些控制游戏逻辑的组件，将消息派送到这些组件的`updateWithDeltaTime:`方法中。GameplayKit 提供了两种方法来处理该派发：

- **按实体更新（Pre-entity updates）** 在一个有许多实体的游戏中，你可以遍历游戏中的活跃组件列表，并调用它们的`updateWithDeltaTime:`方法。这时`GKEntity`类将转发更新消息到它所绑定的组件中去。
- **按组件更新（Pre-component updates）** 在一个更为复杂的游戏中，保证被实体使用的所有组件按照一个严格规定的顺序来更新，比一直追踪哪些实体包含那些组件来更新要更有效。这种情况下，你可以创建一些`GKComponentSystem`对象，每一个该类对象管理着某个具体类型的所有组件实例。之后，调用游戏中每个*组件系统*的`updateWithDeltaTime:`方法，此时*组件系统*将会向它所管理的所有组件实例转发更新消息，而不用考虑这些组件实例绑定的是那些实体。详情请见[GKComponentSystem Class Reference](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKComponentSystem_Class/index.html#//apple_ref/doc/uid/TP40015212)。

### 实体与组件设计的游戏示例

**迷宫（Maze）**项目是基于一些经典解密游戏实现的。在这个游戏中，玩家必须探索并走出迷宫。然而迷宫中有四个怪物（原文中为敌人）追杀玩家，玩家一旦被追到将会被传送到最初的入口处。

> 这个章节讨论的实例代码：[Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip)。请下载，并在Xcode中查看。

这个项目展示了*实体与组件*设计的作用（同时也说明了一些其他 GameplayKit 的功能）。在这个游戏中，玩家和怪物都是实体，不同类型的角色使用着不一样的组件集合。在示例代码中，你能发现三个`GKComponent`子类：

- `AAPLSpriteComponent`：这个组件类为包含它的实体管理一些游戏逻辑，并将这些逻辑转化为屏幕上的动作。另外这些组件只将迷宫理解为整数网格，并且负责同步实体精灵在网格中的位置，这些实体精灵的视觉渲染和动画则有 SpriteKit 完成。因此，无论是玩家角色的实体和怪物角色的实体都会使用这个组件。

  由于 *Sprite* 组件负责为游戏角色处理所有 SpriteKit 相关（或者说视觉渲染相关）的逻辑，所以其他的组件就不需要知道游戏场景的尺寸和动画等信息。如果将来游戏的视觉设计改变了（或者甚至转到另一个不同的渲染引擎中，如 SceneKit），其他的那些组件就不需要被重写了。

- `AAPLIntelligenceComponent`：这个组件类只被怪物角色的实体使用，它负责提供允许实体独立移动并对玩家的行为作出反应的逻辑。为了追踪在任何时候哪个怪物应该做什么，这个 *Intelligence* 组件使用了一个状态机（请查看 [State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1) 章节）。同时，一部分逻辑使用了规则系统（请查看 [Rule Systems](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1) 章节）用以抽象出导致怪物不同行为的条件。这个组件还使用了自动寻路（请查看 [Pathfinding](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Pathfinding.html#//apple_ref/doc/uid/TP40015172-CH3-SW1) 章节）来规划怪物在迷宫中的行进路线，同时调用 *Sprite* 组件来处理移动时精灵在屏幕上的表现。

  由于 *Intelligence* 组件是通用的，它可以被其他不同类型的实体（甚至不是怪物）重用。举个例子，将来这个游戏的一个变种可能会包含一个由电脑控制的盟友角色，它将在游戏中追杀怪物。

- `AAPLPlayerControlComponent`：这个组件只被玩家角色的实体使用，它负责将一些指向性输入（来源于玩家操作，如游戏遥控杆，键盘或者屏幕触摸事件等）转化为游戏世界中玩家角色的移动逻辑，并且同样调用 *Sprite* 组件来处理移动时精灵在屏幕上的表现。

  由于 *Control* 组件主导了实体在迷宫中受控制方向上的移动，它可以被其他类型的实体重用。比如，将来这个游戏的一个变种通过玩家分别使用不同的`AAPLPlayerControlComponent`实例来实现多人联机游戏。


虽然你可以直接使用`GKEntity`类（作为你自定义的`GKComponent`子类的容器），但是多个组件之间经常需要共享一些信息，比如它们提供了谁（哪个实体）的行为，等等。所以与其把这些信息放在组件中，不如创建`GKEntity`的子类。在*迷宫*游戏中，所有的组件类需要知道当前实体的位置信息，所以`AAPLEntity`类负责存放这些信息。
