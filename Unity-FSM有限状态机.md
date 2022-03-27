# Unity-FSM有限状态机



## 什么是有限状态机？

​	在编写一些需要判断多个条件的程序时，我们常常会用到 **if-else** 语句，这样能够很好的帮我们解决多数问题。但在游戏开发过程中，一个角色的行为不是一成不变的，需要实时的进行修改，此时如果我们使用的是 **if-else** 来判断角色所处状态，就需要修改整个代码体，十分麻烦。而有限状态机很好的解决了这一些问题。

​	有限状态机实现的方式是，将判断条件、角色状态、状态机分别封装成一个类，这样当我们需要增加或者减少角色的状态时，直接将对应的状态与相对应的条件删除即可，不会影响到所有的代码，极大的减少了开发时间。

​	下面将介绍有限状态机该如何编写。



## 有限状态机代码编写

​	如上所述，我们需要将代码变成三个部分，判断条件、角色状态、状态机，三个部分对应三个基类。下面将一一介绍。



### 判断条件(Trigger)

​	条件类需要以下几个成员：

>1. 条件编号：属性成员代替，方便其他类查找条件
>2. 构造函数：强制要求对属性赋值
>3. 判断函数：需要传进一个状态机参数，因为条件类本身不自带参数，需要状态机的参数进行条件判断

~~~c#
public abstract class Trigger
{
    public TriggerID triggerID{get; set;}//属性编号
    
    public Trigger()
    {
        Init();
    }
    
    public abstract void Init();//构造函数，用于对属性赋值
    
    public abstract bool HandleTrigger(Base fsmBase);
    //判断函数，可以让别的类调用，并返回一个bool值
}
~~~



### 角色状态(State)

​	状态类需要以下几个成员：

>1. 状态编号：用属性代替，方便查找对应的条件
>2. 条件集合：由于保存各种条件对象
>3. 字典集合：用于查询条件成立时对应的状态
>4. 构造函数：强制要求对属性赋值
>5. 配置字典函数：向字典集合中添加成员
>6. 判断函数：由状态机调用，判断此时有何条件成立
>7. 三个基本函数：进入状态、状态执行、退出状态

~~~c#
public abstract class State
    {

        private Dictionary<TriggerID,StateID> map;//字典

        public List<Trigger> Triggers;//条件集合

        public StateID stateID { get; set; }//状态属性

        public State()
        {
            map = new Dictionary<TriggerID, StateID>();
            
            Init();
        }

        public abstract void Init();//抽象化构造函数

        public void Reason(Base fsmBase)//由 状态机 调用的 检测方法
        {
            for(int i=0;i<Triggers.Count;i++)//遍历条件集合
            {
                if(Triggers[i].HandleTrigger(fsmBase))
                {
                    //切换状态
                    fsmBase.ChangeActiveState(map[Triggers[i].triggerID]);
                    //将条件对应的状态ID传引用给状态机
                    return;
                }
            }
        }

        public void AddMap(TriggerID triggerID,StateID stateID)
        {
            map.Add(triggerID,stateID);
            //创建条件对象

            Type type = Type.GetType("FSM."+triggerID+"Trigger");
            Trigger trigger = Activator.CreateInstance(type) as Trigger;
            Triggers.Add(trigger);
        }

        public virtual void EnterState(Base fsmBase){}
        public virtual void ActionState(Base fsmBase){}
        public virtual void ExitState(Base fsmBase){}
    }
~~~



### 状态机(Base)

​	状态机所需成员：

>1. 默认状态ID：在Unity界面选择
>2. 状态列表：存储所有状态对象
>3. 初始化组件函数：初始化对应组件
>4. 初始化默认状态函数：根据默认状态ID设置默认状态
>5. 配置状态机函数：向条件列表与条件类中的字典集合中添加对象
>6. 切换状态函数：由状态类调用，条件满足后切换状态

~~~c#
public class Base : MonoBehaviour
    {
        
        [Header("角色基本组件")]
        [HideInInspector]public Animator Anim;
        [HideInInspector]public Rigidbody2D rb;
        
        [Tooltip("初始状态")]public StateID defaultStateID;
        private List<State> states;//状态列表
        public State currentState;//当前状态
        public State defaultState;

        public virtual void Start() 
        {
            InitCommponent();
            SetBase();
            InitDefaultState();
        }

        public void InitCommponent()//初始化组件
        {
            Anim = GetComponentInChildren<Animator>();
            rb = GetComponentInChildren<Rigidbody2D>();
        }


        public void InitDefaultState()//初始化默认状态
        {
            defaultState = states.Find(s => s.stateID == defaultStateID);
            currentState = defaultState;
            currentState.EnterState(this);
        }

        //配置状态机
        private void SetBase()
        {
            states = new List<State>();
            // //--创建状态对象
            // IdleState idleState = new IdleState();
            // //--添加映射
            // idleState.AddMap(TriggerID.NoHealth,StateID.Dead);
            // //--加入状态机
            // states.Add(idleState);

            // DeadState deadState = new DeadState();
            // states.Add(deadState);
        }

        //每帧处理的逻辑
        public virtual void Update() 
        {
            //判断当前状态条件
            currentState.Reason(this);
            //执行当前状态逻辑
            currentState.ActionState(this);
        }

        public void ChangeActiveState(StateID stateID)//切换状态
        {
            //离开上一个状态
            currentState.ExitState(this);
            //切换状态
            currentState = stateID == StateID.Default?defaultState:states.Find(s => s.stateID == stateID);
            //进入下一个状态
            currentState.EnterState(this);
        }
    }
~~~

