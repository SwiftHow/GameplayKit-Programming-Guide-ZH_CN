# 寻路

在很多游戏中路径导航是非常重要的一项功能。一些回合制游戏要求玩家在棋盘上选择出到达某个目标的最佳路径；许多动作游戏和冒险游戏将玩家放置在某种形式的迷宫中，玩家角色必须逃出迷宫来达到游戏的最终目标，同时敌方角色也必须用它们的方式通过迷宫来给玩家增加挑战。像这样决定如何通过棋盘，迷宫等其他可导航空间的过程被称之为*寻路*。GameplayKit 提供了一系列工具来映射你的游戏世界，并寻求通过它的路径，然后你就可以用它来移动人物或者其他游戏实体。

在 GameplayKit 中使用寻路工具需要使用*图（Graph）*详细描述出游戏中的可导航区域——也就是说，你就是说你需要指定一个包含不同位置或节点，并指定实体从一个位置如何导航到另一个位置的集合。因为大多数游戏需要像这样描述 2D 移动，这类游戏使用寻路的关键就是找到一个简单地方法创建游戏世界的图描述。

## 寻路游戏设计

你可以使用`GKGraph`类和其相关的接口描述一个图，它提供了三个方法来描述可导航区域。

- **比如在被障碍物中断的连续空间中** 通过使用`GKPolygonObstacle`类来表示障碍物区域，使用`GKGraphNode2D`类来表示开放空间中的关键坐标，再使用`GKObstacleGraph`类来创建包含两者的图。图 6-1 说明了这个形式，这种方式在许多 2D 游戏，冒险和解谜游戏，甚至许多使用 2D 控制移动的 3D 游戏中非常常见。

  **图 6-1** 在障碍物间寻路
  
  ![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/obstacle_pathfinder_2x.png)
  
  当你使用 SpriteKit 创建游戏时，你可以基于场景或节点的内容来创建一组`GKPolygonObstacle`对象。利用这个技术，你可以用那些你可能已经在其他游戏中使用的方法来定义导航区域。比方说，你可以定义一层使用 Xcode 里的 SpriteKit 场景编辑器利用节点的物理模型来构建出玩家不能通过的区域的层级，然后使用`obstaclesFromNodePhysicsBodies:`方法来生成匹配这些区域的`GKPolygonObstacle`对象。
  
- **比如在一个有效位置为整数坐标的离散二位网格中** 使用`GKGridGraph`类和`GKGridGraphNode`类来创建基于网格的图。图 6-2 说明了这种情况，这种情况在很多经典的街机游戏，棋盘游戏和策略化角色扮演游戏中经常出现。

  **图 6-2** 在网格中寻路
  
  ![](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Art/grid_pathfinder_2x.png)
  
  在一些这样的游戏里，角色的运动似乎是连续的，但是为了游戏逻辑所有的位置都被限制为整数网格。
  
- **比如在一个包含离散坐标和坐标间连接的集合中** 使用`GKGraphNode`类来表示没有多边形关系的分散节点或连接，然后使用`GKGraph`类来统一处理。这种类型特别适用一些游戏实体需要在明显标记的空间内移动，且空间内的实际位置是不相关的游戏逻辑。一些棋牌游戏和策略游戏适合这种形式。

  这种情况下，节点与节点间的联系是定向的，也就是说，节点 A 到节点 B 的连接只说明可以从节点 A 向节点 B 导航，如果想要从节点 B 向节点 A 导航，仍然需要另一个节点 B 到节点 A 的定向连接。
  
在你创建了图之后，你可以使用该`GKGraph`对象连接基于图中已经存在的节点关系连接到新的节点，也可以不管已存在节点的关系从图形中删除一些节点，最重要的是，你可以找到从一个节点到另一个节点的路径。

## 寻路游戏应用

在游戏中使用寻路一般需要三到四步：

1. 在游戏开始之前，用上文提及的设计方法创建`GKGraph`类或其子类的实例来表示游戏场景的静态内容。
2. 当你需要为游戏中的动态实体寻找路径时，比方说，用一个点击或者触摸事件让玩家角色在障碍之间行进以到达目标位置时，你应该从已存在节点中找出符合这些实体位置的节点，或者创建代表这些实体的零时节点，并把它们加入到图中。
3. 使用`findPathFromNode:toNode:`方法，从图形中找到路径。这个方法返回了一个包含`GKGraphNode`对象的数组，这个数组表示一条可用路径。（如果是创建图形时使用的是`GKGraphNode`的子类，那么这个数组也可能包含这些子类。）你可以使用这些节点包含的位置信息，沿着路径来移动实体，比如一个 SpriteKit 游戏可以创建包含一系列`SKAction`对象的动画来按照路径移动一个游戏角色。
4. （可选项）寻路完成后，将之前创建的零时节点从图形中移除，那样它们就不会干扰之后的寻路逻辑。

下面的章节说明了使用寻路的两个游戏例子。

### 在网格中寻路

**迷宫（Maze）**项目（已经在 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节和 [State Machines](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/StateMachine.html#//apple_ref/doc/uid/TP40015172-CH7-SW1) 中提及）是基于一些经典解密游戏实现的。在这个游戏中，玩家和怪物（敌人）角色都在一个整数网格的迷宫中移动。移动看上去是连续的，只是因为网格位置间隔是通过动画来补充的。

> 这个章节讨论的实例代码：[Maze: Getting Started with GameplayKit](https://developer.apple.com/sample-code/wwdc/2015/downloads/Maze.zip)。请下载，并在Xcode中查看。

游戏创建了一个`GKGridGraph`对象来表示迷宫，如代码 6-1所示。

**代码 6-1** 生成一个网格图

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

其中`graphFromGridStartingAt:width:height:diagonalsAllowed:`方法创建了在指定网格中每一个节点位置的图。之后代码删除网格中与迷宫壁对应的节点，之留下迷宫可穿越的区域作为图的节点。

当敌方角色处于追逐状态时，它们利用寻路来获得到达玩家当前位置的路径。代码 6-2 显示了这个行为的实现。

**代码 6-2** 寻路到某个网格位置

```oc

- (NSArray<GKGridGraphNode *> *)pathToNode:(GKGridGraphNode *)node {
    GKGridGraph *graph = self.game.level.pathfindingGraph;
    GKGridGraphNode *enemyNode = [graph nodeAtGridPosition:self.entity.gridPosition];
    NSArray<GKGridGraphNode *> *path = [graph findPathFromNode:enemyNode toNode:node];
    return path;
}
 
- (void)startFollowingPath:(NSArray<GKGridGraphNode *> *)path {
    // Set up a move to the first node on the path, but
    // no farther because the next update will recalculate the path.
    if (path.count > 1) {
        GKGridGraphNode *firstMove = path[1]; // path[0] is the enemy's current position.
        AAPLSpriteComponent *component = (AAPLSpriteComponent *)[self.entity componentForClass:[AAPLSpriteComponent class]];
        component.nextGridPosition = firstMove.gridPosition;
    }
}

```

在这个例子中，怪物和玩家需要使用的位置都已经存在于图中了。（再次重申，虽然角色的移动由于动画看上去是连续的，但在游戏逻辑中，每个角色始终在一个整数网格位置。）因此，为了帮助怪物找到追逐玩家的路径，这个方法只需要寻找对应怪物和玩家的当前位置的`GKGridGraphNode`对象，之后使用`findPathFromNode:toNode:`的方法来找到路径。

为了移动敌方角色，这个方法只检查了路径节点中的前两个节点。由于这个方法是从`updateWithDeltaTime:`方法中被调用的，所以它只在动画的每一帧执行，因此当玩家的位置改变时，它也可以找到新的追逐路径。因为沿着路径的第一步移动敌方角色花费的时间要比计算新路径的时间长的多，这个方法只是简单地通过设置游戏中`SpriteComponent`类的目标位置来开始移动动画。（更多关于这个类的讨论请查看 [Entities and Components](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/EntityComponent.html#//apple_ref/doc/uid/TP40015172-CH6-SW1) 章节。）

### 在障碍间寻路

**演示机器人（DemoBots）**项目实现了一个不一样的游戏，游戏中角色可以自由的在开放空间里移动，所以基于网格的寻路就不合适了。相反，这个游戏使用了基于障碍物的寻路逻辑。

> 这个章节讨论的实例代码：[DemoBots: Building a Cross Platform Game with SpriteKit and GameplayKit](https://developer.apple.com/library/prerelease/ios/samplecode/DemoBots/Introduction/Intro.html#//apple_ref/doc/uid/TP40015179)。请下载，并在Xcode中查看。

为了准备这类寻路方式，在不同级别的场景文件（由 Xcode 中的 SpriteKit 场景编辑器创建）中，这个例子使用了一组游戏角色无法通过的物理刚体区域的节点。之后，在`LevelScene`类实现了一个`graph`属性，它构造了一个`GKObstacleGraph`对象，如代码 6-3 所示。

**代码 6-3** 创建一个基于障碍物的图

```oc

lazy var obstacleSpriteNodes: [SKSpriteNode] = self["world/obstacles/*"] as! [SKSpriteNode]
 
lazy var polygonObstacles: [GKPolygonObstacle] = SKNode.obstaclesFromNodePhysicsBodies(self.obstacleSpriteNodes)
 
lazy var graph: GKObstacleGraph = GKObstacleGraph(obstacles: self.polygonObstacles, bufferRadius: GameplayConfiguration.TaskBot.pathfindingGraphBufferRadius)

```

> 在 DemoBots 项目中，`GameplayConfiguration`对象储存着游戏逻辑的全局常量。

`graph`属性的初始化方法首先读取了`polygonObstacles`属性，这些属性搜索了该层级的节点树（请查看 [SKNode Class Reference](https://developer.apple.com/library/prerelease/ios/documentation/SpriteKit/Reference/SKNode_Ref/index.html#//apple_ref/doc/uid/TP40013023) 中的 [Searching the Node Tree](https://developer.apple.com/library/prerelease/ios/documentation/SpriteKit/Reference/SKNode_Ref/index.html#//apple_ref/doc/uid/TP40013023-CH1-SW74)）并提供了一个`obstacles`节点的所有子节点的数组。之后，`SKNode`的`obstaclesFromNodePhysicsBodies:`方法根据每个障碍物节点的物理形状创建了一个包含`GKPolygonObstacle`对象的数组。最终这些对象创建出图。

当论及为游戏角色准备一个路径时，游戏中的`TaskBotBehavior`类使用了代码 6-4 中所示的方法。

**代码 6-4** 在基于障碍物的图中寻路

```oc

private func addGoalsToFollowPathFromStartPoint(startPoint: float2, toEndPoint endPoint: float2, pathRadius: Float, inScene scene: LevelScene) -> [CGPoint] {
    
    // Convert the provided `CGPoint`s into nodes for the `GPGraph`.
    let startNode = connectedNodeForPoint(startPoint, onObstacleGraphInScene: scene)
    let endNode = connectedNodeForPoint(endPoint, onObstacleGraphInScene: scene)
    
    // Find a path between these two nodes.
    let pathNodes = scene.graph.findPathFromNode(startNode, toNode: endNode) as! [GKGraphNode2D]
    
    // Create a new `GKPath` from the found nodes with the requested path radius.
    let path = GKPath(graphNodes: pathNodes, radius: pathRadius)
    
    // Add "follow path" and "stay on path" goals for this path.
    addFollowAndStayOnPathGoalsForPath(path)
    
    // Remove the "start" and "end" nodes now that the path has been calculated.
    scene.graph.removeNodes([startNode, endNode])
    
    // Convert the `GKGraphNode2D` nodes into `CGPoint`s for debug drawing.
    let pathPoints: [CGPoint] = pathNodes.map { CGPoint($0.position) }
    return pathPoints
}

```

`addGoalsToFollowPathFromStartPoint`方法和代码 6-2 所示步骤相似，但在障碍物系图上有些区别：

1. 这个方法创建并连接了代表所需的开始到结束路径的零时节点。不同于基于网格的图，包含连续的 2D 空间的图形并不是已经包含了这些位置的节点。（在这个项目中，`connectedNodeForPoint`是一个简便方法，它即使用`connectNodeUsingObstacles:`方法创建了新的节点，又将节点添加到了图中。）
2. `findPathFromNode:toNode:`方法返回了包含一组代表零时节点间路径的图节点数组。
3. 而后，项目中的`addFollowAndStayOnPathGoalsForPath`方法使用了 GameplayKit 中的代理（Agents）方法让一个游戏角色沿着路径移动。你可以查看[代码 7-1](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/GameplayKit_Guide/Agent.html#//apple_ref/doc/uid/TP40015172-CH8-SW4)，了解更多该方法的实现细节。
4. 在所有步骤完成后，`startNode`和`endNode`节点从图形中被移除了，那样它们就不会影响之后的寻路操作了。
5. 最后，这个方法也返回了一个包含`CGPoint`结构的数组，这样方便开发者在代码调试时可以按照这些坐标画出相应的路径。

