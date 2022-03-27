# Unity-射线与重载

## 射线

​	射线需要用到Unity自带的结构体 *RaycastHit2D* ，具体使用方法：

~~~c#
	RaycastHit2D (变量名) = Physics2D.Raycast(位置, 方向, 距离, 图层);
	if((变量名)) //该变量为bool变量，可用于判断
    {.......}
~~~

​	当然，Physics2D.Raycast() 还有很多种写法，可以运用于不同的场景，这里时间有限只学了一个

## 重载



