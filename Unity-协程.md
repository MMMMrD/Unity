# Unity-协程

​	一般的程序执行都是线性的，也就是必须一行一行的执行代码。

​	使用Unity提供的协程，就可以类似于开辟另一条线程，调整根据你所写的代码，调整下一行代码执行的时间。

~~~c#
IEnumerator MoveToAttackTarget()
    {
        agent.isStopped = false;

        transform.LookAt(attackTarget.transform);

        //TODO:后期修改攻击距离参数
        while(Vector3.Distance(attackTarget.transform.position,transform.position)>2)
        {
            agent.destination = attackTarget.transform.position;
            yield return null;
        }

        agent.isStopped = true;
        
        //Attack
        if(LastAttackTime<0)
        {
            anim.SetTrigger("Attack");

            LastAttackTime = 0.5f;
        }
    }
~~~



​	

