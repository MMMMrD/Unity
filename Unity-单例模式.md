# Unity-单例模式

​	对于某些特殊的类，我们希望 **在整个程序的生命周期只创建一个该类的对象** ，或是希望==在不拖拽的情况下就可以调用该类中的函数==，我们就需要将这个类写成 ==单例模式==

~~~c#
public class Test()
{
	pubic abstract Test Instance;//创建程序中该类的唯一一个对象
    
    private void Awake()//在程序启动时，直接创建一个该类的对象
    {
        if(Instance!=null)//如果已经创建了一个变量，便将该变量删除
        {
            Destroy(gameObject);
        }

        Instance = this;
    }
}
~~~



## 泛型单例

​	在通常的游戏制作中，我们会创建许多的 **Manager** 来管理我们的游戏，如果每创建一个 **Manager** 都要重新写一遍单例会非常麻烦，因此就需要泛型单例

​	泛型单例，顾名思义就是事先不知道单例的类型，其类型取决于继承该泛型父类的子类

​	代码如下：

~~~c#
public class Singleton<T> : MonoBehaviour where T :Singleton<T> //where确定继承该父类的是什么类型的子类
{
    private static T instance;

    //用于获取单例的属性
    public static T Instance
    {
        get { return instance;}
    }

    //程序启动时调用，给单例唯一变量赋值
    protected virtual void Awake()
    {
        if(instance != null)
        {
            Destroy(gameObject);
        }
        else
        {
            instance = (T)this;
        }
    }

    //判断单例是否已经被创建的属性
    public static bool IsInitialized
    {
        get { return instance != null;}
    }
    
    //销毁函数
    protected void OnDestroy() 
    {
        if(instance == this)
        {
            instance = null;
        }    
    }
}
~~~
