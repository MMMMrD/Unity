# Unity-随手记



## 数值记录

​	可以直接创建一个继承于 **ScriptableObject** 的脚本，在里面写入我们需要记录的数值，这样就可以通过创建数值文件，直接在Unity窗口中对数据进行赋值，同时也方便我们对数值进行修改

​	下图是数值文件的代码，我们还需要创建一个脚本，用于访问数值文件中的各个数值。（M-studio 3D-RPG）

~~~c#
[CreateAssetMenu(fileName = "New Data",menuName = "Character Stats/Data")]
public class CharacterData_SO : ScriptableObject
{
    [Header("Stats Info")]
    public int maxHealth;
    public int currentHealth;
    public int baseDefence;
    public int currentDefence;
}
~~~

![image-20220424100945601](C:\Users\liaoz\AppData\Roaming\Typora\typora-user-images\image-20220424100945601.png)



## 防止人物在受伤时移动

​	可以在人物受伤动画中添加一个行为，修改人物动画的状态机：在进入和执行动画的时候停止移动，在退出动画的时候开始移动

​	下面拿3DRPG Demo举例：

![image-20220424120443228](C:\Users\liaoz\AppData\Roaming\Typora\typora-user-images\image-20220424120443228.png)

