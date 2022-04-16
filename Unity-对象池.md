# Unity-对象池



## 简介

​	在制作游戏的过程中，人物和boss的设计往往会有释放多个子弹的攻击方式。我们可以用直接创造子弹然后销毁的办法来实现这些技能的效果，但当子弹开始变多，游戏就会不断的消耗我们的内存。为了解决这个问题，开发者们就引入了状态机。



## 创建思路



### 框架

​	在不创建新的实例的前提下，我们应该怎么实现多个实例同时出现的效果呢？其实我们只要将之前创建过的实例充分利用起来就可以了。

​	首先还是要创建足够的实例，并将这些实例保存同一个父类对象下，我们将其称为 **对象池**。对象池中的子类，我们只用在需要时候开启，不需要的时候关闭，这样就实现了与之前一样的效果。



### 实现（三个函数）

​	首先我们需要注意的是，在同一个场景中，同一个对象，我们只希望拥有一个保存它实例的对象池。因此我们要将对象池写成单例模式，保证在一个场景中只能有一个对应的对象池。

​	其次，我们需要在 **游戏开始** 的时候给对象池 **创建足够的对象**，保证在角色在使用对象池的时候，对象池中的对象足够多，防止程序报错。因此我们需要一个 **初始化对象池的函数** 以在游戏开始的时候使用对应对象的 GameObject 填充对象池。

​	其次我们还要考虑如何让外部访问我们的对象池。我们知道，用单例模式创建的唯一对象是可以在外部直接访问的，但是这里为了保护代码中的各个参数，我们在实际操作中要避免直接访问对象池的唯一对象。对此，我们可以创建两个函数，一个是 **获得对象池中的对象的函数**，一个是 **将对象返回对象池的函数**。



### 细节

​	为实现对象进出对象池的设想，我们需要使用队列 **(Queue)**，当对象进入对象池的同时进入队列，离开对象池的同时退出队列。

​	其次我们需要注意，我们在游戏开始的时候只创建了有限数量的对象，在某些特殊情况下可能不足以满足外部的使用。因此我们需要在 **获得对象池的对象函数（getFromPool）** 中需要进行一个判断，当对象池中对象的数量**（或者说队列中对象的数量）**为零时，将再次调用我们的 **初始化对象池的函数**，保证对象池中对象的个数能够满足外部的使用。

​	同时，单独一个对象池的代码还不足以实现最开始提到的功能，还需要配合我们对象的代码，在合适的时候调用对象池提供的 **三个函数**。因为想要实现的功能千差万别，在这里只举一个简单的例子：

​	#角色攻击同时调用对象池的 **getFromPool** 函数获得子弹，子弹飞行足够时间/攻击到敌人时自动调用 **ReturnPool** 函数返回对象池。



## 实例

​	下面是一个子弹对象池的代码实例

~~~c#
public class ArrowPool : MonoBehaviour
{
    public static ArrowPool instance;

    public GameObject ArrowPrefab;

    public int ArrowCount;

    //
    public Queue<GameObject> availableObject = new Queue<GameObject>();
    
    void Awake()
    {
        instance = this;
    }

    public void FillPool()
    {
        for(int i=0;i < ArrowCount;i++)
        {
            var newArrow = Instantiate(ArrowPrefab);
            newArrow.transform.SetParent(transform);

            //取消启用，返回对象池
            ReturnPool(newArrow);
        }
    }

    public void ReturnPool(GameObject gameObject)
    {
        gameObject.SetActive(false);
        
        availableObject.Enqueue(gameObject);
    }

    public GameObject getFromPool()
    {
        if(availableObject.Count == 0)
        {
            FillPool();
        }

        var outObject = availableObject.Dequeue();
        outObject.SetActive(true);

        return outObject;
    }
}
~~~



​	