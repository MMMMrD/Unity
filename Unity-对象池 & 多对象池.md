# Unity-对象池 & 多对象池



## 简介

​	在制作游戏的过程中，人物和boss的设计往往会有释放多个子弹的攻击方式。我们可以用直接创造子弹然后销毁的办法来实现这些技能的效果，但当子弹开始变多，游戏就会不断的消耗我们的内存。为了解决这个问题，开发者们就引入了状态机。



## 普通对象池



###  创建思路

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



### 代码实例

​	下面是一个子弹对象池的代码实例

~~~c#
public class ArrowPool : MonoBehaviour
{
    public static ArrowPool instance;//单例
    public GameObject ArrowPrefab;//子弹预制体
    public int ArrowCount;//池中初始子弹个数

    //子弹队列
    public Queue<GameObject> availableObject = new Queue<GameObject>();
    
    void Awake()
    {
        instance = this;
    }

    //填充池子
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

    //出队函数
    public void ReturnPool(GameObject gameObject)
    {
        gameObject.SetActive(false);
        availableObject.Enqueue(gameObject);
    }
    
    //入队函数
    public GameObject getFromPool()
    {
        if(availableObject.Count == 0)
        	FillPool();
        
        var outObject = availableObject.Dequeue();
        outObject.SetActive(true);
        return outObject;
    }
}
~~~



## 多对象池



### 创建思路

​	不同于普通对象池，**多对象池不需要挂载在任何对象上**，所以不需要继承 MonoBehaviour。同时，因为我们没有事先创建好对象池物体，且我们**需要根据物体的信息将其与对应的对象池挂钩**，所以我们不能对对象只进行简单的入队出队操作，还需要在此基础上，将**对象的预制体当作参数传入出队及入队函数**，并根据预制体信息，**创建对应的子对象池**，并通过**字典**将其与对应的子对象池挂钩。

​	总结一下，我们创建多对象池的时候需要完成以下步骤：

>1. 将多对象池写成**单例**，且**不继承 MonoBehaviour**
>2. 创建一个**字典**变量记录物体对应的子对象池
>2. 给入队函数以及出队函数**添加需要的预制体变量**
>2. 同时，在调用出队函数的时候，若**对象池物体为空**，则根据层级关系**创建新的对应对象池**

​	***（更多细节已经在下面的代码实例中写明***



### 代码实例

~~~c#
public class ObjectPool
{
    private static ObjectPool instance;
    
    //Static变量的属性，用于获得单例实例
    public static ObjectPool Instance
    {
        get
        {
            if(instance == null)
                instance = new ObjectPool();
            return instance;
        }
    }
    //创建字典,以键值查询队列
    private Dictionary<string, Queue<GameObject>> objectPool = new Dictionary<string, Queue<GameObject>>();
    
    //为了不让窗口杂乱,声明一个GameObject作为所有物体的父物体
    private GameObject pool;


    //出队函数
    public GameObject GetObject(GameObject prefab)
    {
        GameObject _object;//声明一个过程量,供后续操作使用

        if(!objectPool.ContainsKey(prefab.name) || objectPool[prefab.name].Count == 0)//是否拥有对应子对象池 or 子对象池中是否还有对象
        {
            
            _object = GameObject.Instantiate(prefab);//满足条件则创建对象
            PushObject(_object);          //并且返回对象池

            if(pool == null)
                pool = new GameObject("ObjectPool");//没有父类物体则创建,将该语句放这是为了节省资源,不用每次都对是否存在Pool进行判断
            
            //如果没有对应的子对象池则,则将其创建并设置为Pool的子对象
            GameObject ChildPool = GameObject.Find(prefab.name + "Pool");
            if(!ChildPool)
            {
                ChildPool = new GameObject(prefab.name + "Pool");
                ChildPool.transform.SetParent(pool.transform);
            }

            //将新创建的对象设置为对应对象池的子类
            _object.transform.SetParent(ChildPool.transform);
        }

        //根据传入的预制体,从字典中查找相应队列,并让对象出队
        _object = objectPool[prefab.name].Dequeue();
        _object.SetActive(true);
        return _object;
    }

    //入队函数
    public void PushObject(GameObject prefab)
    {
        string _name = prefab.name.Replace("(Clone)", string.Empty);//更改子物体的名字

        //若没有对应的对象池,则根据键值创建对应的队列,并添加到字典中
        if(!objectPool.ContainsKey(_name))
            objectPool.Add(_name, new Queue<GameObject>());

        //根据传入预制体,从字典中查找相应队列,并让对象出队
        objectPool[_name].Enqueue(prefab);
        prefab.SetActive(false);
    }
}
~~~



