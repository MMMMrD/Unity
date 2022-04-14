# Unity-GameManager



## 为什么需要GameManager

​	在游戏过程中，某些条件为真时需要进行对应的操作，例如：玩家死后结束游戏、玩家进入另一个场景时保存游戏数据、获取某些道具后获得奖励......等等。

​	若是这些对应的条件相对较少时，我们可以将他们写进对应的 **角色/敌人** 中进行相应的判断，例如将游戏结束的判断写进玩家脚本中。但是，当这些条件变多时，脚本代码的整洁性与封装性就会被破坏。因此，我们需要创建一个额外的 **GameManager** 脚本，来管理我们的游戏，

​	

## 创建思路&代码实现

​	为了进行简化，下面就只介绍如何实现 角色死亡结束游戏、退出游戏、重新游玩、杀死所有敌人进入下一关、保存数据 这一些简单的代码。



**1.单例模式**

​	首先，为了保证我们每一次调用 GameManager 时都是同一个对象，保证在控制游戏时不发生错误，我们需要使用 **单例模式** 创建 GameManager ：

>1. 在 GameManager 脚本内创建一个 **静态的（static ）自身对象**
>2. 在 Awake 函数中进行判断，如果静态对象 == null ，则将自身赋值给静态对象；如果静态对象中已经有值，则将自身的 gameObject 删除

~~~c#
public static GameManager instance;//创建静态对象

public void Awake()
{
    if(instance == null)
        instance = this;
    else
        Destroy(gameObject);

    //.......
}
~~~



**2.控制游戏结束**

​	我们需要实现的是，在玩家死亡时弹出游戏结束的窗口，并让玩家选择退出或是重新开始。

​	为实现上述功能，我们需要时刻关注角色的生命值，当其为零时使游戏结束；同时，我们也需要创建一个在角色死亡时弹出的窗口，在窗口中有一个退出按钮和一个重新开始游戏按钮。

​	在玩家脚本 PlayerController 中拥有我们需要的判断死亡的布尔值，我们只需要在 GameManeger 中创建一个对应的布尔值 gameOver ，并实时将后者与前者同步即可。

​	游戏结束窗口控制的代码放在UI统一的 **UIManager** 中，我们只需要创建好UI即可。

~~~c#
//创建布尔值和 PlayerController 对象，方便监视其中的参数
class GameManager
{
    public bool gameOver;
    private PlayerController player;

    //实时更新布尔值，判断合适游戏结束
    public void Update()
    {
        gameOver = player.isDead;
        UIManager.instance.GameOverUI(gameOver);
    }
}
~~~

![image-20220414202650598](C:\Users\liaoz\AppData\Roaming\Typora\typora-user-images\image-20220414202650598.png)



**3.退出游戏、重新游玩**

​	字面意思，我们只需要实现相应的函数，并在需要的时候调用函数即可。

​	需要注意的是，在重新开始的函数中，我们需要引用对应的命名空间

~~~c#
using UnityEngine.SceneManagement;//引用相应的命名空间

class GameManager
{
    //重新开始函数
    public void RestartScene()
    {
     /*SceneManager.LoadScene()：在括号中填入对应场景的名字，可进入对应的场景
     
     SceneManager.GetActiveScene()：可获得对应场景的信息，这里获得的是场景的名字*/
        
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        
        //清楚对应数据
        PlayerPrefs.DeleteKey("playerHealth");
    }

    //退出函数
    public void Quit()
    {
        Application.Quit();
    }
}
~~~

![image-20220414205123135](C:\Users\liaoz\AppData\Roaming\Typora\typora-user-images\image-20220414205123135.png)



**4.所有敌人死亡开启下一关**

​	根据描述我们很容易想到，我们需要监视当前场景中的敌人数量，数量为零时开启下一关。但是我们事先不知道这个关卡中会放置多少个敌人，那么我们应该怎么对敌人的数量进行监视呢？

​	对此，我们需要在 **GameManager** 中创建一个存储 **Enemy** 对象的集合，并创建 添加与删除集合对象 的函数，让外部函数访问。

​	当敌人（Enemy）对象被创建是，需要将他自己添加进 **GameManager** 中对应的集合，死亡时将其删除。当集合为零时，证明敌人已经被清理干净，可以进入下一关。

​	这样，我们就需要实时检测集合中对象的数量即可。

~~~c#
class GameManager
{
    //创建敌人集合
	public List<Enemy> enemies = new List<Enemy>();
    
    //添加对象的函数
    public void IsEnemy(Enemy enemy)
    {
        enemies.Add(enemy);
    }

    //删除对象的函数
    public void EnemyDead(Enemy enemy)
    {
        enemies.Remove(enemy);

        //当集合数量为零
        if(enemies.Count == 0)
        {
            //开启下一关通道
            doorExit.OpenDoor();
            //保存数据
            SaveData();
        }
    }
}
~~~

![image-20220414210613292](C:\Users\liaoz\AppData\Roaming\Typora\typora-user-images\image-20220414210613292.png)



**5.保存数据（PlayerPrefs）**

​	相信看到这里，你已经看到几次对数据进行处理的代码了，在此做详细介绍。

​	在游戏中，数据的保存是相当重要的一部分。游戏需要做到保证，玩家在切换场景时，上一个场景的数据得到保留，在这里拿生命值举例。

​	假设我们需要保存角色的生命值到下一个场景，需要怎么做？

​	当然，我们可以像单例模式那样，创建一个 static 变量保存。在这里我们举出另一个方法。

​	在进入一个场景时，我们需要更新角色的生命值；同时，在退出时，我们也需要保存角色的生命值。Unity自身已经给我们提供了一个专门用于保存数据的对象 **PlayerPrefs** ，其中也提供了各种函数，我们只需要调用它写好的函数即可。

~~~c#
class GameManager
{
    //进入场景时重置生命值
    public float LoadHealth()
    {
        //如果没有对应的词条
        if(!PlayerPrefs.HasKey("playerHealth"))
        {
            //重新创建一个词条
            PlayerPrefs.SetFloat("playerHealth",3f);
        }

        //将对应词条中的值返回
        float currentHealth = PlayerPrefs.GetFloat("playerHealth");
        return currentHealth;
    }

    //保存词条函数
    public void SaveData()
    {
        PlayerPrefs.SetFloat("playerHealth",player.health);
        PlayerPrefs.Save();
    }
}
~~~



## 实例代码

​	将上述例子结合到一起，生成如下代码

~~~c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class GameManager : MonoBehaviour
{
    public bool gameOver;
    private PlayerController player;
    public Door doorExit;
    public List<Enemy> enemies = new List<Enemy>();

    public static GameManager instance;
    public void Awake()
    {
        if(instance == null)
            instance = this;
        else
            Destroy(gameObject);

        player = FindObjectOfType<PlayerController>();
        doorExit = FindObjectOfType<Door>();
    }

    public void Update()
    {
        gameOver = player.isDead;
        UIManager.instance.GameOverUI(gameOver);
    }

    public void IsEnemy(Enemy enemy)
    {
        enemies.Add(enemy);
    }

    public void EnemyDead(Enemy enemy)
    {
        enemies.Remove(enemy);

        if(enemies.Count == 0)
        {
            doorExit.OpenDoor();
            SaveData();
        }
    }

    public void RestartScene()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
        PlayerPrefs.DeleteKey("playerHealth");
    }

    public void NextLevel()
    {
        SceneManager.LoadScene(SceneManager.GetActiveScene().buildIndex + 1);
    }

    public float LoadHealth()
    {
        if(!PlayerPrefs.HasKey("playerHealth"))
        {
            PlayerPrefs.SetFloat("playerHealth",3f);
        }

        float currentHealth = PlayerPrefs.GetFloat("playerHealth");

        return currentHealth;
    }

    public void SaveData()
    {
        PlayerPrefs.SetFloat("playerHealth",player.health);
        PlayerPrefs.Save();
    }

    public void Quit()
    {
        Application.Quit();
    }
}

~~~

