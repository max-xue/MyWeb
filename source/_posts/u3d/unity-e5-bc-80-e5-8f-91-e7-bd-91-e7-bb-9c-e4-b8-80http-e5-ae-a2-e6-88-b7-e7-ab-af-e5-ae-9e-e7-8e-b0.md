---
title: Unity开发之网络一HTTP客户端实现
categories:
  - U3D
url: 1357.html
id: 1357
comments: true
date: 2018-06-22 20:51:32
tags:
---

客户端实现参考：[Unity发送HTTP请求和文件下载](https://www.jianshu.com/p/47e19d5a73db) 注意：仅通过WebRequest实现POST方式，通过测试 Http请求主要涉及两个类：HttpRequest，HttpResponse，在参考的实现基础上有改动 HttpRequest

/// <summary>
/// Http请求
/// </summary>
public class HttpRequest
{
    /// <summary>
    /// 错误代码
    /// </summary>
    public const int ERR_EXCEPTION = -1;

    #region 属性
    private string _url;
    private int _timeout;
    private Action<HttpResponse> _callback;
    private HttpWebRequest _request;
    private string _method;
    private string _contentType;
    private KeyValuePair<string, int> _proxy;
    protected int _range = -1;
    // post内容
    private StringBuilder _postBuilder;
    #region get/set
    /// <summary>
    /// 设置ContentType
    /// </summary>
    /// <value>ContentType value</value>
    public string ContentType
    {
        set
        {
            _contentType = value;
        }
    }
    #endregion

    #endregion

    /// <summary>
    /// 构造函数, 构造GET请求
    /// </summary>
    /// <param name="url">url地址</param>
    /// <param name="timeout">超时时间</param>
    /// <param name="callback">回调函数</param>
    public HttpRequest(string url, string method, int timeout, Action<HttpResponse> callback)
    {
        _url = url;
        _timeout = timeout;
        _callback = callback;
        _method = method.ToUpper();
    }
    /// <summary>
    /// 设置Post内容
    /// </summary>
    /// <param name="data">内容</param>
    public void SetPostData(string data)
    {
        if (_postBuilder == null)
        {
            _postBuilder = new StringBuilder(data.Length);
        }
        if (_postBuilder.Length > 0)
        {
            _postBuilder.Append("&");
        }
        _postBuilder.Append(data);
    }
    /// <summary>
    /// 添加Post内容
    /// </summary>
    /// <param name="key">key值</param>
    /// <param name="value">value值</param>
    public void AddPostData(string key, string value)
    {
        if (_postBuilder == null)
        {
            _postBuilder = new StringBuilder();
        }
        if (_postBuilder.Length > 0)
        {
            _postBuilder.Append("&");
        }

        _postBuilder.Append(key).Append("=").Append(UrlEncode(value));
    }
    /// <summary>
    /// 设置代理
    /// </summary>
    /// <param name="ip">ip地址</param>
    /// <param name="port">端口号</param>
    public void SetProxy(string ip, int port)
    {
        _proxy = new KeyValuePair<string, int>(ip, port);
    }

    /// <summary>
    /// 发动请求
    /// </summary>
    public void Start()
    {
        //Debug.Log("Handle Http Request Start");
        \_request = WebRequest.Create(\_url) as HttpWebRequest;
        \_request.Timeout = \_timeout;
        \_request.Method = \_method;  
        _request.Accept = "text/plain";

        if (_proxy.Key != null)
        {
            \_request.Proxy = new WebProxy(\_proxy.Key, _proxy.Value);
        }

        if (_contentType != null)
        {
            \_request.ContentType = \_contentType;
        }

        if (_range != -1)
        {
            \_request.AddRange(\_range);
        }

        try
        {
            if (_method.Equals("POST"))
            {
                WritePostData();
            }

            AsyncCallback callback = new AsyncCallback(OnResponse);
            _request.BeginGetResponse(callback, null);
        }
        catch (Exception e)
        {
            CallBack(ERR_EXCEPTION, e.Message);
            if (_request != null)
            {
                _request.Abort();
            }
        }
    }
    /// <summary>
    /// 处理读取Response
    /// </summary>
    /// <param name="result">异步回到result</param>
    protected void OnResponse(IAsyncResult result)
    {
        //Debug.Log ("Handle Http Response");
        HttpWebResponse response = null;
        try
        {
            // 获取Response
            response = _request.EndGetResponse(result) as HttpWebResponse;
            if (response.StatusCode == HttpStatusCode.OK)
            {
                if ("HEAD".Equals(_method))
                {
                    // HEAD请求
                    long contentLength = response.ContentLength;
                    CallBack((int)response.StatusCode, contentLength + "");
                    return;
                }

                // 读取请求内容
                string content = string.Empty;
                Stream stream = response.GetResponseStream();
                StreamReader streamReader = new StreamReader(stream, System.Text.Encoding.UTF8);
                content = streamReader.ReadToEnd();

                streamReader.Close();
                response.Close();
                _request.Abort();

                // 调用回调
                CallBack((int)response.StatusCode, content);
            }
            else
            {
                CallBack((int)response.StatusCode, "");
            }
        }
        catch (Exception e)
        {
            CallBack(ERR_EXCEPTION, e.Message);
            if (_request != null)
            {
                _request.Abort();
            }
            if (response != null)
            {
                response.Close();
            }
        }
    }

    private void CallBack(int code, string content)
    {
        if (_callback != null)
        {
            HttpResponse response = new HttpResponse();
            response.StatusCode = code;
            if (code == (int)HttpStatusCode.OK)
            {
                response.Content = content;
            }
            else
            {
                response.Error = content;
            }
                
            _callback(response);
        }
    }

    /// <summary>
    /// 写Post内容
    /// </summary>
    private void WritePostData()
    {
        if (null == \_postBuilder || \_postBuilder.Length <= 0)
        {
            return;
        }

        byte\[\] bytes = Encoding.UTF8.GetBytes(_postBuilder.ToString());
        _request.ContentLength = bytes.Length;
        _request.GetRequestStream().Write(bytes, 0, bytes.Length);
    }

    /// <summary>
    /// URLEncode
    /// </summary>
    /// <returns>encode value</returns>
    /// <param name="value">要encode的值</param>
    private string UrlEncode(string value)
    {
        StringBuilder sb = new StringBuilder();
        byte\[\] byStr = System.Text.Encoding.UTF8.GetBytes(value);
        for (int i = 0; i < byStr.Length; i++)
        {
            sb.Append(@"%" + Convert.ToString(byStr\[i\], 16));
        }
        return (sb.ToString());
    }
}

HttpResponse

/// <summary>
/// HTTP返回内容
/// </summary>
public class HttpResponse
{
    #region 属性
    // 状态码
    private int _statusCode;
    // 响应字节
    private string _content;
    // 错误内容
    private string _error;

    #region get/set

    /// <summary>
    /// 获取状态码
    /// </summary>
    /// <value>状态码</value>
    public int StatusCode
    {
        set
        {
            this._statusCode = value;
        }
        get
        {
            return this._statusCode;
        }
    }
    /// <summary>
    /// 获取错误消息
    /// </summary>
    /// <value>错误消息</value>
    public string Error
    {
        set
        {
            this._error = value;
        }
        get
        {
            return this._error;
        }
    }

    public string Content
    {
        get
        {
            return _content;
        }

        set
        {
            _content = value;
        }
    }
    #endregion

    #endregion

    /// <summary>
    /// 默认构造函数
    /// </summary>
    public HttpResponse()
    {

    }
}

在两个类的基础上，封装HttpHelper方便应用中使用

public class HttpHelper : Singleton<HttpHelper>
{
    //常量
    //应用配置信息
    public const string COMMAND\_APP\_INFO = "app_info";
    //服务器地址列表
    public const string COMMAND\_SERVER\_LIST = "server_list";

    #region 属性
    //基本数据
    private Dictionary<string, string> _baseData;
    private Dictionary<Action<bool, object>, HttpResponse> _responseDict;

    #region get/set
    public Dictionary<string, string> BaseData
    {
        get
        {
            return _baseData;
        }

        set
        {
            _baseData = value;
        }
    }
    #endregion

    #endregion

    public HttpHelper()
    {
        _responseDict = new Dictionary<Action<bool, object>, HttpResponse>();
        _baseData = new Dictionary<string, string>();

        \_baseData.Add("app\_id", Global.GlobalDef.AppId);
        \_baseData.Add("app\_ver", Global.GlobalDef.AppVer);
        \_baseData.Add("device\_id", Global.GlobalDef.DeviceId);
    }

    //消息分发:主线程
    public void Update(float dt)
    {
        if (_responseDict.Count > 0)
        {
            foreach (var item in _responseDict)
            {
                Action<bool, object> callback = item.Key;
                HttpResponse response = item.Value;
                try
                {
                    bool result = false;
                    string content = response.Error;
                    if (response.StatusCode == (int)System.Net.HttpStatusCode.OK)
                    {
                        result = true;
                        content = response.Content;
                        if (callback != null)
                        {
                            JsonData jsonResult = JsonMapper.ToObject(content);
                            callback(result, jsonResult\["data"\]);
                        }
                    }
                    else
                    {
                        callback(result, content);
                    }
                }
                catch (Exception e)
                {
                    UnityLibs.Debuger.Log(e.ToString());
                }
            }

            _responseDict.Clear();
        }
    } 

    #region 应用信息
    public void AppInfo(Action<bool,object> callback)
    {
        LitJson.JsonData jsonData = new LitJson.JsonData();
        jsonData\["cmd"\] = COMMAND\_APP\_INFO;
        foreach (var item in _baseData)
        {
            jsonData\[item.Key\] = item.Value;
        }

        string jsonStr = LitJson.JsonMapper.ToJson(jsonData);
        string url = Global.GlobalDef.HostAddr;
        _onRequest(url, jsonStr, delegate (HttpResponse response)
        {
            _responseDict.Add(callback, response);
        });
    }

    #endregion

    #region 服务器列表
    public void ServerList(Action<bool, object> callback)
    {
        LitJson.JsonData jsonData = new LitJson.JsonData();
        jsonData\["cmd"\] = COMMAND\_SERVER\_LIST;
        foreach (var item in _baseData)
        {
            jsonData\[item.Key\] = item.Value;
        }

        string jsonStr = LitJson.JsonMapper.ToJson(jsonData);
        string url = Global.GlobalDef.HostAddr;
        _onRequest(url, jsonStr, delegate (HttpResponse response)
        {
            _responseDict.Add(callback, response);
        });
    }

    #endregion

    #region 内部函数

    private void _onRequest(string url, string data, Action<HttpResponse> callback)
    {
        HttpRequest client = new HttpRequest(url, "POST", 30 * 1000, callback);
        client.ContentType = "application/x-www-form-urlencoded";
        client.AddPostData("data", data);
        client.Start();
    }

    #endregion
}

调用示例:

HttpHelper.Instance.ServerList(delegate(bool result,object obj) 
{
    if (result)
    {
        LitJson.JsonData jsonData = obj as LitJson.JsonData;
        string serverAddr = jsonData\["ip"\].ToString();
        int port = int.Parse(jsonData\["port"\].ToString());
        HandlerObservable.Instance.OnInit(serverAddr, (short)port);
    }
    else
    {
        HandlerObservable.Instance.OnInit(GlobalDef.SERVER\_ADDR, GlobalDef.SERVER\_PORT); 
    }

    //Logo展示完后加载
    StartCoroutine(\_startLoading(\_loadingTime));
});

说明： 1.ContentType 设置为application/x-www-form-urlencoded，在服务实现部分会说明 2.数据使用Json格式编码; 3.Http响应回调是在单独的线程中，为了调用方式需要将回调在主线程完成（Update的作用）。 4.HttpHelper的Update方法需要在游戏中不会被销毁的继承MonoBehaviour的类每帧调用：

private void Update()
{
    HttpHelper.Instance.Update(UnityEngine.Time.deltaTime);
}

下一篇介绍服务端实现