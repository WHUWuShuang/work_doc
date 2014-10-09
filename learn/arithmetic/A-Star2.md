转：http://bbs.9ria.com/thread-95620-1-1.html
 
对于A\*传统寻路的结果不平滑的问题，我们讨论了一种判断两点间是否存在障碍物的算法，并在用户点击鼠标选择了目的地后先判断起终点间是否存在障碍物，若不存在，则路径数组中将只具有一个终点节点；否则进行A\*寻路运算。大致过程可用下面代码表示：
 
```java
//判断起终点间是否存在障碍物，若存在则调用A\*算法进行寻路，通过A\*寻路得到的路径是一个个所要经过的节点数组；否不存在障碍则直接把路径设置为只含有一个终点元素的数组  
  
var hasBarrier:Boolean = _grid.hasBarrier(startPosX, startPosY, endPosX, endPosY);  
  
if( hasBarrier )  
{  
      _grid.setStartNode(startPosX, startPosY);  
      _grid.setEndNode(endPosX, endPosY);  
      findPath();  
}  
  
else  
{  
        _path = [_grid.getNode(endPosX, endPosY)];  
       _index = 0;  
       addEventListener(Event.ENTER_FRAME, onEnterFrame);//开始行走  
}  
 ```
如果你听从了我的建议这么做了，我只能说声抱歉，因为在实际应用中发现了一些问题，什么问题呢？就是当起终点间不存在障碍物时，主角行走的目的地将跳过 A\*寻路算法的“考验”，直接设置为我点击的位置(endPosX, endPosY)，因此，此位置若有时候会是路径网格外的某一点，主角也以此为目的地进行移动，这样导致的结果就是主角“飞”起来了，或是主角“穿越” 了……为了避免这个问题的发生，我们就必须时时刻刻地使用A\*算法去计算路径，即使起终点间没有障碍物也必须调用寻路方法进行寻路。但是如果用A\*寻路算 法计算出来的路径不平滑怎么办呢？别急，下面我们来讨论A\*寻路算法的路径平滑处理办法：弗洛伊德路径平滑算法。这个算法出之lizhi写的文章：http://wonderfl.net/c/aWCe ，并已被很多人采纳~我们先看一下结果演示：[MyPathFinding2](http://we.zuisg.com/wp-content/uploads/2011/08/MyPathFinding2.swf) 

弗洛伊德路径平滑算法应在通过A\*寻路算法得出路径后进行，它的步骤分为两步：一、合并路径数组中共线的节点；二、尽可能地去掉多余拐点。这个过程如下图所示：

原始A\*寻路路径

![](http://dl.iteye.com/upload/attachment/0072/1372/705c4cdb-43d9-3ace-9f1d-6f2f44e14062.jpg)

 去掉共线点

![](http://dl.iteye.com/upload/attachment/0072/1374/abc3c2e2-e549-309e-af3a-1dbd893c1520.jpg)

 去掉多余拐点

![](http://dl.iteye.com/upload/attachment/0072/1376/76037a30-7b0c-377e-a156-f2ef62799332.jpg)

可以看到，使用弗洛伊德路径平滑处理 后的路径正如我们期望的那样，而且大大削减了路径数组中的节点数目。

那么接下来来讲讲实现思路吧。首先，不难发现，若存在三点A(1,1), B(2,2), C(3,3)，若B与A的横、纵坐标差值分别等于C与B的横、纵坐标差值，则A,B,C三点共线，使用代码来表示就是：
 
 
```java
if( (bx -ax == cx - bx) && (by-ay == cy - by) )  
{  
  
//三点共线  
  
}  
 ```
由上式可知去掉路径中共线节点的方法。接下来讨论如何去掉多余的拐点。
仔细观察第三幅图你会发现，若路径中存在节点A,B,C,D,E,F,G，如果A与G之间的连线所经过的节点中没有一个节点是不可移动节点，则我们称A与 G之间是不存在障碍物的。如两节点间不存在障碍物，则可以去掉此两点间其他所有节点。如上例中A-G这些节点，若A与G之间不存在障碍物，则我们可以去掉 A与G之间的B,C,D,E,F节点，最终路径数组中只剩下A与G两个节点。那么如何判断两个点之间存不存在障碍物呢？在上一篇的教程中我已详细解释过方 法，列位道友若不记得了，可以再回头再去仔细研读一二。
        那么最后我们使用代码来实现弗洛伊德路径平滑处理的算法：
 
 
```java
/** 弗洛伊德路径平滑处理 */  
public function floyd():void {  
        if (path == null)  
                return;  
        _floydPath = path.concat();  
        var len:int = _floydPath.length;  
        if (len > 2)  
        {  
                var vector:ANode = new ANode(0, 0);  
                var tempVector:ANode = new ANode(0, 0);  
                //遍历路径数组中全部路径节点，合并在同一直线上的路径节点  
                //假设有1,2,3,三点，若2与1的横、纵坐标差值分别与3与2的横、纵坐标差值相等则  
                //判断此三点共线，此时可以删除中间点2  
                floydVector(vector, _floydPath[len - 1], _floydPath[len - 2]);  
                for (var i:int = _floydPath.length - 3; i >= 0; i--)  
                {  
                        floydVector(tempVector, _floydPath[i + 1], _floydPath[i]);  
                        if (vector.x == tempVector.x && vector.y == tempVector.y)  
                        {  
                                _floydPath.splice(i + 1, 1);  
                        }  
                        else  
                        {  
                                vector.x = tempVector.x;  
                                vector.y = tempVector.y;  
                        }  
                }  
        }  
        //合并共线节点后进行第二步，消除拐点操作。算法流程如下：  
        //如果一个路径由1-10十个节点组成，那么由节点10从1开始检查  
        //节点间是否存在障碍物，若它们之间不存在障碍物，则直接合并  
        //此两路径节点间所有节点。  
        len = _floydPath.length;  
        for (i = len - 1; i >= 0; i--)  
        {  
                for (var j:int = 0; j <= i - 2; j++)  
                {  
                        if ( _grid.hasBarrier(_floydPath[i].x, _floydPath[i].y, _floydPath[j].x, _floydPath[j].y) == false )  
                        {  
                                for (var k:int = i - 1; k > j; k--)  
                                {  
                                        _floydPath.splice(k, 1);  
                                }  
                                i = j;  
                                len = _floydPath.length;  
                                break;  
                        }  
                }  
        }  
}  
 ```
接下来再讲一讲第二个棘手的问题，就是A\*寻路会当你点击一个不可移动点之后返回false的寻路结果，即告知你无路可走。这不是我们想要的结果，我们想 要的是点击一个不可移动点后玩家应该走到离此不可移动点最近的一个可移动点位上面。那么如果你阅读过我的上一篇帖子，你会看到我解决此问题的一个方式是使 用“将不可移动点设置超大代价法”。不过使用此方法的一个弊病在于当你在选择一个不可移动点作为终点后，A\*寻路算法会遍历大量的点，造成性能的低下。那 么在经过另一番研究后发现还有一种办法能解决这个问题，就是“寻找替代点法”。
先解释一下什么是“寻找替代点法”，当点击一个不可移动点U之后我们将寻找此不可移动点外围一个离起点距离最短的可移动点R作为U的替代点，替代U来完成寻路且玩家将最终移动到此R点位置。如下图所示

![](http://dl.iteye.com/upload/attachment/0072/1378/825eb2e2-66fe-3736-9b23-7c9b57f70738.jpg)
 
那么，为了尽可能地降低寻找替代点的时间，我们提出了一种“埋葬深度”的概念。先看下图：

![](http://dl.iteye.com/upload/attachment/0072/1380/a7b62524-711c-30e3-8341-c4ffd3aa312e.jpg)

这种情况下，U点由一圈甚至两圈或更多圈不可移动点包围着，若我们采用传统的方式以辐射型遍历U点外圈节点（从U点外围第一圈开始遍历，遍历完毕没有找 到替代点，继续再遍历更外边的圈直到找到为止），将会在多次点击同一点时产生冗余遍历时间。那么此时我们为了降低重复遍历的次数，就引入了“埋葬深度的概 念”。若一个节点为不可移动点，则其埋葬深度为1；在此基础上，若其周围一圈全部为不可移动点，则其埋葬深度加一，为2；若更外围一圈依然全部为不可移动 点，埋葬深度再加一，依次类推，下图列出了埋葬深度为1-3的情况：

![](http://dl.iteye.com/upload/attachment/0072/1386/7e8fc46a-4f06-31a2-a3c5-5a53a840aed5.png)

在为某一个节点第一次寻找替代点时以辐射型遍历此点周围节点以计算出此节点的埋葬深度并记录于每个节点对象Node类中新增的一个埋葬深度属性buriedDepth中，下一次为此点寻找替代点时就可以根据埋葬深度从存在可移动点的那一圈遍历

![](http://dl.iteye.com/upload/attachment/0072/1388/8091ee58-572d-3ca0-9010-cecce1caf167.jpg)

在遍历原始终点U周围存在可移动点的那一圈之后把所有可移动点都存到一个数组中，之后比较此数组中全部候选点与起点的距离，选出距离最短的一个点作为替代点R即可。

![](http://dl.iteye.com/upload/attachment/0072/1390/59c65594-a9a1-323e-a889-0806a3d43e80.jpg)

好吧，寻找替代点的过程大致就是这样，最后来看代码呗。先看节点类ANode类中一些要用到的新增属性和方法：
 
```java
public class ANode  
{  
        .....  
        /** 埋葬深度 */  
        public var buriedDepth:int = -1;  
  
        /** 距离 */  
        public var distance:Number;  
  
        .....  
        /** 得到此节点到另一节点的网格距离 */  
        public function getDistanceTo( targetNode:ANode ):Number  
        {  
                var disX:Number = targetNode.x - x;  
                var disY:Number = targetNode.y - y;  
                distance = Math.sqrt( disX * disX + disY * disY );  
                return distance;  
        }  
}  
``` 
再看节点网格NodeGrid类中新增方法。
 
```java
/**当终点不可移动时寻找一个离原终点最近的可移动点来替代之 */  
public function findReplacer( fromNode:ANode, toNode:ANode ):ANode  
{  
        var result:ANode;  
        //若终点可移动则根本无需寻找替代点  
        if( toNode.walkable )  
        {  
                result = toNode;  
        }  
        //否则遍历终点周围节点以寻找离起始点最近一个可移动点作为替代点  
        else  
        {  
                //根据节点的埋葬深度选择遍历的圈  
                //若该节点是第一次遍历，则计算其埋葬深度  
                if( toNode.buriedDepth == -1 )  
                {  
                        toNode.buriedDepth = getNodeBuriedDepth( toNode, Math.max(_numCols, _numRows) );  
                }  
                var xFrom:int = toNode.x - toNode.buriedDepth < 0 ? 0 : toNode.x - toNode.buriedDepth;  
                var xTo:int = toNode.x + toNode.buriedDepth > numCols - 1 ? numCols - 1 : toNode.x + toNode.buriedDepth;  
                var yFrom:int = toNode.y - toNode.buriedDepth < 0 ? 0 : toNode.y - toNode.buriedDepth;  
                var yTo:int = toNode.y + toNode.buriedDepth > numRows - 1 ? numRows - 1 : toNode.y + toNode.buriedDepth;                  
  
                var n:ANode;//当前遍历节点  
  
                for( var i:int=xFrom; i<=xTo; i++ )  
                {  
                        for( var j:int=yFrom; j<=yTo; j++ )  
                        {  
                                if( (i>xFrom && i<xTo) && (j>yFrom && j<yTo) )  
                                {  
                                        continue;  
                                }  
                                n = getNode(i, j);  
                                if( n.walkable )  
                                {  
                                        //计算此候选节点到起点的距离，记录离起点最近的候选点为替代点  
                                        n.getDistanceTo( fromNode );  
  
                                        if( !result )  
                                        {  
                                                result = n;  
                                        }  
                                        else if( n.distance < result.distance )  
                                        {  
                                                result = n;  
                                        }  
                                }  
                        }  
                }  
  
        }  
        return result;  
}  
/** 计算一个节点的埋葬深度 
 * @param node                欲计算深度的节点 
 * @param loopCount        计算深度时遍历此节点外围圈数。默认值为10*/  
private function getNodeBuriedDepth( node:ANode, loopCount:int=10 ):int  
{  
        //如果检测节点本身是不可移动的则默认它的深度为1  
        var result:int = node.walkable ? 0 : 1;  
        var l:int = 1;  
  
        while( l <= loopCount )  
        {  
                var startX:int = node.x - l < 0 ? 0 : node.x - l;  
                var endX:int = node.x + l > numCols - 1 ? numCols - 1 : node.x + l;  
                var startY:int = node.y - l < 0 ? 0 : node.y - l;  
                var endY:int = node.y + l > numRows - 1 ? numRows - 1 : node.y + l;                  
  
                var n:ANode;  
                //遍历一个节点周围一圈看是否周围一圈全部是不可移动点，若是，则深度加一，  
                //否则返回当前累积的深度值  
                for(var i:int = startX; i <= endX; i++)  
                {  
                        for(var j:int = startY; j <= endY; j++)  
                        {  
                                n = getNode(i, j);  
                                if( n != node && n.walkable )  
                                {  
                                        return result;  
                                }  
                        }  
                }  
  
                //遍历完一圈，没发现一个可移动点，则埋葬深度加一。接着遍历下一圈  
                result++;  
                l++;  
        }  
        return result;  
}  
``` 
 
那么最后，看看实际应用这个方法进行寻路的部分吧。
 
```java
private function onGridClick(event:MouseEvent):void  
{  
        var startTime:int = getTimer();  
  
        var startPosX:int = Math.floor(_player.x / _cellSize);  
        var startPosY:int = Math.floor(_player.y / _cellSize);  
        var startNode:ANode = _grid.getNode(startPosX, startPosY);  
  
        var endPosX:int = Math.floor(event.localX / _cellSize);  
        var endPosY:int = Math.floor(event.localY / _cellSize);  
        var endNode:ANode = _grid.getNode(endPosX, endPosY);  
  
        if( endNode.walkable == false )  
        {  
                replacer = _grid.findReplacer(startNode, endNode);  
                if( replacer )  
                {  
                        endPosX = replacer.x;  
                        endPosY = replacer.y;  
                }  
        }  
  
        _grid.setStartNode(startPosX, startPosY);  
        _grid.setEndNode(endPosX, endPosY);  
  
        findPath();  
  
}  
```
 
OK, 这就是本次教程全部内容了，使用“弗洛伊德路径平滑处理”结合“寻找替代点法”做寻路，不论效率还是结果都还算如人意啦。不过，细心的朋友可以发现，“寻找替代点法”的弊端在于无法在存在“孤岛”和“回”字形的路径网格中找到正确的替代点，如下图：

![](http://dl.iteye.com/upload/attachment/0072/1392/c3997689-f368-306e-8501-98e1994660d8.jpg)


所以为了避免此情况，要么就用上一篇帖子中提到的“极大代价法”来进行寻路（结果准确，效率过低），要么就在布置路径网格时刻意避免编出这种形式的路径网格。
最后，奉上全部测试用代码，希望对列位有所帮助： (附件)