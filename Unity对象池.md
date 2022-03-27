# Unity对象池

## 单例模式

创建一个对象池，首先要设置单例模式

~~~
	public static shadowPool instance;

	void Start()
	{
		instance = this;
	}
~~~



