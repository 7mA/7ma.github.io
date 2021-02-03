---
title: Self-propelled Points：用p5.js实现追踪自走点
tags:
  - JavaScript
  - Devlog
categories:
  - 技术笔记
author: Kaku
date: 2021-01-29 15:39:42
permalink: p5js-self-propelled-points/
---

> 完成品：[Self-propelled Points](https://7ma.github.io/self-propelled-points/)

# 1.逻辑分析

既然要实现追踪，首先要理清追踪的逻辑。

## 1.1 空白区域

假设在一片空白的二维区域，点A追踪点B。
如果点B为静止点，那么对于点A来说，只要把移动方向瞄准点B然后按照给定步长移动即可。
步长决定点A的速度。
追踪的伪代码如下：

```
根据点A和点B的位置确定追踪方向角a;
确定步长n;
while(点A没有接触到点B){
  按追踪方向角a前进n;
}
```

<!--more-->

如果点B是运动的，那么为了实现追踪的效果，需要在移动的同时随时地调整追踪方向。
那么伪代码更改如下：

```
根据点A和点B的位置确定追踪方向角a;
确定步长n;
while(点A没有接触到点B){
  按追踪方向角a前进n;
  根据点A和点B的位置调整追踪方向角a;
}
```

在根据点A和点B的位置调整追踪方向时，为了做到真实，不能直接把方向扭到新的追踪方向，而是要根据旧方向与新方向的差分来按照一定步长调整方向。这个步长可以是固定的。
同时为了做到追踪效果，调整方向的旋转要向着差分小的一方旋转。

反映到伪代码如下：

```
根据点A和点B的位置确定追踪方向角a1;
确定移动步长n;
确定角度调整步长b;
while(点A没有接触到点B){
  按追踪方向角a前进n;
  确定点A和点B新的方向角a2;
  计算a1与a2两个方向的差分角，并确定小的一方;
  向小差分角方向旋转b;
}
```

## 1.2 有墙壁的区域

下面增加一点复杂程度，考虑在有墙壁的二维区域，点A追踪点B。
为了去除不必要的麻烦，考虑点B为静止点的情形。点B为动点时只需要把随时更新点B的方向角即可。
因为墙壁的存在，直接根据点A与点B的位置关系确定的直线不一定能够直接连通两点。
所以点A在追踪的过程中要根据前方墙壁的情况调整方向。

相比于点A与点B两个已经确定好的点，墙壁的数量和位置情况是随机可变的。
所以通过一一计算点A与所有墙壁的位置关系来预先规划路线的话计算太复杂。
而且如果点B是动点的话，每移动一次就要重新算一次，并不现实。
因此可以为点A增加一个模拟视野的概念，仅当视野中有墙壁的时候才调整方向。
视野可以用从点A出发的若干条表示视线的线段来表示，线段的长度代表视野的大小。
视野中出现墙壁，可以用视线与墙壁相交来表示。

伪代码如下：

```
根据点A和点B的位置确定追踪方向角a;
确定步长n;
确定墙壁w[m];
确定视野l[k];
while(点A没有接触到点B){
  if(l的成员与w的成员相交){
    调整点A的方向角a;
  }
  按追踪方向角a前进n;
}
```

这里出现了两个问题：

1. 如果视线与墙壁相交，该如何调整？
2. 调整过后的方向角该怎么调整回到点B的方向？

为了解决这两个问题，首先要考虑视野的构成。

视野可以抽象成单个点从某个角度范围接受光线的模型，如下所示：

<picture>
    <source type="image/webp" srcset="/p5js-self-propelled-points/viewline.webp">
    <source type="image/png" srcset="/p5js-self-propelled-points/viewline.png">
    <img src="/p5js-self-propelled-points/viewline.png" alt="视野示意图" />
</picture>

其中点A是二维空间中的视点，射线l和m之间所夹的角是点A能看到的视角。
那么射线l和m之间所有从点A出发的线段都是点A的视线。
那么与这些视线中的任意一条相交的话，都可以认为是被点A“看到”的物体。
不过视线有接近射线l的，也有接近射线m的，正如视野里也分左右。
所以根据看到的物体的位置，可以做出适当的调整。
假如看到的墙壁是与离射线m近的视线相交的话，那么为了回避墙壁，点A应当将视野向射线l方向旋转，反过来为了回避离射线l近的墙壁，视野应该向射线m方向旋转。
当然难免有这样一种情况：一堵墙壁直接横在点A的视野前，墙壁与视野的所有视线都相交。这种情况下并不能根据当前情况决定旋转方向，所以随机旋转方向即可。
通常在旋转过一次以后，这种全部相交的情况就可以化解，进而可以继续按回避的原则决定接下来的旋转方向。

如此以来，第一个问题得到解决了，那第二个问题呢？

为了回避墙壁，确实要选择不直接面向点B的方向继续前进，但是如果不在适当的时机再调整回到点B的方向的话，那么只会离点B越来越远。
那么这个时机是什么呢？

最直观的想法是：当视线不再与墙壁相交的时候，就可以调整回来了。

但是这样会造成下面这种情况：

<picture>
    <source type="image/webp" srcset="/p5js-self-propelled-points/if-no-sensor.webp">
    <source type="image/png" srcset="/p5js-self-propelled-points/if-no-sensor.png">
    <img src="/p5js-self-propelled-points/if-no-sensor.png" alt="回调时机" />
</picture>

假如点A在A1的位置，线段IJ为墙壁。由于视野中右侧出现墙壁，所以点A要像左（逆时针）旋转过后，运动至点A2的位置。
如果在视线不再与墙壁相交时就调整会到点B方向的话，势必会造成刚刚离开视野的墙壁又重新回到视野当中的情况。
或许再次触发回避，然后如此反复，终究能走到墙壁尽头的时候，但是反复摇摆的路线会让点A的运动轨迹很不自然。

墙壁已经离开视野，自然不需要再回避，但是又有什么办法能让已经离开视野的墙壁不会因为回调方向而重新回到视野呢？
需要增加措施只有墙壁刚刚离开视野的情况，如果墙壁已经离点A的视野足够远了，就不需要担心回调一次方向就导致墙壁回到视野了。
针对这个问题可以在两侧视线的延长线上新增一个“传感器”的范围。
传感器不直接影响点A的方向角，即使传感器与墙壁相交，点A也不会主动修改角度。
但是当点A开始根据点B调整角度的时候，要根据差分方向传感器的情况来判断是否要调整。

如上面点A2的例子，A2的视线已经不与墙壁相交了，但是因为右侧传感器仍与墙壁相交，所以维持当前方向，不向点B的方向（右侧）旋转角度。

将传感器逻辑放到伪代码中可得：

```
根据点A和点B的位置确定追踪方向角a;
确定移动步长n;
确定角度调整步长b;
确定墙壁w[m];
确定视野l[k];
while(点A没有接触到点B){
  // 如果点B为动点，在此处获取点B新的方向角
  if(l的成员与w的成员相交){
    根据墙壁的位置调整点A的方向角a;
  } else{
    计算点B方向角与当前方向角a的差分c;
    if(c不等于0且差分方向传感器不与墙壁相交){
      根据差分c旋转方向角，步长为b;
    }
    按追踪方向角a前进n;
  }
}
```

如果点B为动点，可在注释处添加获取点B新的方向角的逻辑。
具体的参数需要在实现中才能调整，不过追踪自走点的基本逻辑已经基本成型了。

---

# 2.代码实现

> 源代码：https://github.com/7mA/self-propelled-points

首先定义追踪自走点（`enemy`）的视线与传感器。

```js
// 自走点前方视线
let viewLine = Ray.getRayFromPoints(
  enemy.vp.pos,
  enemy.vp.pos.move(30, enemy.vp.angle)
)

// 自走点左侧视线
let leftViewLine = Ray.getRayFromPoints(
  enemy.vp.pos,
  enemy.vp.pos.move(30, enemy.vp.angle - PI / 6)
)

// 自走点右侧视线
let rightViewLine = Ray.getRayFromPoints(
  enemy.vp.pos,
  enemy.vp.pos.move(30, enemy.vp.angle + PI / 6)
)

// 自走点左侧传感器
let leftSensor = Ray.getRayFromPoints(
  enemy.vp.pos,
  enemy.vp.pos.move(50, enemy.vp.angle - PI / 6)
)

// 自走点右侧传感器
let rightSensor = Ray.getRayFromPoints(
  enemy.vp.pos,
  enemy.vp.pos.move(50, enemy.vp.angle + PI / 6)
)
```

上述代码中定义三条“视线”，两个“传感器”，其中传感器的方向与左右两侧视线同方向。
自走点的视野可以看作是左右两侧视野所夹的部分。

有了视线和传感器，就可以来感知周围墙壁的信息了。

```js
// 用左传感器探测墙壁
let leftNoIntersection = true;
for(let wall of wallList){
  if(leftSensor.intersection(wall) != null){
    leftNoIntersection = false;
    break;
  }
}

// 用右传感器探测墙壁
let rightNoIntersection = true;
for(let wall of wallList){
  if(rightSensor.intersection(wall) != null){
    rightNoIntersection = false;
    break;
  }
}

// 用视线探测墙壁
let forwardNoIntersection = true;
for(let wall of wallList){
  if(viewLine.intersection(wall) != null){
    forwardNoIntersection = false;
    break;
  }
  if(leftViewLine.intersection(wall) != null){
    forwardNoIntersection = false;
    break;
  }
  if(rightViewLine.intersection(wall) != null){
    forwardNoIntersection = false;
    break;
  }
}
```

传感器与视线在判断是否与墙壁相交上并没有什么不同。
左右传感器分别用一个变量存储状态，视线的探测结果集中到一个变量存储。
不同的变量在后面有不同的用处。

下面是判断角度调整与移动：

```js
// 调整自走点视线角度并前进
let playerWayVector = Ray.getRayFromPoints(enemy.vp.pos, player.pos).way;
if(playerWayVector.mag() > 20){
  if(!forwardNoIntersection){
    // 绘制阻塞自走点震动效果
    enemy.vp.pos = enemy.vp.pos.move(-enemySpeed, enemy.vp.angle);
    point(enemy.vp.pos.x, enemy.vp.pos.y);
    enemy.vp.pos = enemy.vp.pos.move(enemySpeed, enemy.vp.angle);
    if(leftNoIntersection == rightNoIntersection){
      enemy.vp.angle += PI / 6 * random([-1, 1]);
    } else if(!leftNoIntersection){
      enemy.vp.angle += PI / 15
    } else if(!rightNoIntersection){
      enemy.vp.angle -= PI / 15
    }
    enemy.vp.angle = backToBetween0To2Pi(enemy.vp.angle);
  } else{
    if(1 - enemy.vp.viewLineUnitVector.dotProduct(playerWayVector.unitize()) > 0.01){
      let dRadian = playerWayVector.angle - enemy.vp.angle;
      let clockwiseRotation = dRadian / abs(dRadian);
      if(abs(dRadian) < PI){
        if(clockwiseRotation == 1 && rightNoIntersection
          || clockwiseRotation == -1 && leftNoIntersection){
          enemy.vp.angle += clockwiseRotation * PI / 15;
        }
      } else{
        if(clockwiseRotation == -1 && rightNoIntersection
          || clockwiseRotation == 1 && leftNoIntersection){
          enemy.vp.angle -= clockwiseRotation * PI / 15;
        }
      }
      enemy.vp.angle = backToBetween0To2Pi(enemy.vp.angle);
    }
    enemy.vp.pos = enemy.vp.pos.move(enemySpeed, enemy.vp.angle);
  }
} else {
  stopGame(GAME_OVER_BY_TOUCH_ENEMY);
}

backToBetween0To2Pi = function(angle){
  while(angle >= 2 * PI){
    angle -= 2 * PI;
  }
  while(angle < 0){
    angle += 2 * PI;
  }
  return angle;
}
```

虽然当中夹杂了一些渲染逻辑和临界值模糊处理，但是大体上逻辑与伪代码相同。
首先在每次判断视线的阻塞情况（视线是否与墙壁相交）之前，先获得一次当前目标点的方向角（方向向量）。如果自走点没有接触到目标点（`playerWayVector.mag() > 20`）的话，进入角度调整与移动逻辑。
当自走点的视野中出现墙壁时（`!forwardNoIntersection`），自走点根据左右传感器的阻塞情况来判断墙壁在视野中的位置，进而计算角度的调整方向。
为了减少随机数带来的方向摇摆，左右都堵塞时的随机角度步长的极值要大于单侧阻塞的步长。
在判断点B方向角与当前自走点的方向角的差时，采用的是根据两个方向的单位向量的点积来判断的策略。
在进行方向调整的时候，要注意跨越0或者2π的时候的换算处理，要注意使角度维持在`[0, 2π)`区间内，否则在判断差分角大小时会有问题。
在方向调整完毕以后，朝向新的方向移动，直至自走点接触到目标点。

---

# 3.关于完成品

其实最开始没打算把这个逻辑做成一个完整的应用之类的，可能充其量就搞个代码片段贴到CodePen上看看效果就得了。
但是没想到最后还是扩展成了一个完整的程序，而且还是以[游戏的形式](https://7ma.github.io/self-propelled-points/)。

<picture>
    <source type="image/webp" srcset="/p5js-self-propelled-points/game.webp">
    <source type="image/png" srcset="/p5js-self-propelled-points/game.png">
    <img src="/p5js-self-propelled-points/game.png" alt="游戏封面" />
</picture>

最开始为了方便测试，给目标点加了一个鼠标拖拽事件的响应函数，能便捷地修改目标点的位置，模拟目标点移动的情况，尝试各种各样的场景。
同时也是为了提升测试覆盖率，墙壁也是用的随机数生成的。
后来发现拿鼠标拖的话目标点的移动速度不好掌握，于是又加了方向键移动和对应的移动速度。
在用方向键以固定速度移动的时候，就在想如果不光是一个点，要是有好多个点都在追目标点会怎么样呢？
就这样，基于这个想法，一点点往里填一些有游戏性和挑战性的内容，形成了最后的游戏形式的完成品。

除了上面的追踪逻辑和随机墙壁以外，游戏里还加了下面这些内容：

- 为追踪自走点（敌人）添加寿命，敌人会定期消亡，防止敌人因路径收敛出现扎堆追踪的现象。
- 添加道具，并且要求玩家在一定间隔内必须收集一个道具，让目标点（玩家）的游戏目的从只有躲避变成躲避与收集并行。
- 游戏难度（敌人寿命、敌人移动速度、敌人数量）随玩家的分数（收集道具和敌人消亡时获得）的提升而提升，同时玩家的移动速度也会随之提升。

因为敌人消亡之后会在随机的位置重生新的敌人，所以最开始的时候难免会撞到意想不到“闪现”出来的新生成的敌人，直接结束游戏。
为了改良这个体验问题，加了下面的渲染逻辑：

```js
// 绘制敌人
enemy = enemyList[i];
if(enemy.status == ENEMY_STATUS_GENERATED && (getCurrentTime() - enemy.generatedTime) / 500 > 1){
  enemy.status = ENEMY_STATUS_ACTIVE;
}if(enemy.status == ENEMY_STATUS_ACTIVE && (enemy.lifeSpan - (getCurrentTime() - enemy.generatedTime)) / 500 < 1){
  enemy.status = ENEMY_STATUS_EXPIRING;
}
stroke(lerpColor(generatedEnemyColor, expiringEnemyColor, (getCurrentTime() - enemy.generatedTime) / enemy.lifeSpan));
if(enemy.status == ENEMY_STATUS_GENERATED){
  push();
  strokeWeight((getCurrentTime() - enemy.generatedTime) / 500 * 20);
  point(enemy.vp.pos.x, enemy.vp.pos.y);
  pop();
  continue;
} else if(enemy.status == ENEMY_STATUS_EXPIRING){
  push();
  strokeWeight((enemy.lifeSpan - (getCurrentTime() - enemy.generatedTime)) / 500 * 20);
  point(enemy.vp.pos.x, enemy.vp.pos.y);
  pop();
  continue;
} else {
  point(enemy.vp.pos.x, enemy.vp.pos.y);
}
```

刚生成的敌人的点的大小，随时间由小到大，并且为之增加新的状态变量，使刚生成的敌人不做角度调整与移动行为，并且不做接触判断。
同时根据敌人的生成时间和寿命，用渐变色表示敌人的剩余寿命。玩家可以根据敌人的颜色估算消亡时机，进而规划合理路线。
为了一致化视觉效果，敌人消亡的时候也有同样的点的大小的变化，只不过是随时间从大到小。
包括道具的生成也有类似的视觉效果：

```js
// 绘制道具
if(item.status == ITEM_STATUS_GENERATED && (getCurrentTime() - item.generatedTime) / 500 > 1){
  item.status = ITEM_STATUS_ACTIVE;
}
stroke(itemColor);
if(item.status == ITEM_STATUS_GENERATED){
  push();
  strokeWeight((getCurrentTime() - item.generatedTime) / 500 * 20);
  point(item.pos.x, item.pos.y);
  pop();
  continue;
}
point(item.pos.x, item.pos.y);
```

当然，作为游戏，程序本身还有很多值得改良的地方。
比如随机地图的生成，地图的不同对游戏本身的体验会有不小的影响。但是现状下随机地图还只是简单的坐标随机数生成，只保证了不出现从左到右或者从上到下把整个地图分成完全隔离的两部分的情况。

```js
// 初次生成水平随机墙壁
while(horizontalWallList.length < 7){
  if(horizontalWallList.length == 0){
    horizontalWallList = [topEdge, bottomEdge];
  }
  let moveDistance = random(height);
  if(moveDistance == height / 2) continue;
  let tempWall = Ray.getRayFromPoints(
    new Vec(random(width), 0).move(moveDistance, PI / 2),
    new Vec(random(width), 0).move(moveDistance, PI / 2)
  )
  if(tempWall.intersection(leftEdge) != null && tempWall.intersection(rightEdge) != null) continue;
  horizontalWallList.push(tempWall);
}

// 初次生成垂直随机墙壁
while(verticalWallList.length < 7){
  if(verticalWallList.length == 0){
    verticalWallList = [leftEdge, rightEdge];
  }
  let moveDistance = random(width);
  if(moveDistance == width / 2) continue;
  let tempWall = Ray.getRayFromPoints(
    new Vec(0, random(height)).move(moveDistance, 0),
    new Vec(0, random(height)).move(moveDistance, 0)
  )
  if(tempWall.intersection(topEdge) != null && tempWall.intersection(bottomEdge) != null) continue;
  verticalWallList.push(tempWall);
}
```

简单的生成逻辑不光造成了生成的地图不够细腻精良的结果，而且还有出现导致不巧开场直接就被四个墙壁围起来这种开局即死的局面。此外还有在两条相距很近的平行墙壁之间生成的敌人会无法动弹的问题。

提到随机地图，有一个无法回避的话题就是Roguelike。不过Roguelike的地图首先得是网格状的，而且生成逻辑要复杂一些。
而这个程序中的墙壁只是一个有一定粗度的线段……这也造成了墙壁判断上的一个问题，就是玩家从墙壁边缘沿墙壁方向移动的话，是可以与墙壁重合的。
因为玩家判断是否可以沿上下左右某一方向移动的“视线”仅仅是一条指向该方向的线段，如果从墙壁边缘沿墙壁方向移动的话，墙壁没有“侧面”，视线大概率与墙壁所在的线段是平行无交点的，所以就可以前进。
但是重合以后是无法自由地从两侧离开墙壁，因为视线一定是位于墙壁的一侧的，所以向另一侧移动时就会被墙壁挡住而无法移动。
究其根本的原因，其实就是图像外观（实心圆、长矩形）与逻辑模型（点与线）的不一致造成的结果。图像上是有面积与宽度的二维图像，实际的运算中却只有一维属性。
如果墙壁有“侧面”的话，就可以通过探测侧面与视线的相交来避免这种问题的发生。
此外敌人方面偶尔也会因为随机数生成的粗糙地图而出现在特定位置“卡bug”的情况，详情会在后文解释。

另外还有难度递增函数、敌人随机生成位置（避免游戏开始时玩家与敌人的生成位置过近等问题）、UI等在游戏完成度方面仍然值得精雕细琢的地方。比如难度递增方面不应该采用线性提升的思路，因为这样会导致中期到后期（7,000分以上）的体感难度提升速度突然增大。我自己玩的最高分结果也只能局限在13,000分左右……不过这些地方的改良应该得在随机地图问题解决以后再做考量。


---

# 4.下一步

关于下一步的行动，首先是收集各种各样存在于程序当中的bug并且究明原因。

比如像上文提到的敌人在特定位置卡bug的问题，目前确认到的问题当中有这样一种情形：当有两条互相垂直但不相交（程序中只有水平墙壁和垂直墙壁，因此不平行即垂直）、边缘离得比较近的墙壁，敌人将无法自己脱离两个墙壁边缘附近。
这么说肯定不好明白，下面贴一张图。

<picture>
    <source type="image/webp" srcset="/p5js-self-propelled-points/corner-bug.webp">
    <source type="image/png" srcset="/p5js-self-propelled-points/corner-bug.png">
    <img src="/p5js-self-propelled-points/corner-bug.png" alt="角落bug" />
</picture>

如图所示，线段MN和线段PQ为两条墙壁，两条墙壁互相垂直但不相交，点N与点Q相距比较近，两条线段形成一个有缺口的“L”字。
点A为自走点，三条虚线代表三条视线，其中左侧和右侧视线与墙壁相交，这也就意味着这种情况下方向调整方式是随机决定的。

调整无非就两个方向：向左转或者向右转。
如果向左转的话，自走点右侧的视线会离开墙壁，而左侧与前方视线会随之与墙壁相交。因为是一个角落，所以一次旋转是无法完全摆脱的，因而就会进入下一次方向调整。

<picture>
    <source type="image/webp" srcset="/p5js-self-propelled-points/turn-left.webp">
    <source type="image/png" srcset="/p5js-self-propelled-points/turn-left.png">
    <img src="/p5js-self-propelled-points/turn-left.png" alt="向左转" />
</picture>

但是再次审视现在的情况，现在自走点的左侧视线与墙壁相交，右侧视线（和右侧传感器）不与墙壁相交，所以自走点会选择向右旋转。
但是刚刚左转过来的自走点又向右侧旋转的话，就会又回到刚刚两边视线都与墙壁相交的情况，然后如此循环反复。
向右转的话也会在转完以后再向左转回来，所以这种情形下自走点是无法脱离窘境的。

缓解策略可以是增加随机旋转的角度极值，给予自走点更大的回转空间（这个逻辑已经在代码中实现了），但是这并不能解决所有的情况。
也可以考虑更复杂的旋转策略算法，不过在解决这个问题之前，在地图生成（以及其他物件的随机生成逻辑）上避免这种逼仄的布局与结果，用矩形代替线作为地图内墙壁的几何模型，都可以从根本上避免类似问题的发生。
所以包括Roguelike的实现在内的随机生成和几何模型的优化与重构，从消除bug的角度上来讲，也应该是比较优先的事项。

此外，程序用到的逻辑通过调整参数也可以实现出自机狙的效果，进而推广到子弹运动、弹幕生成也不是不可能。
所以核心逻辑的代码也可以考虑适当封装，以备复用。

---

**Reference Source:**

1. [高校数学とJavaScriptだけ。FPSの作り方 #1【ゲームプログラミング】【ゲーム開発】](https://www.youtube.com/watch?v=Mtf4rz9UEQo)
