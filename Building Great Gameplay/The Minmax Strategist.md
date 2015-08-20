# 极大化极小策略

在视频游戏出现之前的很长一段时间，许多游戏就已经非常流行了，尤其是一些棋类游戏。比如，双陆棋，国际象棋，井字棋和围棋等等，本质上都是逻辑类的游戏。这类游戏中，一放玩家通过考虑对方玩家（们）的下一步行动来决定自己本回合的最佳行动来获得最终胜利，无论对手是玩家还是电脑，这种胜利总是令人兴奋。

在 GameplayKit 中，一个*策略*就是一种简单的人工智能，你可以利用它创建这类游戏的电脑对手。[GKMinmaxStrategist](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKMinmaxStrategist_Class/index.html#//apple_ref/occ/cl/GKMinmaxStrategist) 类及其支持的接口实现了一个*极小化极大策略*用于实现包含大多数以下特点得游戏逻辑：

- **顺序的** 每个玩家必须按顺序操作，比方说回合制。
- **对抗的** 如果一个玩家赢了，那么他的对手一定会输掉比赛。游戏设计假定在每一回合里，一个玩家的动作将导致另一个玩家接近或者远离胜利。
- **确定的**（一旦规则既定）玩家按照预期的操作制定的行为就会达到效果，而不会有机会发生改变。
- **信息公开的** 玩家总是能够发现所有对于游戏结果至关重要的信息。一盘棋的输赢取决于棋子在棋盘所处的位置，而双方玩家同时都能看着这些信息。如果存在一中棋类游戏，每个玩家可以持有相互不可见的手牌，这些手牌影响着棋局，那么这就不是一个信息公开的游戏了，因为对于一个玩家而言他无法明确知道另一个玩家的可能操作。

你可以使用极小化极大策略构建不满足以上所有特性的游戏（或者复杂游戏中的元素），前提是你可以把游戏设计转化为符合 GameplayKit 接口的描述。比方说，在一个单人游戏中（缺少一个对手），你可以使用策略来给出最佳移动的建议。或者，在一个包含知道被发现时才显示的隐藏元素的游戏中，该策略可以就已知信息来规划移动（操作）。一般情况下，如果你的游戏符许多上述特性，策略就能帮你产生更优的游戏建议（即，每一场游戏中更接近胜利）。


## 极小化极大策略的设计

极小化极大策略的工作原理是在一个决策树中预测并评级游戏未来可能的状态，之后通过遍历树依照最高评分决策来选择下一步的行动。对己方回合，该策略对那些能够有益胜利的可能步数给出较高的评分；在对方回合，该策略对那些能让对方获胜的步数则给出较低评分。（之所以被命名为 Minmax 策略，是应为它将最大化己方操作的评分，并最小化对方操作的评分。）

设想有一款游戏，每个玩家（红方和黑方）会往网格某一列放置一枚代表本方颜色的棋子，当存在一列连续四枚棋子在横向，纵向或斜向颜色一致的一方获胜。图 5-1 展示了这样一个游戏的策略树。*四子棋（FourInARow）*项目实现了这个游戏。

>本章讨论的实例代码：[FourInARow: Using the GameplayKit Minmax Strategist for Opponent AI](https://developer.apple.com/library/prerelease/ios/samplecode/FourInARow/Introduction/Intro.html#//apple_ref/doc/uid/TP40016142)。请下载，并在Xcode中查看。

**图 5-1** 四子棋游戏中有可能的下一步

![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/minmax_2x.png)

图中，红方可以在棋盘七列中的任意一列放置一枚棋子。对于每一列，*策略*考虑了如果在该列放置棋子后游戏所处的状态，并为每一种结果给出评分。（上图只显示了几个可能情况。）

- 如果红方在第一列（最左侧一列）落子，黑方如在下回合也在第一列落子，黑方就会取胜。这对红方而言并非好的结果，所以该策略为这一步打了低分。
- 如果红方在第七列（最右侧一列）落子，那么黑方在下回合无论在哪里落子都不会取胜，所以该策略为红方的这步棋打了个中间分。
- 因为玩家在第四列（中间这一列）落子就获胜了，所以这步棋获得了最高的评分。

所以从这个例子中，我们能看出，在游戏中使用*极小化极大策略*需要你的游戏结构包含下面三个要点：

- 包含一个游戏棋盘，能够呈现出影响比赛结果的重要元素。
- 包含一个玩家具体如何移动的方法，并且指明玩家的每一步如何改变了棋局状态。
- 包含一个评价游戏模型状态得分的方法，这样对棋局更为有利的状态就会从其他状态中区分出来。

一个有着良好架构的游戏往往已经包含了这些功能。比方说，四子棋（FourInARow）项目使用了 [Model-View-Controller](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/DevPedia-CocoaCore/MVC.html#//apple_ref/doc/uid/TP40008195-CH32) 结构将用户界面和展示从游戏逻辑中剥离出来。代码 5-1 中的`AAPLBoard`类展示了这个功能实现：

**代码 5-1** 四子棋的界面

```oc
@interface AAPLBoard : NSObject
{
    AAPLChip _cells[AAPLBoardWidth * AAPLBoardHeight];
}
@property AAPLPlayer *currentPlayer;
- (AAPLChip)chipInColumn:(NSInteger)column row:(NSInteger)row;
- (BOOL)canMoveInColumn:(NSInteger)column;
- (void)addChip:(AAPLChip)chip inColumn:(NSInteger)column;
- (BOOL)isFull;
- (NSArray<NSNumber *> *)runCountsForPlayer:(AAPLPlayer *)player;
@end
```

这些属性和方法给出了*策略*所需的所有游戏信息：

- `AAPLBoard`对象本身代表了游戏状态（或者潜在的未来状态）。它包含了一个内容为`AAPLChip`值的二维数组（使用整数标记并枚举出红棋，黑棋和空格区域）。同时，该对象也包含一个`AAPLPlayer`对象的引用，者表示出当前是哪个玩家的回合。
- `canMoveInColumn`和`addChip`方法指定了任何时间可能的操作，并模拟出这些操作结果。为了推测出将来的游戏状态，*策略*会拷贝出一个代表当前游戏状态的`AAPLBoard`对象，在此基础上调用`addChip`方法。
- `isFull`方法通过棋盘绘制来判断游戏是否已经结束了（棋盘被占满）。`runCountsForPlayer`方法帮助判断任意一方玩家是否将要赢得游戏。如果你拷贝了`AAPLBoard`对象来推测将来的游戏状态，这些方法可以用于为这些将来的状态评分。

## 极小化极大策略的应用

在你的游戏中使用 [GKMinmaxStrategist](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKMinmaxStrategist_Class/index.html#//apple_ref/occ/cl/GKMinmaxStrategist) 类创建实例来实现*极小化极大策略*。该类使用你定义的游戏模型来推测游戏的状态，并随时向玩家提供下一步的“最好”建议。然而，在`GKMinmaxStrategist`类能够推理出你的游戏模型之前，你必须使用 GameplayKit 能够理解的理解的标准方式来描述游戏模型。你可以利用 [GKGameModel](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKGameModel_Protocol/index.html#//apple_ref/occ/intf/GKGameModel)，[GKGameModelUpdate](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKGameModelUpdate_Protocol/index.html#//apple_ref/occ/intf/GKGameModelUpdate)，和 [GKGameModelPlayer](https://developer.apple.com/library/prerelease/ios/documentation/GameplayKit/Reference/GKGameModelPlayer_Protocol/index.html#//apple_ref/occ/intf/GKGameModelPlayer) 来实现这一步。

>本章讨论的实例代码：[FourInARow: Using the GameplayKit Minmax Strategist for Opponent AI](https://developer.apple.com/library/prerelease/ios/samplecode/FourInARow/Introduction/Intro.html#//apple_ref/doc/uid/TP40016142)。请下载，并在Xcode中查看。

### 在游戏中采用 GameplayKit 协议

GameplayKit 中包含三个关键协议供你定义界面，并帮助 GameplayKit 推导出游戏的状态。

- 将`GKGameModel`协议应用在对象上，能够代表游戏的不同状态。在四子棋的例子中，`AAPLBoard`类是一个很好地使用这个协议的例子。
- `GKGameModelPlayer`协议所在的对象是游戏中玩家角色的抽象概念。如例子中的`AAPLPlayer`类。
- `GKGameModelUpdate`协议所在的对象描述了两个游戏建状态的转化。在前面的例子中，并没有这样的对象因为`addChip`方法负责状态的转化。这样的话，如果还要使用`GKGameModelUpdate`协议就需要另外创建新的类了。

下面的代码介绍了在四子棋游戏中，我们是如何扩展游戏的类来应用这些协议的。首先，代码 5-2 创建了一个`AAPLMove`类应用了`GKGameModelUpdate`协议。

**代码 5-2** Move 类应用了 GKGameModelUpdate 协议

```oc
@interface AAPLMove : NSObject <GKGameModelUpdate>
@property (nonatomic) NSInteger value;
@property (nonatomic) NSInteger column;
+ (AAPLMove *)moveInColumn:(NSInteger)column;
@end
```

`GKGameModelUpdate`协议只需要一个属性——`value`。*极小化极大策略*在权衡每一个可能性是否可取时，将在该属性中存放评分。为了实现这个协议剩下的部分，你需要为你的游戏创建属性和方法来描述一步操作。在四子棋游戏中，唯一需要具体描述的信息就是在那一列落子。

接下来，在代码 5-3 中，我们向`AAPLPlayer`类添加了`GKGameModelPlayer`协议。

**代码 5-3** Player 类应用了 GKGameModelPlayer 协议

```oc
@interface AAPLPlayer (AAPLMinmaxStrategy) <GKGameModelPlayer>
@end
 
@implementation AAPLPlayer (AAPLMinmaxStrategy)
- (NSInteger)playerId { return self.chip; }
@end
```

`AAPLPlayer`类简单的暴露出它的`AAPLChip`枚举的整数值作为它的`playerId`属性。

向`AAPLBoard`类添加`GKGameModel`协议需要添加三块函数：保持追踪玩家，处理落子，拷贝游戏状态信息。代码 5-4 展示了第一步。

**代码 5-4** GKGameModel 实现：管理玩家角色

```oc
- (NSArray<AAPLPlayer *> *)players { return [AAPLPlayer allPlayers]; }
- (AAPLPlayer *)activePlayer { return self.currentPlayer; }
```

其中，`players`属性将一直返回游戏中的两个玩家。对于四子棋游戏而言，玩家数量和 id 是常量，所以这个数组被保存在了`AAPLPlayer`类中。而`activePlayer`属性只是简单返回了已经在`AAPLBoard`类中实现的`currentPlayer`属性。

`GKGameModel`的一个重要实现就是预测每个玩家可能的操作。代码 5-5 显示了`AAPLBoard`类是如何实现这一点的。

**代码 5-5** GKGameModel 实现：发现并处理落子

```oc
- (NSArray<AAPLMove *> *)gameModelUpdatesForPlayer:(AAPLPlayer *)player {
    NSMutableArray<AAPLMove *> *moves = [NSMutableArray arrayWithCapacity:AAPLBoardWidth];
    for (NSInteger column = 0; column < AAPLBoardWidth; column++) {
        if ([self canMoveInColumn:column]) {
            [moves addObject:[AAPLMove moveInColumn:column]];
        }
    }
    return moves; // will be empty if isFull
}
 
- (void)applyGameModelUpdate:(AAPLMove *)gameModelUpdate {
    [self addChip:self.currentPlayer.chip inColumn:gameModelUpdate.column];
    self.currentPlayer = self.currentPlayer.opponent;
}
```

其中，`gameModelUpdatesForPlayers:`方法返回了某个玩家的所有可能的操作。每一步操作都是游戏里包含`GKGameModelUpdate`协议的实例对象，也就是例子中的`Move`类。四子棋游戏通过返回一个对应所有未满列的`AAPLMove`对象的数组，来实现这个方法。

> 如果你的游戏中没有实现`isWinForPlayer:`和`isLossForPlayer:`方法（请查看代码 5-6 和下面的讨论），你要确保如果游戏结束时，`gameModelUpdatesForPlayers:`方法返回`nil`。

`applyGameModelUpdate:`方法包含一个`GKGameModelUpdate`对象，并处理该对象的任何指定动作。在这个例子中，一个游戏落子更新是一个`AAPLMove`对象，所以棋盘对象通过调用`addChip:`方法来向`AAPLMove`对象指定的列中落入当前玩家的棋子。这个方法同时也告诉 GameplayKit 当前游戏的回合顺序，这样*策略*就能提前知道玩家在哪个回合操作，并且事先准备一些接下来的几回合操作。因为实例游戏是两人游戏，所以`applyGameModelUpdate:`方法设置`activePlayer`属性为当前落子玩家的对立玩家。

为了让*极小化极大策略*能够决定哪一步棋是最好的选择，你的游戏逻辑必须指出什么时候游戏状态是更理想的，如代码 5-6 所示。

**代码 5-6** GKGameModel 实现：计算棋盘状态

```oc
- (BOOL)isWinForPlayer:(AAPLPlayer *)player {
    NSArray<NSNumber *> *runCounts = [self runCountsForPlayer:player];
    // The player wins if there are any runs of 4 (or more, but that shouldn't happen in a regular game).
    NSNumber *longestRun = [runCounts valueForKeyPath:@"@max.self"];
    return longestRun.integerValue >= AAPLCountToWin;
}
 
- (BOOL)isLossForPlayer:(AAPLPlayer *)player {
    return [self isWinForPlayer:player.opponent];
}
 
- (NSInteger)scoreForPlayer:(AAPLPlayer *)player {
    NSArray<NSNumber *> *playerRunCounts = [self runCountsForPlayer:player];
    NSNumber *playerTotal = [playerRunCounts valueForKeyPath:@"@sum.self"];
    NSArray<NSNumber *> *opponentRunCounts = [self runCountsForPlayer:player.opponent];
    NSNumber *opponentTotal = [opponentRunCounts valueForKeyPath:@"@sum.self"];
    // Return the sum of player runs minus the sum of opponent runs.
    return playerTotal.integerValue - opponentTotal.integerValue;
}
```