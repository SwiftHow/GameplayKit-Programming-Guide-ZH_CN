## 关于 GameplayKit

> **重要** 
> 
> 这是一个用于 API 或技术开发的初步文档。苹果公司提供这些信息，通过描述一些技术要点和代码结构让你了解苹果相关产品的用法。此信息可能随时发生变化，根据这个文档的软件实现将会使用最终的操作系统和文档系统进行测试。较新版本的文档可能设有 API 或技术的未来 beta 版本。

GameplayKit 框架集合了为编写 iOS 和 OS X 游戏的基础工具和技术。与 SpriteKit 或 SceneKit 这些高级引擎不同的是，GameplayKit 不参与动画和视觉渲染等相关内容。相反，你可以使用 GameplayKit 来开发你的游戏机制，并利用最小的代价来设计模块化，可扩展的游戏架构。

## GameplayKit 功能

构建，改良和维护一个复杂的游戏需要一个精心策划的设计。GameplayKit 提供了*实体与组件*结构帮你设计更加可重用的游戏代码，提供了一个*状态机*系统来拆解开复杂的代码逻辑，同时还包含了许多随机化工具用以为多种游戏建立基础。

构建一个宏大的游戏也需要为常见的游戏机制实现复杂的逻辑算法。你可以直接使用 GameplayKit 已经实现的这些算法，而无需自己再去实现它们，这让你有更多的时间专注于制作你游戏中独一无二的功能。你可以使用*极小化极大策略*为对抗类的回合制游戏创建 AI 系统，你可以使用*寻路*功能为游戏角色在游戏世界中寻找路径，你还可使用*代理模拟*让你的游戏角色依据高级目标移动，或者你可以使用*规则系统*从代码中独立出游戏设计，并允许模糊逻辑推理。

2D 游戏使用 SpriteKit，3D 游戏使用 SceneKit，一些自定义或第三方的游戏引擎使用 Metal 和 OpenGL ES。因为 GameplayKit 是依赖 iOS 和 OS X 中这些高级游戏引擎的技术，你可以综合这些技术构建复杂的游戏应用。对那些图形绘制要求不高的游戏，你甚至可以在 iOS 的 UIKit 或者 OS X 的 AppKit 中使用 GameplayKit。

## 示例项目

文中介绍的示例游戏都提供示例代码，你可以下载每一个示例代码来查看 GameplayKit（以及其他技术）相关的内容：

- [Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip) 是一个经典的迷宫类游戏，使用了许多 GameplayKit 提供的游戏逻辑及方法。在 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1)，[State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1)，[Pathfinding](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Pathfinding.html#//apple_ref/doc/uid/TP40015172-CH3-SW1) 和 [Rule Systems](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/RuleSystems.html#//apple_ref/doc/uid/TP40015172-CH10-SW1) 等章节中有对该项目的讨论。
- [FourInARow: Using the GameplayKit Minmax Strategist for Opponent AI](https://developer.apple.com/library/prerelease/ios/samplecode/FourInARow/Introduction/Intro.html#//apple_ref/doc/uid/TP40016142) 是一个使用 UIKit 编写的简单的棋盘游戏（只有 iOS 版本），介绍了`GKMinmaxStrategist`类及相关协议的用法，在 [The Minmax Strategist](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Minmax.html#//apple_ref/doc/uid/TP40015172-CH2-SW1) 章节有更多讨论。
- [AgentsCatalog: Using the Agents System in GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/AgentsCatalog/Introduction/Intro.html#//apple_ref/doc/uid/TP40016141) 演示了`GKAgent`类的用法和*代理*在独立目标上的应用，以及如何将目标和复杂行为组合在一起的方法。[Agents, Goals, and Behaviors](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Agent.html#//apple_ref/doc/uid/TP40015172-CH8-SW1) 章节主要讨论这个项目。
- [DemoBots: Building a Cross Platform Game with SpriteKit and GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/DemoBots/Introduction/Intro.html#//apple_ref/doc/uid/TP40015179) 是一个功能完善的游戏，它使用了 GameplayKit 各个系统的功能，演示了如何使用 SpriteKit 设计并构建一个多关卡游戏，同时介绍了按需（on-demand）资源方案和 Xcode 7 中的新功能。[Agents, Goals, and Behaviors](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Agent.html#//apple_ref/doc/uid/TP40015172-CH8-SW1) 和 [Rule Systems](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/RuleSystems.html#//apple_ref/doc/uid/TP40015172-CH10-SW1) 章节讨论了这个项目的部分内容。

## 参考更多

示例中的游戏是基于 UIKit 和 SpriteKit 引擎构建的，并且GameplayKit 也能应用在 SceneKit 引擎构建的 3D 游戏中。所以在将 GameplayKit 应用到这些框架之前，你需要了解框架相关的概念和工具，并对如何为 iOS 或 OS X 编写游戏有大概的认识。查看下面的文档已获得关于你想要使用的游戏引擎技术的更多信息：

- [SpriteKit Programming Guide](https://developer.apple.com/library/prerelease/ios/documentation/GraphicsAnimation/Conceptual/SpriteKit_PG/Introduction/Introduction.html#//apple_ref/doc/uid/TP40013043) 和 [SpriteKit Framework Reference](https://developer.apple.com/library/prerelease/ios/documentation/SpriteKit/Reference/SpriteKitFramework_Ref/index.html#//apple_ref/doc/uid/TP40013041)
- [SceneKit Framework Reference](https://developer.apple.com/library/prerelease/ios/documentation/SceneKit/Reference/SceneKit_Framework/index.html#//apple_ref/doc/uid/TP40012283)
- [Start Developing iOS Apps Today](Start Developing iOS Apps Today) 和 [Xcode Overview](Xcode Overview)