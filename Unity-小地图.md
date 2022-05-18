# Unity-小地图



## 整体构思

>1. 首先需要我们在窗口中**创建一块画布（Canvas）** 用于小地图的渲染
>1. 需要一个 **Image 预制体**，在敌人进入地图范围时可以**通过预制体生成对应的图标**。
>1. 通过**代码控制**各个图标在小地图上的移动。
>

​	***（这里代码控制有很多种实现方法，下面只讲例子，不限制实现方式***



## 实现细节



### 创建画布（Canvas）

​	需要在 层级(Hierarchy) 窗口中创建 **挂载小地图的画布(Canvas)**。



### 初始化图标

>1. 首先我们需要选择合适的素材
>2. 将其**纹理类型设置为Sprite**
>3. 将其拖拽进 **Resources** 文件夹作为预制体，供后续使用



### 代码逻辑及编写

​	首先梳理一下代码需要实现的功能：

> 1. 需要控制**图标的生成**、**图标的显示与消失**
> 2. 需要**判断人物与目标之间的距离**，并**根据比例实时更新图标位置**



​	细化一下如何讲上述功能实现：

>1. 需要一个控制 UI 的 **UIManager**，当小地图存在时，就向 UIManager 注册，当以后**需要访问小地图时，就需要通过 UIManager 进行访问**
>
>   
>
>2. 图标生成、显示与消失需要代码实现，所以代码需要包含以下几点：
>
>   >1. 需要一个**创建 Image（CreateImage）** 的函数，通过外部调用**创建对应图标** 
>   >2. 需要**ShowPlayer函数**以及**ShowEnemy函数**，根据外部传进的**Image和距离参数**，**完成图标的设置**，同时**按比例更新其在小地图之中的位置**
>
>   
>
>3. 判断人物与目标之间距离，需要在 **敌人（Enemy）** 的对应脚本中完成
>
>   >​	调用场景中的小地图脚本，**通过 CreateImage 创建图片**，判断**自身与角色x轴与y轴的距离**，并将**图片与距离数据作为参数**，调用场景内小地图中的 **ShowEnemy 函数**，更新敌人图标的位置



### 代码实例

​	以下是小地图的代码，更加细小的问题已经通过注释的形式阐述

***（关键需要看代码的第 57 行，有关于代码写法的解释***

~~~c#
public class MiniMap : MonoBehaviour
{
    private static Image item; //图片预制体，后续会根据该预制体生成图标
    private RectTransform rect; 
    private Transform player;  
    private Image playerImage; //Player的图标

    private void Awake()
    {
        UIManager.Instance.MiniMap = this;  //向UIManager注册自身，可供其他类通过调用UIManager，完成对当前场景中的小地图的操作
    }
    
    private void Start()
    {
        item = Resources.Load<Image>("Image");//在Resources文件夹中查找图片预制体，并将其赋值给item
        rect = GetComponent<RectTransform>();
        player = GameManager.Instance.playerStats.transform;//通过GameManager获得Player的Transform

        if(player != null)
            playerImage = Instantiate(item);  //根据预制体创建新的Image，并赋值给PlayerImage
        
    }

    private void Update() 
    {
        ShowPlayer();    
    }

    //展示Player图标
    void ShowPlayer()
    {
        if(playerImage != null)//若playerImage不为空
        {

            playerImage.rectTransform.sizeDelta = new Vector2(20,20);//通过rectTransform设置图片大小
            playerImage.rectTransform.anchoredPosition = new Vector2(0,0);//设置图片在Canvas上的坐标位置
            playerImage.rectTransform.eulerAngles = new Vector3(0,0,-player.eulerAngles.y);//设置图片的旋转
            playerImage.sprite = Resources.Load<Sprite>("Texture/Player");//修改Image的图片素材
            playerImage.transform.SetParent(transform, false);//设置为小地图的子物体
        }
    }

    //外部调用函数，展示Enemy图标
    public void ShowEnemy(Image image, float disX, float disY)
    {
        //步骤详情参考 ShowPlayer 函数
        image.rectTransform.sizeDelta = new Vector2(20,20);
        //算法：distance/10*100 获得距离在小地图上的缩放
        image.rectTransform.anchoredPosition = new Vector2(disX * 10, disY * 10);
        image.sprite = Resources.Load<Sprite>("Texture/Enemy");
        image.transform.SetParent(transform, false);
    }

    //外部调用函数，创建Image物体
    public Image CreateImage()
    {
        /*这里不可以直接写在ShowEnemy中，因为需要实时更新坐标的位置，所以需要在Update中调用该函数，
        如果将创建Image对象语句写入 ShowEnemy 中，则会每帧都创建一个Image对象，导致严重的资源浪费。
        将创建Image语句写成单独的函数，在 Enemy 类中调用并将新创建的 Image 对象传引用给 ShowEnemy 函数，
        即避免了资源的浪费，也可以保证每个Enemy个体可以单独控制自身的图标，可谓是一举两得*/
        return Instantiate(item);
    }
}
~~~

