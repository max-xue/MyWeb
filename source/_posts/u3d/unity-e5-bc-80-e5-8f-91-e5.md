---
title: Unity开发之工具---GPS数据采集
categories:
  - U3D
url: 1374.html
id: 1374
comments: true
date: 2018-07-04 21:06:47
tags:
  - u3d
  - network
---

工具系列： [**Unity开发之导表（Excel）工具的制作分析**](http://www.le-more.com/?p=1154) 工欲善其事，必先利其器。程序开发离不开各种工具的制作。下面介绍获取GPS数据工具的制作。 制作这个工具的前提，必须先制作好Unity定位插件的开发，关于定位插件的开发介绍，传送门： [**Unity开发高德地图定位和地理围栏插件(iOS)**](http://www.le-more.com/?p=1176) [**Unity开发高德地图定位和地理围栏插件(Android)**](http://www.le-more.com/?p=1171) 先看一下效果图： ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_gps_0.png) 从界面看还是比较简陋的，但能完成目标的工具就是好工具。 首先开启定位
    
    //打开定位
    public void OnStartLocation( ) {
        amapHelper.StartLocation(GlobalDef.AMap_ApiKey);
    }
    
    //关闭定位
    public void OnStopLocation() {
        amapHelper.StopLocation();
    }
    
    //定位回调
    void locationChanged() {
        UnityLibs.Debuger.Log("locationChanged" + amapHelper.locationInfo.Longitude.ToString() + " " + amapHelper.locationInfo.Latitude);
        lat = amapHelper.locationInfo.Latitude;
        lng = amapHelper.locationInfo.Longitude;
        txtLongitude.text = amapHelper.locationInfo.Longitude.ToString();
        txtLatitude.text = amapHelper.locationInfo.Latitude.ToString();
    }
    
    void OnDestroy()
    {
        //移除监听
        amapHelper.locationChanged -= locationChanged;
    }

在成功开启定位后，走到要添加的位置，点击添加
    
    //添加
    public void OnAdd() {
        var item = Instantiate(itemPrefab);
    
        double distance = 0;
        if (itemList.Count > 0) {
            ItemBehaviour last = itemList\[itemList.Count - 1\].GetComponent<ItemBehaviour>();
            distance = Utils.GetDistance(last.lat, last.lng,lat,lng);
        }
        item.GetComponent<ItemBehaviour>().OnInit(lat, lng, distance);
    
        itemList.Add(item);
        item.transform.parent = content;
        content.sizeDelta = new Vector2(0, itemList.Count * 140);
    
        SaveFile();
    }
    
    //删除
    public void OnDelete() {
        if (itemList.Count > 0)
        {
            var last = itemList\[itemList.Count - 1\];
            Destroy(last);
            itemList.Remove(last);
        }
    
        content.sizeDelta = new Vector2(0, itemList.Count * 140);
    
        SaveFile();
    }
    
    //保存到文件
    void SaveFile()
    {
        string strLoc = "";
        foreach (var item in itemList) {
            strLoc += item.GetComponent<ItemBehaviour>().lat + "," + item.GetComponent<ItemBehaviour>().lng;
            strLoc += ";";
        }
    
        strLoc = strLoc.Remove(strLoc.Length - 1);
    
        string destPath = Path.Combine(FileUtils.GetWritePath(), "Location.txt");
        if (File.Exists(destPath)) File.Delete(destPath);
    
        StreamWriter sw;
        FileInfo t = new FileInfo(destPath);
        if (!t.Exists)
        {
            sw = t.CreateText();
        }
        else
        {
            sw = t.AppendText();
        }
    
        sw.WriteLine(strLoc);
        sw.Close();
        sw.Dispose();
    }

添加的同时会从将所有记录的GPS坐标点写入到Location.txt文件中，类似如下的数据：

31.1875618489583, 121.436771104601;31.1880425347222, 121.436774359809;31.1878998480903, 121.435769042969;31.1877416992188, 121.43583062066

当记录完所有位置的坐标后，从手机中取出Location.txt，加载到实际的场景中： ![](http://www.le-more.com/wp-content/uploads/2018/07/tool_gps_2.png) 红色区域就是从经纬度转为二维坐标，然后使用Gizmos描绘出的记录的GPS坐标，和现场的场景是可以对应的上的。