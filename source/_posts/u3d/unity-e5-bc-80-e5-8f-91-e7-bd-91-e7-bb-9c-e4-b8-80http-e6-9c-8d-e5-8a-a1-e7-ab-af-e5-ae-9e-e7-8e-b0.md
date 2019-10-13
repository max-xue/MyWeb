---
title: Unity开发之网络一HTTP服务端实现
categories:
  - U3D
url: 1359.html
id: 1359
comments: true
date: 2018-06-22 21:10:01
tags:
---

在前一篇介绍Http客户端的实现方式：[Unity开发网络一HTTP客户端实现](http://www.le-more.com/?p=1357)，下面介绍服务端 1.使用VS创建Web项目 2.添加Handler.ashx Handler.ashx.cs实现

/// <summary>
/// Summary description for Handler
/// </summary>
public class Handler : IHttpHandler
{
    //常量
    //应用配置信息
    public const string COMMAND\_APP\_INFO = "app_info";
    //服务器地址列表
    public const string COMMAND\_SERVER\_LIST = "server_list";

    #region 属性
    //协议标识
    private string _cmd;
    //应用Id
    private string _appId;
    //设备Id
    private string _deviceId;
    //客户端版本
    private string _appVer;

    public bool IsReusable
    {
        get
        {
            return false;
        }
    }

    #endregion

    public void ProcessRequest(HttpContext context)
    {
        if (DNTRequest.IsPost())
        {
            string data = DNTRequest.GetFormString("data");
            if (data.Length == 0) return;

            //解析数据
            Dictionary<string, string> dict = JsonConvert.DeserializeObject<Dictionary<string, string>>(data);
            if (dict.Count == 0) return;
            if (!dict.TryGetValue("cmd", out _cmd)) return;
            if (!dict.TryGetValue("app\_id", out \_appId)) return;
            if (!dict.TryGetValue("device\_id", out \_deviceId)) return;
            if (!dict.TryGetValue("app\_ver", out \_appVer)) return;

            switch (_cmd)
            {
                case COMMAND\_SERVER\_LIST:
                    _serverList();
                    break;
                default:
                    break;
            }
        }
        else
        {
            _write(true, "");
        }
    }

    private void _serverList()
    {
        string jsonString = "{";

        //temp
        if (_appVer == "2.0.250")
        {
            jsonString += "\\"ip\\":\\"192.168.20.131\\",";
            jsonString += "\\"port\\":9020,";
        }
        else
        {
            jsonString += "\\"ip\\":\\"192.168.20.131\\",";
            jsonString += "\\"port\\":9010,";
        }

        jsonString = jsonString.TrimEnd(',');
        jsonString += "}";

        _write(true, jsonString);
    }

    #region 内部函数
    private void _write(bool result, string data)
    {
        string jsonString = "{";
        jsonString += "\\"" + "result" + "\\":\\"" + string.Format("{0}", result) + "\\",";
        jsonString += "\\"" + "time" + "\\":\\"" + string.Format("{0}", DateTime.Now.Millisecond) + "\\",";
        jsonString += "\\"" + "data" + "\\":" + data + ",";
        jsonString = jsonString.TrimEnd(',');
        jsonString += "}";

        HttpContext.Current.Response.ContentType = "text/json";
        HttpContext.Current.Response.Cache.SetNoStore();
        HttpContext.Current.Response.Write(jsonString);
        HttpContext.Current.Response.End();
    }
    #endregion
}

说明： 1.ProcessRequest接收到客户端请求； 2.处理请求分POST和其他方式，暂时只处理POST； 3.读取数据使用DNTRequest.GetFormString("data") 从Form中读取； 4.客户端的ContentType 必须设置为"application/x-www-form-urlencoded"才能读取到，否则需要读取输入流； 5.返回的数据用Json格式。