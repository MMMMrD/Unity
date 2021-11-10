# Unity中Class(类)的调用

*创建的父类中的代码，可以被其他脚本调用*

*可以将几个 gameObject 的脚本中代码的共通部分写入父类，实现==代码简化==*

## *公用死亡代码写法*：

### 父类死亡代码

~~~
protected virtual void Start()
{
	
}
~~~

### 子类继承代码

~~~
protected override void Start()
    {
        base.Start();
   	}
~~~

