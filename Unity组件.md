# Unity组件



## Collider

>​    Collider的基本作用就是阻止一个物体进入另一个物体。如果你没有给墙壁添加Collider，那么你就能穿过一堵墙。当然在Unity中，当你Create一个3D Object的时候，那个3D Object自动带有Collider。
>
>​    Collider还分为动态与静态两大类：
>
>动态：
>
>>位置会改变，需要多次计算位置
>
>静态：
>
>> 位置不改变，只需要计算一次位置



## Rigibody

> 来自 Rigibody over 的翻译：
>
> ​    Rigibody时对于一个游戏对象激活其物理行为的主要组件。通过添加刚体，对象能立即响应重力效应。
>
> ​    如果添加一个或多个碰撞组件，那么这个游戏对象就会通过碰撞输入来移动。当游戏对象添加上RIgibody组件，你就不能通过改变transform的position和rotation的脚本来移动游戏对象，应该使用force去推动游戏对象让物理引擎去计算。



## Animator

>用于控制Mecanim动画系统的接口。
>
>Unity手册地址：
>
>[UnityEngine.Animator - Unity 脚本 API](https://docs.unity.cn/cn/current/ScriptReference/Animator.html)



## Transform

>可在Unity界面右侧的Transform列表中查看/修改GameObject的数据
