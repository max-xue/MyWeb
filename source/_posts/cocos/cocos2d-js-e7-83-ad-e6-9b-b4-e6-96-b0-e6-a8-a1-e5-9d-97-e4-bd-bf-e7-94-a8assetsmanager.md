---
title: cocos2d-js 热更新模块 使用AssetsManager
categories:
- Cocos
url: 44.html
id: 44
date: 2016-09-01 18:14:35
tags:
---

情况说明： 1.满足Web和Native一套工程：为了维护方便 2.检测更新每天只执行一次：每次打开会等待,一天打开多次会卡多次。这个改动只适合自己的 AssetsManager 类：

var __failCount = 0;

toFix = function(num, length) {
return ('' + num).length < length ? ((new Array(length + 1)).join('0') + num).slice(-length) : '' + num;
};

var AssetsManagerLoaderScene = cc.Scene.extend({
_am:null,
_progress:null,
_percent:0,
_percentByFile:0,
_progressText : "",
run:function(){
if (!cc.sys.isNative){
this.loadGame();
return;
}
else{
var myDate = new Date();
var year = myDate.getFullYear();
var month = toFix(myDate.getMonth() + 1,2);
var day = toFix(myDate.getDate(),2);
var nowDate = year + "" + month + "" + day;
var lastDate = cc.sys.localStorage.getItem("kAssetsDate");

//一天只检测一次
if(parseInt(nowDate) > parseInt(lastDate)
|| lastDate == null
|| lastDate == undefined
|| lastDate == ""){
cc.sys.localStorage.setItem("kAssetsDate",nowDate);
}
else{
this.loadGame();
return;
}
}

var layer = new cc.Layer();
this.addChild(layer);
this._progressText = "升级中 ";
if(cc.sys.language == cc.sys.LANGUAGE_ENGLISH){
this._progressText = "UPDATING ";
}

this.\_progress = new cc.LabelTTF(this.\_progressText +"0.00%", "Arial", 40);
this._progress.x = cc.winSize.width / 2;
this._progress.y = cc.winSize.height / 2 + 50;
layer.addChild(this._progress);

// android: /data/data/com.huanle.magic/files/
var storagePath = (jsb.fileUtils ? jsb.fileUtils.getWritablePath() : "/");
cc.log("AssetsManagerLoaderScene storagePath="+storagePath);
this._am = new jsb.AssetsManager("res/project.manifest", storagePath);
this._am.retain();

if (!this._am.getLocalManifest().isLoaded())
{
cc.log("Fail to update assets, step skipped.");
this.loadGame();
}
else
{
// cc.log("packageUrl = "+);

var that = this;
var listener = new jsb.EventListenerAssetsManager(this._am, function(event) {
cc.log("listener................");
switch (event.getEventCode()){
case jsb.EventAssetsManager.ERROR\_NO\_LOCAL_MANIFEST:
cc.log("No local manifest file found, skip assets update.");
that.loadGame();
break;
case jsb.EventAssetsManager.UPDATE_PROGRESSION:
that._percent = event.getPercent();
that._percentByFile = event.getPercentByFile();
cc.log(that._percent + "%");

var msg = event.getMessage();
if (msg) {
cc.log(msg);
}
break;
case jsb.EventAssetsManager.ERROR\_DOWNLOAD\_MANIFEST:
case jsb.EventAssetsManager.ERROR\_PARSE\_MANIFEST:
cc.log("Fail to download manifest file, update skipped.");
that.loadGame();
break;
case jsb.EventAssetsManager.ALREADY\_UP\_TO_DATE:
case jsb.EventAssetsManager.UPDATE_FINISHED:
cc.log("Update finished.");
that.loadGame();
break;
case jsb.EventAssetsManager.UPDATE_FAILED:
cc.log("Update failed. " + event.getMessage());

__failCount ++;
if (__failCount < 5)
{
that._am.downloadFailedAssets();
}
else
{
cc.log("Reach maximum fail count, exit update process");
__failCount = 0;
that.loadGame();
}
break;
case jsb.EventAssetsManager.ERROR_UPDATING:
cc.log("Asset update error: " + event.getAssetId() + ", " + event.getMessage());
// that.loadGame();
break;
case jsb.EventAssetsManager.ERROR_DECOMPRESS:
cc.log(event.getMessage());
that.loadGame();
break;
default:
break;
}
});

cc.eventManager.addListener(listener, 1);
this._am.update();
cc.director.runScene(this);
}

this.schedule(this.updateProgress, 0.5);
},
loadGame:function(){
cc.log("loadGame");
if (cc.sys.isNative){
cc.loader.loadJs(\["src/files.js"\], function(err){
cc.loader.loadJs(jsFiles, function(err){
cc.LoaderScene.preload(g_resources, function () {
cc.director.runScene(new MainScene());
}, this);
});
});
}
else{
cc.LoaderScene.preload(g_resources, function () {
cc.director.runScene(new MainScene());
}, this);
}
},
updateProgress:function(dt){
this.\_progress.string = this.\_progressText + this.\_percent.toFixed(2)+"%";//"" + this.\_percent;
},
onExit:function(){
cc.log("AssetsManager::onExit");

this._am.release();
this._super();
}
});

QQ群：239759131 开发技术交流 欢迎您