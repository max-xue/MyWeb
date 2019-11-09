---
title: Unity开发之网络一TCP客户端实现
categories:
  - U3D
url: 1364.html
id: 1364
comments: true
date: 2018-07-03 21:05:41
tags:
  - u3d
  - network
---

网络开发系列 [Unity开发之网络---集成Protobuf](http://www.le-more.com/?p=1100) [Unity开发之网络一HTTP客户端实现](http://www.le-more.com/?p=1357) [Unity开发之网络一HTTP服务端实现](http://www.le-more.com/?p=1359) 下面介绍Sockt通信的实现，和Http相比Tcp/IP实现要复杂一些。不过使用C#语言开发要容易了很多。为什么这样说呢，因为以前写过使用C++实现Socket通信，在项目中已经编写完成，但因为外因，项目没有上线，最后日志也不了了之 [跨平台客户端Socket 一 数据包定义](http://www.le-more.com/?p=58) 下面介绍使用C#实现Socket通信，包体结构及定义从C++修改而来。 主要涉及三个类SocketModule，ClientSocket，ClientSocketHelper SocketModule：定义包头结构，ClientSocket的接收连接等接口，以及其他定义 ClientSocket：网络连接的具体实现 ClientSocketHelper：二次封装，方便应用层调用 ClientSocket主要代码如下： 网络连接
    
    public virtual bool Connect(string szServerIP, short wPort){
        //效验参数
        Debug.Assert(m\_SocketState == enSocketState.SocketState\_NoConnect);
        m\_SocketState = enSocketState.SocketState\_Connecting;
    
        IPAddress\[\] addressArray = Dns.GetHostAddresses(szServerIP);
        if (addressArray\[0\].AddressFamily == AddressFamily.InterNetworkV6)
        {
            //创建Socket对象， 连接类型是TCP  
            m_hSocket = new Socket(AddressFamily.InterNetworkV6, SocketType.Stream, ProtocolType.Tcp);
        }
        else
        {
            //创建Socket对象， 连接类型是TCP  
            m_hSocket = new Socket(AddressFamily.InterNetwork, SocketType.Stream, ProtocolType.Tcp);
        }
    
        //服务器IP地址  
        IPAddress ipAddress = IPAddress.Parse(addressArray\[0\].ToString());
        //服务器端口  
        IPEndPoint ipEndpoint = new IPEndPoint(ipAddress, wPort);
        //这是一个异步的建立连接，当连接建立成功时调用connectCallback方法  
        IAsyncResult result = m\_hSocket.BeginConnect(ipEndpoint, new AsyncCallback(connectCallback), m\_hSocket);
        //这里做一个超时的监测，当连接超过30秒还没成功表示超时  
        bool success = result.AsyncWaitHandle.WaitOne(30000, true);
        if (!success)
        {
            //超时  
            CloseSocket(false);
            UnityLibs.Debuger.Log("connect Time Out");
    
            m\_SocketState = enSocketState.SocketState\_NoConnect;
            if (m_pClientSocketDelegate != null)
            {
                m_pClientSocketDelegate.OnSocketConnect(-1, "连接超时", this);
            }
    
            return false;
        }
        else
        {
            return true;
        }
    }
    
    private void connectCallback(IAsyncResult async)
    {
        try
        {
            //结束挂起的异步连接
            Socket socket = (Socket)async.AsyncState;
            if (socket.Connected)
            {
                socket.EndConnect(async);
    
                //设置参数
                m_dwRecvSize = 0;
                m_cbSendRound = 0;
                m_cbRecvRound = 0;
                m_dwSendXorKey = 0x12345678;
                m_dwRecvXorKey = 0x12345678;
                m_dwSendTickCount = DateTime.Now.Millisecond / 1000;
                m_dwRecvTickCount = DateTime.Now.Millisecond / 1000;
    
                m\_SocketState = enSocketState.SocketState\_Connected;
                if(m_pClientSocketDelegate != null)
                {
                    m_pClientSocketDelegate.OnSocketConnect(0, "", this);
                }
                            
                //与m_hSocket建立连接成功，开启线程接受服务端数据。  
                Thread thread = new Thread(new ThreadStart(RecvSocket));
                thread.IsBackground = true;
                thread.Start();
    
                //开启心跳检测,毫秒为单位
                m\_timerDetectSocket = new Timer(timerHandler, m\_hSocket, 60 * 1000, 5 * 1000);
                UnityLibs.Debuger.Log("connectSuccess");
            }
            else
            {
                m\_SocketState = enSocketState.SocketState\_NoConnect;
                if (m_pClientSocketDelegate != null)
                {
                    m_pClientSocketDelegate.OnSocketConnect(-2, "连接出错", this);
                }
            }
        }
        catch (Exception e)
        {
            UnityLibs.Debuger.Log("error." + e);
            m\_SocketState = enSocketState.SocketState\_NoConnect;
            if (m_pClientSocketDelegate != null)
            {
                m_pClientSocketDelegate.OnSocketConnect(-2, "连接异常", this);
            }
        }
    }

需要注意的是连接时要检测是否是IPV6的网络，否则iOS无法通过审核。连接成功会开启接收线程和心跳包检测定时任务。 接收：
    
    private void RecvSocket()
    {
        //在这个线程中接受服务器返回的数据  
        while (m\_SocketState == enSocketState.SocketState\_Connected)
        {
            if (!m_hSocket.Connected)
            {
                //与服务器断开连接跳出循环  
                UnityLibs.Debuger.Log("Failed to clientSocket server.");
                CloseSocket(true);
                break;
            }
    
            try
            {
                if (m_hSocket.Available <= 0) continue;
    
                int iRetCode = m\_hSocket.Receive(m\_cbRecvBuf,m\_dwRecvSize,m\_cbRecvBuf.GetLength(0) - m_dwRecvSize, SocketFlags.None);
                //int iRetCode = m\_hSocket.Receive(m\_cbRecvBuf, m\_dwRecvSize, SocketModule.SOCKET\_BUFFER - m_dwRecvSize, SocketFlags.None);
                if (iRetCode <= 0)
                {
                    CloseSocket(true);
                    break;
                }
                else
                {
                    Debug.Assert(m_dwSendPacketCount > 0);
                    m_dwRecvSize += iRetCode;
                    m_dwRecvTickCount = DateTime.Now.Millisecond / 1000;
    
                    //变量定义
                    short wPacketSize = 0;
                    //byte \[\]cbDataBuffer = new byte\[SocketModule.SOCKET_BUFFER\];
                    CMD\_Head pHead = new CMD\_Head();
    
                    while (m\_dwRecvSize >= CMD\_Head.sizeof\_CMD\_Head)
                    {
                        //效验参数
                        pHead.Decode(m_cbRecvBuf);
                        wPacketSize = pHead.cmdInfo.wDataSize;
                        Debug.Assert(wPacketSize <= (SocketModule.SOCKET\_PACKAGE + CMD\_Head.sizeof\_CMD\_Head));
                        if (pHead.cmdInfo.cbMessageVer != SocketModule.SOCKET_VER) break;// throw ("数据包版本错误");
                        if (wPacketSize > (SocketModule.SOCKET\_PACKAGE + CMD\_Head.sizeof\_CMD\_Head)) break;// throw ("数据包太大");
                        if (m_dwRecvSize < wPacketSize) break;//throw ("数据包太小");
    
                        byte\[\] cbDataBuffer = new byte\[wPacketSize\];
                        //拷贝数据
                        m_dwRecvPacketCount++;
                        Array.Copy(m_cbRecvBuf,cbDataBuffer,wPacketSize);
                        //删除缓存数据
                        m_dwRecvSize -= wPacketSize;
                        Array.Copy(m\_cbRecvBuf, wPacketSize, m\_cbRecvBuf, 0, m_dwRecvSize);
    
                        //解密数据
                        short wRealySize = CrevasseBuffer(cbDataBuffer, wPacketSize);
                        Debug.Assert(wRealySize >= CMD\_Head.sizeof\_CMD_Head);
    
                        //解释数据
                        short wDataSize = (short)(wRealySize - CMD\_Head.sizeof\_CMD_Head);
                        CMD_Command Command = pHead.command;
                        Array.Copy(cbDataBuffer, CMD\_Head.sizeof\_CMD_Head, cbDataBuffer, 0, wDataSize);
    
                        //UnityLibs.Debuger.Log("RecvData :" + pHead.command.wMainCmdID
                        //  \+ " sub:" + pHead.command.wSubCmdID
                        //  \+ " dataSize:" + (pHead.cmdInfo.wDataSize - CMD\_Head.sizeof\_CMD_Head)
                        //  \+ " recvSize:" + pHead.cmdInfo.wDataSize
                        //  );
    
                        //内核命令
                        if (Command.wMainCmdID == SocketModule.MDM\_KN\_COMMAND)
                        {
                            switch (Command.wSubCmdID)
                            {
                                case SocketModule.SUB\_KN\_DETECT_SOCKET:  //网络检测
                                    {
                                        //发送数据
                                        SendData(SocketModule.MDM\_KN\_COMMAND, SocketModule.SUB\_KN\_DETECT_SOCKET, cbDataBuffer, wDataSize);
                                        break;
                                    }
                            }
                        }
                        else
                        {
                            bool bSuccess = false;
                            if (m_pClientSocketDelegate != null)
                            {
                                bSuccess = m_pClientSocketDelegate.OnSocketRead(Command, cbDataBuffer, wDataSize, this);
                            }
                                        
                            if (bSuccess == false)
                            {
                                foreach (IClientSocketRecvDelegate socketRecv in m_listSocketRecvDelegate)
                                {
                                    bSuccess = socketRecv.OnSocketRead(Command, cbDataBuffer, wDataSize, this);
                                    if (bSuccess)
                                    {
                                        break;
                                    }
                                }
    
                                if (bSuccess == false)
                                {
                                    UnityLibs.Debuger.Log("网络数据包处理失败");
                                }
                            }
                        }
                    }
                }
            }
            catch (Exception e)
            {
                UnityLibs.Debuger.Log("Failed to clientSocket error." + e);
                CloseSocket(true);
                break;
            }
        }
    }

接收线程主要是对网络传递的数据流解析为定义的数据包，要注意的是数据的连包和沾包处理。 数据发送
    
    public virtual bool SendData(short wMainCmdID, short wSubCmdID){
        //效验状态
        if (m_hSocket == null) return false;
        if (m\_SocketState != enSocketState.SocketState\_Connected) return false;
    
        //构造数据
        CMD\_Head pHead = new CMD\_Head();
        pHead.command.wMainCmdID = wMainCmdID;
        pHead.command.wSubCmdID = wSubCmdID;
        //if (AppUser.Instance.Session.Length == SocketModule.SOCKET\_PACKAGE\_SESSION)
        //    Array.Copy(Encoding.Default.GetBytes(AppUser.Instance.Session), pHead.cbSession, SocketModule.SOCKET\_PACKAGE\_SESSION);
    
        byte\[\] cbDataBuffer = new byte\[SocketModule.SOCKET_BUFFER\];
        Array.Copy(pHead.Encode(), cbDataBuffer, CMD\_Head.sizeof\_CMD_Head);
    
        //加密数据
        short wSendSize = EncryptBuffer(cbDataBuffer, CMD\_Head.sizeof\_CMD_Head, (short)cbDataBuffer.GetLength(0));
    
        //发送数据
        return SendBuffer(cbDataBuffer, wSendSize);
    }
    
    //发送函数
    public virtual bool SendData(short wMainCmdID, short wSubCmdID, byte\[\] pData, short wDataSize){
        //效验状态
        if (m_hSocket == null) return false;
        if (m\_SocketState != enSocketState.SocketState\_Connected) return false;
    
        //效验大小
        Debug.Assert(wDataSize <= SocketModule.SOCKET_PACKAGE);
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
            Array.Copy(pData,0, cbDataBuffer, CMD\_Head.sizeof\_CMD_Head,wDataSize);
        }
    
        //加密数据
        short wSendSize = EncryptBuffer(cbDataBuffer, (short)(CMD\_Head.sizeof\_CMD_Head + wDataSize), (short)cbDataBuffer.GetLength(0));
    
        //发送数据
        return SendBuffer(cbDataBuffer, wSendSize);
    }
    
    bool SendBuffer(byte\[\] pBuffer, int dwSendSize)
    {
        bool ret = true;
        try
        {
            IAsyncResult asyncSend = m\_hSocket.BeginSend(pBuffer, 0, dwSendSize, SocketFlags.None, new AsyncCallback(sendCallback), m\_hSocket);
            bool success = asyncSend.AsyncWaitHandle.WaitOne(5000, true);
            if (!success)
            {
                CloseSocket(true);
                ret = false;
                UnityLibs.Debuger.Log("Failed to SendMessage server.");
            }
        }
        catch
        {
            ret = false;
            UnityLibs.Debuger.Log("send message error");
        }
    
        return ret;
    }
    
    private void sendCallback(IAsyncResult async)
    {
        try
        {
            //结束挂起的异步发送
            SocketError errorCode;
            Socket client = (Socket)async.AsyncState;
            int dwSendSize = client.EndSend(async,out errorCode);
            if (dwSendSize == 0) {
                CloseSocket(true);
            }
    
            //UnityLibs.Debuger.Log("send size :" + dwSendSize.ToString());
        }
        catch (Exception e)
        {
            CloseSocket(true);
        }
    }

心跳检测
    
    //心跳包
    private void timerHandler(object state)
    {
        if (m\_SocketState == enSocketState.SocketState\_Connected)
        {
            SendData(SocketModule.MDM\_KN\_COMMAND, SocketModule.SUB\_KN\_DETECT_SOCKET);
            if (m_pClientSocketDelegate != null)
            {
                m_pClientSocketDelegate.OnSocketDetect(this);
            }
        }
    }

在ClientSocket的变量定义中有一个：
    
    //连接回调 
    protected IClientSocketDelegate m_pClientSocketDelegate;

通过这个变量将消息传递出去
    
     public class ClientSocketHelper : Singleton<ClientSocketHelper>, IClientSocketDelegate
    
    ClientSocketHelper 实现这个接口，并接收来自ClientSocket的消息
    
    public bool OnSocketConnect(int iErrorCode, string pszErrorDesc, IClientSocket pIClientSocket)
    {
        if (iErrorCode != 0)
        {
            --ConnectTimes;
            if (ConnectTimes > 0)
            {
                //再次连接
                ConnectServer();
                return false;
            }
            else
            {
                \_connectTimes = SocketModule.CONNECT\_MAX_TIMES;
            }
        }
    
        if (onSocketConnect != null)
        {
            return onSocketConnect(iErrorCode, pszErrorDesc);
        }
    
        return false;
    }
    
    public bool OnSocketRead(CMD_Command Command, byte\[\] pBuffer, short wDataSize, IClientSocket pIClientSocket)
    {
        if (onSocketRead != null)
        {
            return onSocketRead(Command, pBuffer, wDataSize);
        }
    
        return false;
    }
    
    public bool OnSocketClose(IClientSocket pIClientSocke, bool bCloseByServer)
    {
        if (onSocketClose != null)
        {
            return onSocketClose();
        }
    
        return false;
    }
    
    public bool OnSocketDetect(IClientSocket pIClientSocke)
    {
        if (IsScheduleClose)
        {
            _scheduleTime++;
            //UnityLibs.Debuger.Log("OnSocketDetect:" + _scheduleTime);
            if (\_scheduleTime * SocketModule.SOCKET\_DETECT >= SocketModule.CONNECT\_ACTIVE\_BACKGROUND)
            {
                CloseSocket();
            }
        }
    
        return true;
    }

应用层又是如果接收到最终的数据呢，通过代理的方式
    
    public delegate bool OnSocketConnectEvent(int iErrorCode, string pszErrorDesc);
    public delegate bool OnSocketReadEvent(CMD_Command Command, byte\[\] pBuffer, short wDataSize);
    public delegate bool OnSocketCloseEvent();
    public delegate bool OnSocketErrorEvent(object data);

示例
    
     public class HandlerObservable : Observable
        {
            public event OnSocketConnectEvent onSocketConnect;
            public event OnSocketCloseEvent onSocketClose;
            public event OnSocketErrorEvent onSocketError;
    .........................................................
    public void OnInit(string addr, short port)
            {
                if (_isInit == false)
                {
                    ClientSocketHelper.Instance.ServerAddr = addr;
                    ClientSocketHelper.Instance.ServerPort = port;
    
                    ClientSocketHelper.Instance.onSocketConnect += OnSocketConnect;
                    ClientSocketHelper.Instance.onSocketRead += OnSocketRead;
                    ClientSocketHelper.Instance.onSocketClose += OnSocketClose;
    
                    _isInit = true;
                }
            }

因为在消息的传递过程中，是在自定义的线程中（接收线程）处理的，需要将消息传送到主线程中，方便Unity的调用。和HTTP封装类类似，需要在Update中将消息分发到具体的处理类中。 具体参考HTTP客户端的实现。 因为涉及的内容比较多，就定这些吧，

使用Socket通信需要注意的有： 

1.网络连接重试，可能第一次未连接上，需要多试几次; 

2.断线重连处理，移动应用网络不稳定，网络断线重连对用户透明； 

3.应用退到后台，iOS会自动断开，Android不会，所以加逻辑断开； 

4.IPV6的支持，曾因这个被拒了两次。 

下一篇介绍服务端的实现