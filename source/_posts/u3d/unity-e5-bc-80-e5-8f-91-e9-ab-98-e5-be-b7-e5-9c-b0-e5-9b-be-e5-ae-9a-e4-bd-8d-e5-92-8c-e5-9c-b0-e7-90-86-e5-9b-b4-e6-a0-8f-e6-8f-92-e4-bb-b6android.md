---
title: Unity开发之插件---高德地图定位和地理围栏(Android)
categories:
  - U3D
url: 1171.html
id: 1171
comments: true
date: 2018-01-03 15:47:14
tags:
---

Unity3d是一款3D游戏开发引擎，也可以开发2D游戏。可一键式发布到多种平台，可发布到iOS,Android,Windows,macOS等。其原理就是在底层帮开发者根据不同平台做了处理。但这并不是万能的，有时候我们需要自己来开发Unity和平台API交互的程序，即Unity插件。下面总结开发iOS和Android插件。 Android制作Unity插件请参考：[在 Unity 中使用 Android SDK](http://www.tuicool.com/articles/qmmeemR) ，简单有效。下面说一下集成高德定位的部分。 下载高德定位sdk，集成的版本是AMap\_Location\_V3.5.0_20170731.jar,将下载好的文件放置在libs目录 ![](http://www.le-more.com/wp-content/uploads/2018/01/u3d_plugin_001.png) 参考高德提供的功能示例，编写AMapHelper.java类,主要功能代码： 启动定位：

public void startLocation(String apiKey){
		AMapLocationClient.setApiKey(apiKey);
		
		//初始化client
		locationClient = new AMapLocationClient(Utils.getContext());
		locationOption = getDefaultOption();
		//设置定位参数
		locationClient.setLocationOption(locationOption);
		// 设置定位监听 	
		locationClient.setLocationListener(locationListener);
		
		// 启动定位
		locationClient.startLocation();
	}

停止定位：

public void stopLocation(){
		// 停止定位
		locationClient.stopLocation();
		
		showLog("停止定位成功!");
	}

添加围栏及停用：

public void startGeoFence(String jsonString) throws Exception{
		initGeoFence();
		addFence(jsonString);
	}
	
	public void stopGeoFence(){
		if (null != fenceClient) {
			fenceClient.removeGeoFence();
		}
		
		try {
			Utils.getContext().unregisterReceiver(mGeoFenceReceiver);
		} 
		catch (Throwable e) {
		}
	}

回调Unity代码：

private void SendMessage(int what, String msg){
    	switch(what){
    	case 0:
    		UnityPlayer.UnitySendMessage("AMapHelper","onLocationChanged",msg); 
    		break;
    	case 1:
    		UnityPlayer.UnitySendMessage("AMapHelper","onFenceChanged",msg); 
    		break;
    	}
    }

Unity中同样定义AMapHelper类用来调用Java类,关键代码：

#elif UNITY_ANDROID
        private AndroidJavaClass jcu;
        private AndroidJavaObject jou;
        private AndroidJavaObject amapHelper;

          /// <summary>
        /// 开始定位
        /// </summary>
        public void StartLocation(string apiKey)
        {
            error = false;
            errorInfo = "";

            try
            {
                if(jcu == null){
        
                    jcu = new AndroidJavaClass("com.unity3d.player.UnityPlayer");
                    jou = jcu.GetStatic<AndroidJavaObject>("currentActivity");
                    amapHelper = new AndroidJavaObject("com.shuimu.plugin.AMapHelper");
                }

                amapHelper.Call("startLocation",apiKey);
            }
            catch (System.Exception ex)
            {
                UnityLibs.Debuger.Log(ex.Message);
                error = true;
                errorInfo = ex.Message;
            }
        }

        public void StopLocation()
        {
            amapHelper.Call("stopLocation");
        }


        public void StartGeoFence(string jsonString) {
            amapHelper.Call("startGeoFence", jsonString);
        }

        public void StopGeoFence() {
            amapHelper.Call("stopGeoFence");
        }

         void OnDestroy()
        {
            if(amapHelper != null)
                amapHelper.Call("deactivate");
        }

#elif UNITY_IOS

定位及围栏回调：

 //围栏状态更新
        void onFenceChanged(string msg) {
            int status = int.Parse(msg);
            fenceStatus = (GeoFence)status;

            if (fenceChanged != null)
            {
                fenceChanged();
            }
        }

        //定位更新
        void onLocationChanged(string msg) {
            string\[\] msgs = msg.Split(';');
            if (msgs\[0\] == "0")
            {
                locationInfo.Latitude = double.Parse(msgs\[1\]);
                locationInfo.Longitude = double.Parse(msgs\[2\]);
            }
            else {
                error = false;
                errorInfo = "定位失败";
            }

            if (locationChanged != null)
            {
                locationChanged();
            }
        }
    }

使用方法： 1.在场景中AMapHelper附加到一空对象 2.初始化及设置回调：

  amapHelper.locationChanged += locationChanged;
  amapHelper.StartLocation(GlobalDef.AMap_ApiKey);

下一篇介绍iOS插件。