# 寻路

在很多游戏中路径导航是非常重要的一部分功能。一些回合制游戏要求玩家在棋盘上选择出到达某个目标的最佳路径；许多动作游戏和冒险游戏将玩家放置在某种形式的迷宫中，玩家角色必须逃出迷宫来达到游戏的最终目标，同时敌方角色也必须用它们的方式通过迷宫来给玩家增加挑战。像这样决定如何通过棋盘，迷宫等其他可导航空间的过程被称之为*寻路*。GameplayKit 提供了一系列工具来映射你的游戏世界，并寻求通过它的路径，然后你就可以用它来移动人物或者其他游戏实体。

在 GameplayKit 中使用寻路工具需要使用*曲线*详细描述出游戏中的可导航区域——也就是说，你就是说你需要指定一个包含不同位置或节点，并指定实体从一个位置如何导航到另一个位置的集合。因为大多数游戏需要像这样描述 2D 移动，这类游戏使用寻路的关键就是找到一个简单地方法创建游戏世界的曲线描述。

## 寻路游戏设计

你可以使用`GKGraph`类和其相关的接口描述一个曲线，它提供了三个方法来描述可导航区域。

- **比如在被障碍物中断的连续空间中** 通过使用`GKPolygonObstacle`类来表示障碍物区域，使用`GKGraphNode2D`类来表示开放空间中的关键坐标，再使用`GKObstacleGraph`类来创建包含两者的图形。图 6-1 说明了这个形式，这种方式在许多 2D 游戏，冒险和解谜游戏，甚至许多使用 2D 控制移动的 3D 游戏中非常常见。

  **图 6-1** 在障碍物间寻路
  
  ![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/obstacle_pathfinder_2x.png)
  
  当你使用 SpriteKit 创建游戏室，你可以基于场景或节点的内容来创建一组`GKPolygonObstacle`对象。利用这个技术，你可以用那些你可能已经在其他游戏中使用的方法来定义导航区域。比方说，你可以定义一层使用 Xcode 里的 SpriteKit 场景编辑器利用节点的物理模型来构建出玩家不能通过的区域的层级，然后使用`obstaclesFromNodePhysicsBodies:`方法来生成匹配这些区域的`GKPolygonObstacle`对象。
  
- **比如在一个有效位置为整数坐标的离散二位网格中** 使用`GKGridGraph`类和`GKGridGraphNode`类来创建基于网格的图形。图 6-2 说明了这种情况，这种情况在很多经典的街机游戏，棋盘游戏和策略化角色扮演游戏中经常出现。

  **图 6-2** 在网格中寻路
  
  ![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/grid_pathfinder_2x.png)
  
  在一些这样的游戏里，角色的运动似乎是连续的，但是为了游戏逻辑所有的位置都被限制为整数网格。
  
- **比如在一个包含离散坐标和坐标间连接的集合中** 使用`GKGraphNode`类来表示没有多边形关系的分散节点或连接，然后使用`GKGraph`类来统一处理。这种类型特别适用一些游戏实体需要在明显标记的空间内移动，且空间内的实际位置是不相关的游戏逻辑。一些棋牌游戏和策略游戏适合这种形式。

  这种情况下，节点与节点间的联系是定向的，也就是说，节点 A 到节点 B 的连接只说明可以从节点 A 向节点 B 导航，如果想要从节点 B 向节点 A 导航，仍然需要另一个节点 B 到节点 A 的定向连接。
  
在你创建了图形（或曲线）之后，你可以使用该`GKGraph`对象连接基于图中已经存在的节点关系连接到新的节点，也可以不管已存在节点的关系从图形中删除一些节点，最重要的是，你可以找到从一个节点到另一个节点的路径。

## 寻路游戏应用

在游戏中使用寻路一般需要三到四步：

1. 第一步
2. 第二部
3. 第三部
4. 第四部

下面的章节说明了使用寻路的两个游戏例子。

### 在网格中寻路

**迷宫（Maze）**项目（已经在 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节和 [State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1) 中提及）是基于一些经典解密游戏实现的。在这个游戏中，玩家和怪物（敌人）角色都在一个整数网格的迷宫中移动。移动看上去是连续的，只是因为网格位置间隔是通过动画来补充的。

> 这个章节讨论的实例代码：[Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip)。请下载，并在Xcode中查看。

游戏创建了一个`GKGridGraph`对象来表示迷宫，如代码 6-1所示。

**代码 6-1** 生成一个网格图形

```oc

GKGridGraph *graph = [GKGridGraph graphFromGridStartingAt:(vector_int2){0, 0} width:AAPLMazeWidth height:AAPLMazeHeight diagonalsAllowed:NO];
NSMutableArray *walls = [NSMutableArray arrayWithCapacity:AAPLMazeWidth*AAPLMazeHeight];
NSMutableArray *spawnPoints = [NSMutableArray array];
for (int i = 0; i < AAPLMazeWidth; i++) {
    for (int j = 0; j < AAPLMazeHeight; j++) {
        int tile = [self tileAtRow:i column:j];
        if (tile == TileTypeWall) {
            [walls addObject:[graph nodeAtGridPosition:(vector_int2){i, j}]];
        } else if (tile == TileTypePortal) {
            [spawnPoints addObject:[graph nodeAtGridPosition:(vector_int2){i, j}]];
        } else if (tile == TileTypeStart) {
            _startPosition = [graph nodeAtGridPosition:(vector_int2){i, j}];
        }
    }
}

// 从图形中移除路径通过的墙体瓦片。
[graph removeNodes:walls];

```


### 在障碍间寻路




