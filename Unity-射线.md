# Unity-射线

## 射线

​	射线需要用到Unity自带的结构体 *RaycastHit2D* ，具体使用方法：

~~~c#
	RaycastHit2D (变量名) = Physics2D.Raycast(位置, 方向, 距离, 图层);
	if((变量名)) //该变量为bool变量，可用于判断
    {.......}
~~~

​	该代码的意思是，再 **指定位置** 朝向 **指定方向** 创建一个 **指定长度** 的射线，当射线碰撞到 **指定图层** 时，返回 **true** ，实际例子如下：

~~~c#
RaycastHit2D leftCheck = Raycast(new Vector2(-footOffset,-0.9f), Vector2.left, wallCheck, Ground);
~~~

 

​	当然，Physics2D.Raycast() 还有很多种写法，可以运用于不同的场景，这里时间有限只学了一个

