# Unity-敌人（Enemy）



## 引言

​	敌人是每个游戏中不可缺少的部分，设计得好的敌人可以给游戏增添很多乐趣，设计得差的则会非常影响我们的游戏体验。

​	经过这段时间的学习，我们已经接触了非常多的敌人代码的写法，但是就是没有系统的归类，导致每次写敌人，都要从头开始。现在是时候将他们进行一个系统性的整理了。



## 逻辑思考



### 基本框架

​	我们首先来想一想，敌人需要执行什么操作，方便我们进行后面的构思。

​	首先，我们的敌人需要移动、攻击、受伤这些基本的操作，有些敌人还会跳跃。

​	为了实现敌人的移动，我们可能需要两个**固定的目标点**、一个发现目标时 **记录目标的点**，还可能需要 **转向** 的函数，还有 **切换目标** 的函数，以实现我们 **巡逻移动、或朝向目标移动**的目的。

​	为了实现敌人的攻击，我们需要 **动画器组件**，播放对应的攻击动画，同时还需要一些 **参数**，记录敌人自己的 **攻击距离、攻击速度**。因为我们使用了状态机，所以只需要写好攻击函数，使敌人进入攻击状态后每帧调用攻击函数就可以了。

​	为了实现敌人的受伤，我们需要创建一个 **接口**，里面包含一个抽象的受伤（GetHit）函数，需要子类实现，玩家的受伤函数也可以继承这个接口。



### 敌人需要的基本参数

​	根据上述构思的大概框架，我们可以简单想一下我们需要哪些参数

​	首先，敌人需要一些最基本的参数：

>1. 生命值（health）
>2. 移动速度（speed）
>3. 跳跃力（jumpForce）
>4. 死亡判定变量（isDead）
>5. 巡逻点A、B（pointA，pointB）
>6. 目标点（targetPoint）



​	其次，还需要一些特殊的参数，记录某些值或是执行某些代码

>1. 目标点 Transform 集合：记录一个范围之内的所有物体的 Transform
>2. 攻击参数：由几个参数组合而成，分别为攻击间隔（attackRate）、攻击时的时间点（nextAttack）、攻击距离（attackRange）
>3. 各种组件：动画器（Animator）、刚体（Rigidbody）
>4. 状态机及其子类对象



### 敌人需要的基本函数

​	构思完基本参数后，我们可以开始构思我们该如何写我们的函数了

​	首先还是基本的函数：

>1. **移动函数：**每帧都调用，让敌人朝向目标点移动，还需调用转向函数，保证敌人一直朝向目标
>2. **攻击函数：**由**状态机**调用，当目标进入敌人攻击范围后攻击
>3. **受伤函数：**实现接口，由攻击放调用，在角色受到攻击时，根据传进的数值变量减少相应的生命值。



​	其次，是根据为了辅助基本函数的执行，需要添加的函数

>1. **切换巡逻目的地函数：**由 **状态机** 调用，达到目标点后，转换敌人当前的巡逻目标点，需要在程序开始调用一次，为目标点赋值
>2. **转向函数：**在移动函数中调用，判断目标点在敌人的那一边，并将敌人的方向朝向目标点
>3. **有关于触发器(Trigger)的函数：**当指定目标进入敌人视野范围，则将目标添加进入目标点集合，离开则删除

​	

​	以上这些函数，有些是在敌人自身的类中调用，有的则需要在状态机中调用，我们也来简单理一下这其中的逻辑。

​	假设这个敌人只有简单的两个状态：巡逻和攻击，那么：

**1.巡逻状态：**

>​	敌人默认状态，需要在进入该状态就调用**切换巡逻目标点函数**，将初始目标点初始化。
>
>​	此外，还需要每帧都判断与目标点的距离，如果到达了目标点，则在此进入该状态，重置目标点。在该状态中也需要持续判断攻击目标集合是否为0，如果不为0则切换为攻击状态。



**2.攻击状态：**

>​	进入攻击状态，就要立刻将攻击目标集合中的第一个值赋值给目标点（重置目标点），敌人将持续朝向目标点移动。
>
>​	在该状态中，要持续判断攻击目标集合的个数，并执行相应的函数，包括但不限于切换状态、切换攻击目标........



### 其他

​	写完这些基本的参数与函数后，我们就需要将我们的有限状态机加入敌人的基类，详细的可以看有限状态机的笔记，这里不过多赘述。



## 最终代码

​	根据上述对敌人的构思，产生了如下所述的代码，在此已经做了关键注释

​	知识点较为细碎，在这里不做总结归纳，相对重要的为如下几点：

>1. 与外部物体、有限状态机配合进行的攻击函数：85行
>2. 新的转向代码：118行





~~~c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public abstract class Enemy : MonoBehaviour, IDamageable
{

    [Header("基本参数")]
    public float speed;//速度值
    public float jumpForce;//跳跃力
    public float health;//生命值
    public bool isDead;//死亡判定变量
    public Transform pointA, pointB;//巡逻点
    public Transform targetPoint;//目标点
    //目标点的Transform集合
    public List<Transform> attackList = new List<Transform>();

    [Header("Attack Setting")]
    //若当前时间点 > 攻击间隔 + 上一次攻击的时间点，那么就可以攻击
    public float attackRate;//攻击间隔（攻击速度）
    [HideInInspector]public float nextAttack;//在攻击时，记录此次攻击的时间点
    public float attackRange,skillRange;//普通攻击距离和特殊攻击距离

    [Header("基本组件")]
    private GameObject alarmSign; //遇到敌人标识
    [HideInInspector]public Animator anim;//动画器组件
    [HideInInspector]public Rigidbody2D rb;//刚体

    [Header("状态机相关")]
    EnemyBaseState currentState; //创建状态机对象
    [HideInInspector]public int animState;//控制动画状态的参数
    public PatrolState patrolState = new PatrolState();//实例化巡逻类对象
    public AttackState attackState = new AttackState();//实例化攻击类对象

    public void Awake()
    {
        InitComponent();//初始化组件
    }
    protected virtual void Start()
    {
        TransitionState(patrolState);//进入巡逻类，并初始化目标点
    }

    protected virtual void Update()
    {
        anim.SetBool("dead",isDead); //实时同步动画器中的bool值
        
        if(isDead)  //假如角色死亡
        {
            rb.velocity = Vector2.zero; //将速度设置为0
            return; //直接返回,不执行下列代码
        }

        currentState.OnUpdate(this);//各个状态中每一帧都需要执行的函数

        anim.SetInteger("state",animState);//设置动画机中的状态
    }

    public void TransitionState(EnemyBaseState state)//切换状态类的函数
    {
        currentState = state;
        currentState.EnterState(this);//进入状态需要执行的函数
    }

    public virtual void InitComponent()//初始化组件
    {
        anim = GetComponent<Animator>();
        rb = GetComponent<Rigidbody2D>();

        //获得第一个子类的GameObject组件
        alarmSign = transform.GetChild(0).gameObject;
    }

    public virtual void Movement()//移动函数
    {
        if(targetPoint != null)
        {
            transform.position = Vector2.MoveTowards(transform.position, targetPoint.position, speed*Time.deltaTime);
        }
        FilpDirection();//转向函数
    }

    protected virtual void Jump(){}//跳跃函数

    public virtual void AttackAction()//基础攻击函数
    {
       	//如果攻击距离大于与目标点的距离
        if(Vector2.Distance(targetPoint.position, transform.position) < attackRange)
        {
            //且攻击cd已经转好
            if(Time.time > attackRange + nextAttack)
            {
                //播放攻击动画
                anim.SetTrigger("attack");
                //对目标造成伤害的具体实现，将用外部的物体的方法执行

                //刷新cd
                nextAttack = Time.time + attackRange;
            }
        }
    }

    public virtual void SkillAttack()//特殊攻击函数
    {
        if(Vector2.Distance(targetPoint.position, transform.position) < skillRange)
        {
            if(Time.time > attackRange + nextAttack)
            {
                //播放攻击动画
                anim.SetTrigger("skill");

                //刷新cd
                nextAttack = Time.time + attackRange;
            }
        }
    }

    protected void FilpDirection()//转向函数
    {
       /*由于不同的素材原来的方向不同，我们单纯的使用改变 Transform 缩放的方法，会出现应素材不同而代码不同的问题，大大降低了代码的复用性
       为提高代码复用性，这里采用修改 Tramsform 旋转的方法，达到调整敌人朝向的问题*/
        
        if(transform.position.x < targetPoint.position.x)//当目标点在右侧
        {
            //将y方向的旋转改为0
            transform.rotation = Quaternion.Euler(0f, 0f, 0f);
        }
        else
        {
            //将y方向的旋转改为180
            transform.rotation = Quaternion.Euler(0f, 180f, 0f);
        }
    }

    public void SwitchPoint()//转换默认目标点
    {
        //将目标切换为最近的目标点
        if(Mathf.Abs(pointA.position.x - transform.position.x) > Mathf.Abs(pointB.position.x - transform.position.x))
        {
            targetPoint = pointA;
        }
        else
        {
            targetPoint = pointB;
        }
    }

    //受伤函数（实现接口）
    public void GetHit(float damage) 
    {
        health -= damage;

        if(health < 1)
        {
            health = 0;
            isDead = true;
        }

        anim.SetTrigger("hit");
    }

    //以下函数均为碰撞体进入时会触发的函数
    public void OnTriggerStay2D(Collider2D collision) 
    {
        if(!attackList.Contains(collision.transform))
        {
            attackList.Add(collision.transform); 
        } 
    }

    public void OnTriggerExit2D(Collider2D collision) 
    {
        attackList.Remove(collision.transform);
    }
    
}
~~~

