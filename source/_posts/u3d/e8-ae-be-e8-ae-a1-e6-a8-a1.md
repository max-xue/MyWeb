---
title: Unity开发之设计模式---单例模式
categories:
  - U3D
url: 1184.html
id: 1184
comments: true
date: 2018-05-18 21:00:48
tags:
  - u3d
  - pattern
---

曾记录一篇关于设计模式的日志：[《Head First 设计模式》------学习总结](http://www.le-more.com/?p=92),在那篇文章中只是记录一些重要的概念，缺乏实战，也不好理解。下面记录在项目中使用到的设计模式，理论和实际相结合才有价值，“把模式装进脑子里，然后在你的设计和已有的应用中，寻找何处可以使用它们。”。 单例模式是最简单也是用的最多一种模式，概念简单就不再详细说明了 C#中定义单例模式
    
    public class DataManager
    {
            #region Singleton
            protected static DataManager m_Instance;
            public static DataManager Instance
            {
                get
                {
                    if (m_Instance == null)
                    {
                        m_Instance = new DataManager();
                    }
                    return m_Instance;
                }
            }
            #endregion
    
        public void OnLoad(){}
    }
    
    //使用示例
    DataManager.Instance.OnLoad();
    
    模板定义及示例
    
    /// <summary>
    /// 单例模板类
    /// </summary>
    /// <typeparam name="T">类型</typeparam>
    public class Singleton<T> where T : class, new()
    {
        //
        // Static Fields
        //
        protected static T m_Instance;
    
        //
        // Static Properties
        //
        public static T Instance
        {
            get
            {
                if (m_Instance == null)
                {
                    m_Instance = new T();
                }
                return m_Instance;
            }
        }
    
        //
        // Static Methods
        //
        public static T GetInstance()
        {
            return Instance;
        }
    }
    
    public class DataManager :Singleton<DataManager>
    {
         public void OnLoad(){}
    }
    
    //使用示例 DataManager.Instance.OnLoad();
    
    Unity中挂在组件上的单例
    
    public class AppUser : MonoBehaviour
    {
            #region Sington and MonoBehaviour
            private static AppUser instance;
            public static AppUser Instance
            {
                get
                {
                    if (instance) {
                        return instance;
                    }
                    else {
                        instance = GameObject.FindObjectOfType<AppUser>();
                    }
    
                    if (instance)
                    {
                        return instance;
                    }
                    else {
                        GameObject obj = new GameObject(typeof(AppUser).ToString());
                        //场景不消除
                        GameObject.DontDestroyOnLoad(obj);
                        instance = obj.AddComponent<AppUser>();
                    }
    
                    return instance;
                }
            }
    
            void Awake()
            {
                instance = this;
            }
    
            private void Update()
            {
                
            }
            #endregion
    
        public void OnStart(){}
    }
    
    //使用示例,场景控制脚本中的Start
    void Start()
    {
        AppUser.Instance.OnStart();
    }

在场景控制脚本中调用,会创建一个空对象挂载该脚本。并在场景切换后不销毁。 在客户端开发中基本不用考虑线程安全的问题。同时在模块很多的情况下，为了调用方便，单例类使用会很多。