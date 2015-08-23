# 规则系统

很多游戏包含复杂的规则设定。例如，一个回合制的角色扮演游戏就可能包括以下的规则：当对立的角色进入同一个空间时会发生什么，谁将在接下来的战斗中占上风？或者说角色是否有机会去攻击对方？当其他角色试图攻击或者和参战者对战，会发生什么？这些规则的设定会变得非常复杂，以至于编程语言里的条件逻辑语句变得非常笨重。

一些非常有趣的游戏会包含突发行为系统，依附于一些简单规则的简单实体，它们之间的相互作用会在整个系统中呈现出有趣的模式。例如，单个敌人角色在动作游戏中的目标可能取决于人物的血量，敌人如何看到玩家角色，有多少其他的敌对角色在附近，玩家如何经常击败敌人，以及其他因素。总之，在游戏中，这些因素会使得游戏变得更加逼真，一些敌人会成群的攻击玩家，有些则会逃跑。

在 GameplayKit 中，规则系统解决了这两个问题。通过把游戏逻辑中确定的基本部分抽象成数据，规则系统能够帮助你把游戏分解成功能性的、可重复的、可扩展的块。通过合并模糊的逻辑，规则系统能够把角色的行为判断变为连续的变量而不是离散的状态，即使是简单的规则组合也能完成完成复杂的动作。


### 设计规则系统
GameplayKit 包含了两个主要的类来构建规则系统：[GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 和 [GKRuleSystem](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/cl/GKRuleSystem)

* `GKRule`：代表基于外部状态而作出的具体决定；

* `GKRuleSystem`：计算一系列对应状态数据的规则来决定一系列的事实；


### 规则作出基本决定
一个 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 对象有两部分组成：谓词和动作。规则的谓词是产生决策的部分－它计算一组状态信息，并返回一个 Boolean 值。规则的动作是仅当谓词为 true 时触发的代码。GameplayKit 通常在系统规则的上下文里计算。通常，规则的谓词检测被规则系统维护的状态数据，它的作用是改变系统的状态，或者决定规则系统输出的事实部分。

由于规则计算系统的状态并且改变系统的性能，所以它不需要本地状态数据。这样的设计能让你写出单独的规则作为功能性模块，他有定义良好的行为，并且在任何规则系统中能够重复使用


### 一个规则系统使用规则和状态决定事实

一个 [GKRuleSystem](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/cl/GKRuleSystem) 对象有三个关键部分：`规则议程`，`状态数据`和`事实`。

- `议程`。将一组规则添加到规则系统对象的 [agenda](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instp/GKRuleSystem/agenda) 中（ [addRule:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/addRule:) 或[addRulesFromArray:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/addRulesFromArray:)方法）。默认情况下，系统会按照他们添加到议程中的顺序来计算规则，这样做能够确保某些规则始终在其他规则前面计算，改变这些规则的显著性。

- `状态`。规则系统的 [state](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/instp/GKRule/salience) 字典包含的信息规则可以根据其进行测试。状态字典可以为你的规则设置提供任何有用的参考，例如你游戏模式中的字符串、数字或者自定义类。你如何编写规则决定了这些规则将使用数据结构。

- `事实`。事实表示从系统中的规则评估得出一个结论，并且可以是任何类型的对象，通常是像字符串这样简单的数据对象就足够了，从游戏模型中定义对象也可以被使用。一个事实的隶属度是一个数字，决定了它在事实的系统设置存在。1.0 级的事实包括，但零级的事实并非如此。当一个规则系统评估其规则，该规则操作可以确定一个事实，将其添加到系统中，或收回一个事实，从系统中删除。

然而，等级不一定是一个简单的二进制状态，一个事实可以有 0.0 和 1.0 之间的任何等级。通过让事实来改变等级，你可以使用规则系统来定义模糊逻辑，这种规则会生成具有可变度，真值，或可信水平的结论。当规则产生或回收一个事实时，它会增加或减去事实的等级。

你的游戏就可以利用各种事实的相对力度来创造游戏效果。例如，一个敌人是否会去攻击玩家可能基于一个“简单”的事实，它的的等级会基于几个规则。如果玩家距离很近，那么有一个规则会增加它的等级。如果玩家血量是满的，一个规则可能会减少这个等级，当然还有其他更多的规则。你也可以建立一个规则系统，在这个系统中一些规则用来产生事实，其他规则用来评估这些事实的等级以用来产生和回收更多的结论。你甚至能使用系统中事实的等级去改变相应相应因素的影响----例如，你可以把 [GKBehavior](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKBehavior_Class/index.html#//apple_ref/occ/cl/GKBehavior) 对象中收集的目标权重同相应事实的等级联系起来。

### 用规则系统建立游戏

GameplayKit 提供了三种方法创建 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 对象。你所选择的方法取决于你想如何将规则系统融入到你的开发当中。

- [ruleWithPredicate:assertingFact:grade:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/clm/GKRule/ruleWithPredicate:assertingFact:grade:) 和 [ruleWithPredicate:retractingFact:grade:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/clm/GKRule/ruleWithPredicate:retractingFact:grade:)这个方法创建的规则，它的谓词是由一个 [NSPredicate](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSPredicate_Class/index.html#//apple_ref/occ/cl/NSPredicate) 对象决定的，它的作用是硬编码的生成或者回收一个具体的事实。因为`NSPredicate`类能够描述条件逻辑然后通过关键路径访问数据，并且规则的作用是在创建时确定的，所以这样创建的规则能够很容易的归档。这样的设计能够很好的配合数据驱动的工作流，游戏设计者可以在不写代码的情况下去调整游戏的规则系统，（你甚至可以使用 OS X 的`NSRuleEditor`类去支持在运行时编辑规则。）但是这些规则产生和回收的事实拥有恒定的等级，所以他他们不适用于事实等级连续变化的逻辑模糊的系统中。

- [ruleWithBlockPredicate:action:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/clm/GKRule/ruleWithBlockPredicate:action:)
这个方法从两块创建一个规则，一个判断块去计算规则并返回一个 Boolean
 结果，一个动作快在判断去true 时被调用。这个方法允许你在代码中快速简单的创建规则，也包括那些逻辑模糊的规则。然后，生成的规则不是可存档的—他们只存在于游戏的内存对象图中。

- 你可以使用 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 来创建你的规则类，用 [evaluatePredicateWithSystem:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/instm/GKRule/evaluatePredicateWithSystem:)        方法来决定规则的谓词，用 [performActionWithSystem:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/instm/GKRule/performActionWithSystem:) 方法来执行一个动作。或者你可以使用 [GKNSPredicateRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKNSPredicateRule_Class/index.html#//apple_ref/occ/cl/GKNSPredicateRule) 子类，它使用 [NSPredicate](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSPredicate_Class/index.html#//apple_ref/occ/cl/NSPredicate) 对象来决定一个规则的谓词，你只需要实现规则动作的执行代码。因为规则的关键属性是在子类中实现的，所以这种方法生成规则的灵活性和复用性取决于你如何设计这些类。

在完成你的规则设计后，建立和使用规则系统需要四个基本步骤：

1. 初始化时，创建一个 [GKRuleSystem](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/cl/GKRuleSystem) 对象并填充规则列表

2. 设置规则系统的 [state](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instp/GKRuleSystem/state) 字典来包含任何游戏规则应采取的。根据你如何构造这个信息，你可以在初始化时设置状态，来包含游戏进程中内部状态变化的对象。或者，你可以设置状态来包含静态的对象，然后在你需要计算规则系统的时候更新这些状态来检查当前的游戏状态。

3. 当需要计算规则时，调用规则系统中的 [evaluate](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/evaluate) 方法。如果你周期性的计算游戏规则系统，记得在计算之前调用 [reset](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/reset) 方法来清理之  前的计算结果。

4. 在计算规则系统之后，对其所指定的事实进行检验，然后得出结论，并选择一个行动的过程。

下面的章节讨论了两个游戏中的规则系统的使用实例。

### 一个游戏效果的基本规则系统

这是一个迷宫的示例代码，它有一个简单的规则系统。在这个游戏中（基于几款经典街机游戏的改变， 拥有其他在 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1), [State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1), 和 [Pathfinding](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Pathfinding.html#//apple_ref/doc/uid/TP40015172-CH3-SW1) 章节讨论过的特点），这个简单的规则系统会决定敌对角色在通常状态下的行为特征。代码 8-1 展示了规则系统的创建和使用的相关模块。

> 这个章节讨论的实例代码：[Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip)。请下载，并在Xcode中查看。

**代码 8-1** 最小规则系统

```oc
- (instancetype)initWithGame:(AAPLGame *)game entity:(AAPLEntity *)entity {
    self = [super initWithGame:game entity:entity];
    if (self) {
        // 1
        _ruleSystem = [[GKRuleSystem alloc] init];
        NSPredicate *playerFar = [NSPredicate predicateWithFormat:@"$distanceToPlayer.floatValue >= 10.0"];
        [_ruleSystem addRule:[GKRule ruleWithPredicate:playerFar assertingFact:@"hunt" grade:1.0]];
        NSPredicate *playerNear = [NSPredicate predicateWithFormat:@"$distanceToPlayer.floatValue < 10.0"];
        [_ruleSystem addRule:[GKRule ruleWithPredicate:playerNear retractingFact:@"hunt" grade:1.0]];
    }
    return self;
}
 
 
- (void)updateWithDeltaTime:(NSTimeInterval)seconds {
    // 2
    NSUInteger distanceToPlayer = [self pathToPlayer].count;
    self.ruleSystem.state[@"distanceToPlayer"] = @(distanceToPlayer);
 
    // 3
    [self.ruleSystem reset];
    [self.ruleSystem evaluate];
 
    // 4
    self.hunting = ([self.ruleSystem gradeForFact:@"hunt"] > 0.0);
    if (self.hunting) {
        [self startFollowingPath:[self pathToPlayer]];
    } else {
        [self startFollowingPath:[self pathToNode:self.scatterTarget]];
    }
```

在这个例子中，使用规则系统有四个步骤：

1. `initWithGame:entity:`方法创建了 [GKRuleSystem](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/cl/GKRuleSystem) 实例，然后用 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 对象初始化。这个规则系统将被系统重用，所以例子中将它保存在一个属性中。

2. 每帧都会调用的 [updateWithDeltaTime:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKState_Class/index.html#//apple_ref/occ/instm/GKState/updateWithDeltaTime:) 方法使用规则系统来决定敌对角色接下来会做什么。首先，它计算敌方和玩家的距离，并存储在规则系统的 [state](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instp/GKRuleSystem/state) 字典中。

3. 然后，该方法重置规则系统（返回之前计算的规则给系统的议程，并且清除之前调用更新方法产生的事实）并用刚更新状态信息计算规则系统。

4. 最后，这个方法使用 [gradeForFact:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/gradeForFact:) 方法去检测规则系统是否能生成一个事实，然后基于这个信息，为这个敌方角色选择一个动作来执行。


这段代码展示了一个很小的规则系统的使用—实际上，用这么简单的语句并不能真正实现功能。但是，通过扩展这个例子，你能够轻松的给一个游戏增加负责的行为。例如：

- 加入更多的状态信息，比如可以增加这个敌方角色与其他敌方角色的距离、游戏开始的时间、或者是玩家任务的完成情况。

- 加入更多的规则去检测这些附加的状态信息。并为这些新添的规则加入更多的事实表现，然后把规则系统产生的事实组合到一起为敌方角色选择一个行为。

- 将 [NSPredicate](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSPredicate_Class/index.html#//apple_ref/occ/cl/NSPredicate) 定义从你的代码移动到数据文件中，并设置环境让每一个规则都去检测，你和其他成员不需要在Xcode中重新编译，就能直接改变游戏行为。
- 此外，也可以使用 [ruleWithBlockPredicate:action:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/clm/GKRule/ruleWithBlockPredicate:action:) 方法或者创建一个通用的 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 子类来生成规则，用这些规则基于变化的等级来生成事实，并且使用模糊逻辑的方法[minimumGradeForFacts:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/minimumGradeForFacts:) 和  [maximumGradeForFacts:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/maximumGradeForFacts:) 基于系统产生的一系列事件的集合为角色选择一个行为。


### 专注突发行为的模糊逻辑系统

**演示机器人（DemoBots）**示例代码采用了之前介绍的几个方法来创建让人感兴趣的角色。在游戏中的每个等级中，存在“task bot”角色，他们可以处在“好”或“坏”的状态。游戏的目的是让玩家找出每一个”坏“的机器人并“调试”它，将它的状态变为“好”。好的机器人仅会按照预定的路线水平移动，但是坏的机器人会便显出好几种方式—它可能会捕捉并且攻击玩家角色， 也可能会找出好的机器人把他们变坏，更或是巡视周围找点麻烦。

> 这个章节讨论的实例代码：[DemoBots: Building a Cross Platform Game with SpriteKit and GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/DemoBots/Introduction/Intro.html#//apple_ref/doc/uid/TP40015179)。请下载，并在Xcode中查看。

这个游戏使用规则系统去决定一个坏的机器人在任何时刻的行为。不像之前的例子，这个规则系统使用了模糊逻辑—系统中的规则会根据变化的等级生成事实，在计算规则系统之后，一个机器人会和其他机器人的等级作比较，然后决定采取什么样的动作。

每个规则使用游戏状态的总览信息快照，所以多个规则类都能够使用这些信息而不需要重新计算。游戏只有在计算规则系统的时候生成快照，并且这些计算只是周期的进行，而不是每帧，这样做减少了那些不必要的计算。

创建该系统，DemoBots 定义了一系列的 [GKRule](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRule_Class/index.html#//apple_ref/occ/cl/GKRule) 子类，每一个子类完成简单的计算去决定一个相关联动作的等级。例如，代码 8-2 展示一个规则类，它判断当前坏机器人数目在当前等级下是否是否很低。


**代码 8-2** 模糊逻辑规则示例

```oc
class BadTaskBotPercentageLowRule: FuzzyTaskBotRule {
    override func grade() -> Float {
        return max(0.0, 1.0 - 3.0 * snapshot.badBotPercentage)
    }
    init() { super.init(fact: .BadTaskBotPercentageLow) }
}
```

这条规则提供了最高等级时的水平包含零个坏机器人，并且当坏机器人占机器人总数比例变高时，会降低该等级。当存在三分之一或者更多的坏机器人的时候，规则的等级是零。

在代码 8-3 中简单展示了一个 DemoBots 中的父类，它提供了一个通用的模块用来计算规则，并且用每个规则的计算结果生成事实。

**代码 8-3** 一个模糊逻辑规则类的父类

```oc
class FuzzyTaskBotRule: GKRule {
    var snapshot: EntitySnapshot!
    func grade() -> Float { return 0.0 }
    let fact: Fact
    
    init(fact: Fact) {
        self.fact = fact
        super.init()
        // Set the salience so that 'fuzzy' rules will evaluate first.
        salience = Int.max
    }
    
    override func evaluatePredicateWithSystem(system: GKRuleSystem) -> Bool {
        snapshot = system.state["snapshot"] as! EntitySnapshot
        if grade() >= 0.0 {
            return true
        }
        return false
    }
    
    override func performActionWithSystem(system: GKRuleSystem) {
        system.assertFact(fact.rawValue, grade: grade())
    }
}
```

有了这个通用模块，构建规则系统是一件非常简单的事，只需要创建每个规则类的实例就可以。然后，在提供快照和计算规则系统之后，`TaskBot`类的实例会用最小值和最大值方法对比每个系统生成事实进行对比。

**代码 8-4** 从一个规则系统得到结论

```oc
// 1
let huntPlayerBotRaw = [
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageHigh.rawValue, Fact.PlayerBotNear.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageMedium.rawValue, Fact.PlayerBotNear.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageHigh.rawValue, Fact.PlayerBotFar.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageHigh.rawValue, Fact.PlayerBotMedium.rawValue, Fact.GoodTaskBotMedium.rawValue]),
]
let huntPlayerBot = huntPlayerBotRaw.reduce(0.0, combine: max)
let huntTaskBotRaw = [
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageLow.rawValue, Fact.GoodTaskBotNear.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageMedium.rawValue, Fact.GoodTaskBotNear.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageLow.rawValue, Fact.PlayerBotMedium.rawValue, Fact.GoodTaskBotMedium.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageMedium.rawValue, Fact.PlayerBotFar.rawValue, Fact.GoodTaskBotMedium.rawValue]),
    ruleSystem.minimumGradeForFacts([Fact.BadTaskBotPercentageLow.rawValue, Fact.PlayerBotFar.rawValue, Fact.GoodTaskBotFar.rawValue])
]
let huntTaskBot = huntTaskBotRaw.reduce(0.0, combine: max)
 
// 2
if huntPlayerBot >= huntTaskBot && huntPlayerBot > 0.0 {
    guard let playerBotAgent = state.playerBotTarget?.target.agent else { return }
    mandate = .HuntAgent(playerBotAgent)
} else if huntTaskBot > huntPlayerBot {
    mandate = .HuntAgent(state.nearestGoodTaskBotTarget!.target.agent)
} else {
    switch mandate {
    case .FollowBadPatrolPath:
        break
    default:
        let closestPointOnBadPath = closestPointOnPath(badPathPoints)
        mandate = .ReturnToPositionOnPath(float2(closestPointOnBadPath))
    }
}
```

清单罗列的代码执行了两个主要任务（有上面标识的数字表示）：

1. 比较系统中事实的等级获得结论。在模糊逻辑中，最小值函数（在上面的例子中，[minimumGradeForFacts:](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKRuleSystem_Class/index.html#//apple_ref/occ/instm/GKRuleSystem/minimumGradeForFacts:) 调用的）和最大值函数（`reduce(_:combine:)`调用的）表面上与布尔逻辑运算符的“与”和“或”是类似的。因此，`huntPlayerBotRaw`和`huntPlayerBot`的计算和下面的伪代码是一样:

	```
	untPlayerBot = (PlayerBotNear AND BadTaskBotPercentageHigh) OR (PlayerBotNear AND BadTaskBotPercentageMedium) OR ...
	```

	然而，不像布尔逻辑，计算中的每一个元素都是一个等级—一个断言的可信度—得到的`huntPlayerBot`值也是一个等级。你可以把它当成一个概率：“我需要寻找的玩家可能是。。。”

2. 使用这个结论去选择一个行动。对`TaskBot`对象的`mandate`属性设置值，决定了游戏在下面几个时刻的目标。如果`huntPlayerBot`的得分非零，并且大于 `huntTaskBot`的得分，那么这个机器人会去攻击玩家角色；如果`huntTaskBot`较高，那么这个机器人将会把最近的好机器人变坏；如果两个得分都是零，那么这个机器人仅会在线路上巡逻。
这个目标会一直维持到下次规则系统的计算。同时，在游戏的其他模块中，每帧更新的方法会使用授权的值去驱动机器人的行为。根据机器人的任务，负责机器人运动的 [GKAgent](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKAgent_Class/index.html#//apple_ref/occ/cl/GKAgent) 对象会获取一组目标，然后让这些机器人发现并按照一定的路径去追踪目标，或者不同的目标，让机器人在游戏中巡逻。