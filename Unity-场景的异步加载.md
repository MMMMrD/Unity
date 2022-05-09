# Unity-场景的异步加载



## 为什么需要异步加载

​	在诸多大型游戏里，场景渲染精度都是动态的，随着场景与角色距离的增加，渲染精度也在递减，这样极大的减少了硬件性能的消耗。

​	但如果角色使用了某些传送技能，将自己传送到为渲染的地点，游戏可能就会因为需要瞬间渲染大量的场景而卡顿。此时就需要用到 **场景的异步加载** 了。

​	**异步加载**，指的就是在加载角色之前，事先将角色周围的场景渲染好，防止角色传送后出现严重的卡顿，以提高玩家的游戏体验。



## 场景加载构想

​	在提供直接的代码之前，我们需要对基本的传送进行一个简单的构思。首先我们需要一个**传送门**以确定我们 **能够触发传送** 的地点；传送门前还需要设置一点以确定 **传送地点**，并将其作为传送门的子物体。

​	注意这两者的区别：**传送门** 只是确定 **触发传送的地点**，而角色需要前往的地点由 **传送门前的点** 决定。

​	编写代码，分别给传送门与传送点设置对应的属性（传送门：设置**传送模式、传送场景、传送终点名字**，以及编写 **判断何时传送的函数**）（传送点：设置**传送点名字**），这样简易的传送门就设置好了。

​	关键在于场景传送代码的逻辑：

>1. 我们需要创建一个控制场景的代码，因为Unity自带 SceneManager 类，所以我们在这里将场景控制的代码命名为 **SceneController** ，同样将其写成单例模式。 
>2. 就如同 GameManager 一样，**SceneController** 也一样不需要自己执行代码，需要别的类在外部调用。我们只需要在 **SceneController** 中写上必要的参数（**PlayerPrefab**：player的预制体，用于在其他场景生成角色），一个进行 **异步加载的协程**，一个 **查找传送终点** 的函数，一个 **传送函数** 供传送门调用即可。
>3. **SceneController** 中的传送函数需要用到传送门的参数，所以传送函数需要一个 **传送门类** 作为参数。传送开始前，首先需要**判断传送类型**（同场景 or 异场景），然后将 **传送终点名、终点场景名** 传给协程，同时不论同异场景，都需要在协程中将 传送终点名 传给 查找传送终点位置的函数，以**获得传送终点的位置信息**，在获得终点信息后才能进行**场景的异步加载**。



## 实例代码

​	在上面的简单构思中，已经对基本的传送逻辑进行了简单的梳理，现在就是将逻辑实现的时候了。

### 传送目标点

​	传送终点的代码十分简单，只是通过枚举（enum）对传送点进行记录，枚举要在Unity窗口中自行选择

~~~c#
public class TransitionDestination : MonoBehaviour
{
    public enum DestinationTag //设立目标点名枚举
    {
        ENTER, A, B, C
    }

    public DestinationTag destinationTag;//目标点名枚举变量，在Unity窗口中设置
}
~~~



### 传送门（可传送地点）

​	传送门需要判定角色是否在可传送范围内，如果是，则可以在按下某键时进行传送

~~~c#
public class TransitionPoint : MonoBehaviour
{
    public enum TransitionType //创建传送类型的枚举
    {
        SameScene, DifferentScene
    }

    [Header("Transition Info")]
    public string sceneName;//以字符串形式记录 终点场景名
    
    public TransitionType transitionType;//传送类型枚举变量，需要在Unity窗口中设置
    
    public TransitionDestination.DestinationTag destinationTag;//设置传送终点名
    
    private bool canTrans;	//判断是否可传送
    
    private void Update() 
    {
        if(Input.GetKeyDown(KeyCode.E) && canTrans)
        {
            if(SceneController.Instance != null)
            {
                //调用SceneController中的传送函数
                SceneController.Instance.TransitionToDestination(this);
            }
        }
    }

    //利用碰撞体判断Player是否处在可传送位置，是则将可传送Bool值设置为true，否则设置为false
    private void OnTriggerStay(Collider other) 
    {
        if(other.gameObject.CompareTag("Player"))
        {
            canTrans = true;
        }
    }

    private void OnTriggerExit(Collider other) 
    {
        if(other.gameObject.CompareTag("Player"))
        {
            canTrans = false;
        }    
    }
}
~~~



### ScenController

​	SceneController 的详细介绍已经在场景转换逻辑写清楚了，在这不赘述

~~~c#
public class SceneController : Singleton<SceneController>
{
    public GameObject playerPrefab;//Player的预制体，在Unity窗口中赋值
    GameObject player;
    NavMeshAgent playerAgent;

    protected override void Awake()
    {
        base.Awake();
        DontDestroyOnLoad(this);
    }

    //由传送门调用，由传送门将自身当作参数传递过来，获得传送门的属性
    public void TransitionToDestination(TransitionPoint transitionPoint)
    {
        //判断传送模式
        switch(transitionPoint.transitionType)
        {
            //场景内传送
            case TransitionPoint.TransitionType.SameScene:
                
                //获得当前场景名后，将场景名与目标点传给协程
                StartCoroutine(Transition(SceneManager.GetActiveScene().name, transitionPoint.destinationTag));
                
                break;
            //场景外传送
            case TransitionPoint.TransitionType.DifferentScene:
                
                //将Unity窗口中设置的场景名与目标点传给协程
                StartCoroutine(Transition(transitionPoint.sceneName, transitionPoint.destinationTag));
                break;
        }
    }

    //场景加载协程
    IEnumerator Transition(string sceneName, TransitionDestination.DestinationTag destinationTag)
    {
		//首先判断是不是异场景传送
        if(SceneManager.GetActiveScene().name != sceneName)
        {
            //异步加载加载场景
            yield return SceneManager.LoadSceneAsync(sceneName);
            //场景加载完毕后，调用获取目标点的函数获得目标点，将Player生成在该点
            yield return Instantiate(playerPrefab, GetDestination(destinationTag).transform.position, GetDestination(destinationTag).transform.rotation);
            yield break;
        }
        else
        {
            //进行初始化，获得 Player 及其 NavMeshAgent
            player = GameManager.Instance.playerStats.gameObject;
            playerAgent = player.GetComponent<NavMeshAgent>();
            playerAgent.isStopped = true;
            //调用函数获得目标点的信息，并传送到该点 
            player.transform.SetPositionAndRotation(GetDestination(destinationTag).transform.position, GetDestination(destinationTag).transform.rotation);
            yield return null;
        }
    }

    //查找传送终点函数
    private TransitionDestination GetDestination(TransitionDestination.DestinationTag destinationTag)
    {
        var entrances = FindObjectsOfType<TransitionDestination>();//获取场景内所有的传送目标点
        //遍历目标点数组
        foreach (var item in entrances)
        {
            if(item.destinationTag == destinationTag)
            {
                return item;//查找到后将其return
            }
        }

        return null;
    }
}
~~~
