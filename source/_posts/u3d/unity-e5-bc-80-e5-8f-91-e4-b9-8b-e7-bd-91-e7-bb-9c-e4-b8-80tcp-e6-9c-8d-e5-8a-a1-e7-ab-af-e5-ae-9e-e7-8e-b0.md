---
title: Unity开发之网络一TCP服务端实现
categories:
  - U3D
url: 1366.html
id: 1366
comments: true
date: 2018-07-03 21:40:05
tags:
---

[Unity开发之网络---集成Protobuf](http://www.le-more.com/?p=1100) [Unity开发之网络一HTTP客户端实现](http://www.le-more.com/?p=1357) [Unity开发之网络一HTTP服务端实现](http://www.le-more.com/?p=1359) [Unity开发之网络一TCP客户端实现](http://www.le-more.com/?p=1364) 下面介绍服务端，共涉及三个类：SocketModule，SocketEngine，ServerSocketItem SocketModule：同客户端，定义接口及数据包结构等定义； SocketEngine：服务端接收客户端连接等处理； ServerSocketItem：单个客户端数据交互的处理； 下面主要介绍SocketEngine 开启服务

public bool StartServer(string ip, string port)
{
    m_bService = true;

    // 创建负责监听的套接字
    m_hServerSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
    m_hServerSocket.SetSocketOption(SocketOptionLevel.Socket, SocketOptionName.ReuseAddress, true);

    IPAddress address = IPAddress.Parse(ip);
    // 创建包含ip和端口号的网络节点对象；
    IPEndPoint endPoint = new IPEndPoint(address, int.Parse(port));
    try
    {
        //将负责监听的套接字绑定到唯一的ip和端口上；
        m_hServerSocket.Bind(endPoint);
    }
    catch (SocketException se)
    {
        onShowMsg("异常：" + se.Message);
        return false;
    }

    // 设置监听队列的长度；
    m_hServerSocket.Listen(10);
    // 创建负责监听的线程；
    m_SocketAcceptThread = new Thread(ListenSocket);
    m_SocketAcceptThread.IsBackground = true;
    m_SocketAcceptThread.Start();

    //检测线程
    m_dwTickCount = 0;
    m_SocketDetectThread = new Thread(DetectSocket);
    m_SocketDetectThread.IsBackground = true;
    m_SocketDetectThread.Start();

    onShowMsg("服务启动成功！");
    return true;
}

服务启动成功后，会启动监听线程，接收客户端的连接

/// <summary>
/// 监听客户端请求的方法；
/// </summary>
void ListenSocket()
{
    while (IsService())
    {
        try
        {
            // 开始监听客户端连接请求，Accept方法会阻断当前的线程
            Socket connect = m_hServerSocket.Accept();

            if (connect.Connected && !m_SocketRSThread.ContainsKey(connect.RemoteEndPoint.ToString()))
            {
                //创建接收对象
                ServerSocketItem socketItem = new ServerSocketItem((short)m_StorageSocketItem.Count, this);
                socketItem.Attach(connect);

                // 将与客户端连接的 套接字 对象添加到集合中
                m_StorageSocketItem.Add(socketItem);
                onSocketConnect(true, socketItem.GetClientAddr());
                if (m_pIServerSocketItemSink != null)
                {
                    m_pIServerSocketItemSink.OnSocketAcceptEvent(socketItem);
                }

                Thread thread = new Thread(RecvSocket);
                thread.IsBackground = true;
                thread.Start(socketItem);
                m_SocketRSThread.Add(connect.RemoteEndPoint.ToString(), thread);
            }
        }
        catch(Exception e)
        {
            Console.WriteLine(this.GetType().ToString() + ":" + e.ToString());
        }
    }
}

void RecvSocket(object item)
{
    ServerSocketItem socketItem = item as ServerSocketItem;
    while (IsService() && socketItem.IsValidSocket())
    {
        try
        {
            //length = socketClient.Receive(arrMsgRec); // 接收数据，并返回数据的长度；
            socketItem.RecvData();
        }
        catch (SocketException se)
        {
            onShowMsg("异常：" + se.Message);
            RemoveSocketItem(socketItem);

            break;
        }
        catch (Exception e)
        {
            onShowMsg("异常：" + e.Message);
            RemoveSocketItem(socketItem);

            break;
        }
    }
}

接收到客户端的连接请求后，启动一个读写线程，创建ServerSocketItem变量与之对应 心跳检测

void DetectSocket()
{
    while (IsService())
    {
        //设置间隔
        Thread.Sleep(500);
        m_dwTickCount += 500L;

        if (m\_dwTickCount > SocketModule.TIME\_DETECT\_SOCKET && m\_StorageSocketItem != null)
        {
            m_dwTickCount = 0;

            for (int i = 0; i < m_StorageSocketItem.Count; ++i) 
            {
                var socketItem = m_StorageSocketItem\[i\];
                if (socketItem != null)
                {
                    CMD\_KN\_DetectSocket detectSocket = new CMD\_KN\_DetectSocket();
                    detectSocket.dwSendTickCount = 0;// int.Parse(ZYSoft.Utils.Time.GetTimeStamp(DateTime.Now));
                    byte\[\] cbDataBuffer = detectSocket.Encode();

                    socketItem.SendData(cbDataBuffer, (short)cbDataBuffer.Length, SocketModule.MDM\_KN\_COMMAND, SocketModule.SUB\_KN\_DETECT_SOCKET, socketItem.GetRountID());
                }
            }
        }
    }
}

网络引擎和应用层交互通过IServerSocketItemDelegate接口

//连接对象回调接口
public interface IServerSocketItemDelegate
{
    //应答消息
    bool OnSocketAcceptEvent(ServerSocketItem serverSocketItem);
    //读取消息
    bool OnSocketReadEvent(CMD_Command Command, byte\[\] pBuffer, short wDataSize, ServerSocketItem serverSocketItem);
    //关闭消息
    bool OnSocketCloseEvent(ServerSocketItem serverSocketItem);
};

示例

public class AttemperEngine : IServerSocketItemDelegate
    {
.....................................
 public bool OnSocketAcceptEvent(ServerSocketItem serverSocketItem)
        {
            return true;  
        }

        public bool OnSocketReadEvent(CMD_Command Command, byte\[\] pBuffer, short wDataSize, ServerSocketItem serverSocketItem)
        {
            return false;
        }

        public bool OnSocketCloseEvent(ServerSocketItem serverSocketItem)
        {

            return false;
        }
}

ServerSocketItem处理与客户端数据的接收与发送 接收

//接收操作
public void RecvData()
{
    //效验变量
    //Debug.Assert(m_bRecvIng == false);
    if(m_hSocket == null) return;

    //判断关闭
    if (m_bCloseIng == true)
    {
        CloseSocket(m_wRountID);
        return;
    }

    //设置变量
    m_bRecvIng = true;
    m_dwRecvTickCount = DateTime.Now.Millisecond / 1000;

    //判断关闭
    if (m_hSocket == null)
    {
        CloseSocket(m_wRountID);
    }

    //接收数据
    int iRetCode = m\_hSocket.Receive(m\_cbRecvBuf, m\_wRecvSize, (int)(m\_cbRecvBuf.GetLength(0) - m_wRecvSize), SocketFlags.None);
    if (iRetCode <= 0)
    {
        CloseSocket(m_wRountID);
    }
    //接收完成
    m_wRecvSize += iRetCode;
    //byte\[\] cbBuffer = new byte\[SocketModule.SOCKET_BUFFER\];
    CMD\_Head pHead = new CMD\_Head();
    //pHead.Decode(m_cbRecvBuf);

    //处理数据
    while (m\_wRecvSize >= CMD\_Head.sizeof\_CMD\_Head)
    {
        pHead.Decode(m_cbRecvBuf);
        //效验数据
        short wPacketSize = pHead.cmdInfo.wDataSize;
        if (wPacketSize > SocketModule.SOCKET_BUFFER) break;// throw ("数据包超长");
        if (wPacketSize < CMD\_Head.sizeof\_CMD_Head) break;// throw ("数据包非法");
        if (pHead.cmdInfo.cbMessageVer != SocketModule.SOCKET_VER) break;// throw ("数据包版本错误");
        if (m_wRecvSize < wPacketSize) break;

        byte\[\] cbBuffer = new byte\[wPacketSize\];
        //提取数据
        Buffer.BlockCopy(m_cbRecvBuf, 0, cbBuffer, 0, wPacketSize);
        short wRealySize = CrevasseBuffer(cbBuffer, wPacketSize);
        m_dwRecvPacketCount++;
        //删除缓存数据
        m_wRecvSize -= wPacketSize;
        Buffer.BlockCopy(m\_cbRecvBuf, wPacketSize, m\_cbRecvBuf, 0, m_wRecvSize);

        //解释数据
        short wDataSize = (short)(wRealySize - CMD\_Head.sizeof\_CMD_Head);
        pHead = new CMD_Head();
        pHead.Decode(cbBuffer);
        //m_strRecvSession = Encoding.Default.GetString(pHead.cbSession);
        CMD_Command Command = pHead.command;

        //内核命令
        if (Command.wMainCmdID == SocketModule.MDM\_KN\_COMMAND)
        {
            switch (Command.wSubCmdID)
            {
                case SocketModule.SUB\_KN\_DETECT_SOCKET:  //网络检测
                    {
                        break;
                    }
                default: break;// throw ("非法命令码");
            }
        }
        else
        {
            //数据消息处理
            Buffer.BlockCopy(cbBuffer, CMD\_Head.sizeof\_CMD_Head, cbBuffer, 0, wDataSize);
            m_pIServerSocketItemSink.OnSocketReadEvent(Command, cbBuffer, wDataSize, this);
        }
    }
}

发送

//发送函数
public bool SendData(byte\[\] pData, short wDataSize, short wMainCmdID, short wSubCmdID, short wRountID) {
    //效验参数
    //Debug.Assert(wDataSize <= SocketModule.SOCKET_PACKAGE);

    //效验状态
    if (m_bCloseIng == true) return false;
    if (m_wRountID != wRountID) return false;
    if (m_dwRecvPacketCount == 0) return false;
    if (IsValidSocket() == false) return false;
    if (wDataSize > SocketModule.SOCKET_PACKAGE) return false;

    //构造数据
    byte\[\] cbDataBuffer = new byte\[SocketModule.SOCKET_BUFFER\];

    CMD\_Head pHead = new CMD\_Head();
    pHead.command.wMainCmdID = wMainCmdID;
    pHead.command.wSubCmdID = wSubCmdID;

    Array.Copy(pHead.Encode(), cbDataBuffer, CMD\_Head.sizeof\_CMD_Head);
    if (wDataSize > 0)
    {
        Debug.Assert(pData != null);
        Array.Copy(pData, 0, cbDataBuffer, CMD\_Head.sizeof\_CMD_Head, wDataSize);
    }

    //加密数据
    short wSendSize = EncryptBuffer(cbDataBuffer, (short)(CMD\_Head.sizeof\_CMD_Head + wDataSize), (short)cbDataBuffer.GetLength(0));

    //发送数据
    return SendRawData(cbDataBuffer, wSendSize,wRountID);
}

//发送函数
public bool SendRawData(byte\[\] pData, short wDataSize, short wRountID) {
    bool ret = true;
    try
    {
        IAsyncResult asyncSend = m\_hSocket.BeginSend(pData, 0, wDataSize, SocketFlags.None, new AsyncCallback(sendCallback), m\_hSocket);
        bool success = asyncSend.AsyncWaitHandle.WaitOne(5000, true);
        if (!success)
        {
            CloseSocket(wRountID);
            ret = false;
            Console.WriteLine("Failed to SendMessage server.");
        }
    }
    catch
    {
        ret = false;
        Console.WriteLine("send message error");
    }

    return ret;
}

private void sendCallback(IAsyncResult async)
{
    try
    {
        //结束挂起的异步发送
        SocketError errorCode;
        Socket socket = (Socket)async.AsyncState;
        int dwSendSize = socket.EndSend(async, out errorCode);

        Console.WriteLine("send size :" + dwSendSize.ToString());
    }
    catch (Exception e)
    {
        Console.WriteLine("error :" + e.ToString());
    }
    finally
    {

    }
}

ServerSocketItem和应用层的交互也是通过IServerSocketItemDelegate 一个ServerSocketItem对应一个GameUser实例 界面长这个样子,^^ ![](http://www.le-more.com/wp-content/uploads/2018/06/net_socket_0.png)