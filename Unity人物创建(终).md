# Unity人物创建

## 新建角色

>1.在Hierarchy窗口中新建一个2D sprites，用于承载2D角色
>
>2.将角色拖拽到刚刚新建的2D sprites中，新角色生成

要实现碰撞，跳跃，奔跑的动画效果，就得用代码在player中实现

## 角色移动

>1.左右移动：
>
>> 角色左右移动，可以利用Unity输入设置中预先设好的按键返回值，例如输入设置中的"Horizontal”，当用户输入a/left时，其返回值为-1，当输入d/right时，其返回值为1，当用户不输入时，其返回值为0;
>
>2.跳跃
>
>> 跟角色移动一样，角色的跳跃在Unity也已被预先设置好。当用户按下space(空格)键时，输入预设中的“junm”值就会返回1，否则返回0；
>
>3.角色转向
>
>> 角色转向依旧可以利用“Horizontal”的返回值，判断角色的朝向，并改变该角色
>>
>>  transform 中的 localScale 的值，使其面朝指定方向
>>
>> ~~~
>> float facedirection = Input.GetAxisRaw("Horizontal");
>> if(facedirection != 0)
>> {
>> 	transform.localScale = new Vector3 (facedirection,1,1);
>> }
>> ~~~
>
>4.趴行
>
>>(1)简述：当用户按下s/down键时，将角色切换成爬行的动画，并将角色上方的2D碰撞器关闭，实现角色的爬行。
>>
>>(2)优化：为防止角色在爬行时重新回到站立状态而卡在地形中，需要进行判断。若角色的头顶有碰撞体，那么角色重新站立的代码将不被执行。代码如下：
>>
>>~~~
>>void Crouch()
>>{    			 
>>
>>	if(!Physics2D.OverlapCircle(cellingCheck.position,0.2f,ground))
>>        {
>>            if(Input.GetButton("Crouch"))
>>            {
>>                Anim.SetBool("crouching",true);
>>                disColl.enabled = false;
>>            }
>>            else
>>            {
>>                Anim.SetBool("crouching",false);
>>                disColl.enabled = true;
>>            }
>>        }
>>}
>>~~~



