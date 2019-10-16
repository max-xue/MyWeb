---
title: quick-lua AssetsManager 热更新
categories:
- Cocos
url: 49.html
id: 49
date: 2016-09-01 18:21:11
tags:
---

网上关于热更新文章很多，我在官方例子及js的代码基础上做了移植 quick-lua版本 3.5 UpdateScene类

local UpdateScene = class("UpdateScene", cc.Scene)

function UpdateScene:ctor()
    
end

function UpdateScene:onExit()
    if self.am_ then
        self.am_:release()
        self.am_ = nil
    end
end

function UpdateScene:run()
    self.layer_ = cc.Layer:create()
    self:addChild(self.layer_)

    local winSize = cc.Director:getInstance():getWinSize()
    local lblProgress = cc.Label:createWithTTF("0%","fonts/fzy4.ttf",40)
    lblProgress:setPosition(cc.p(winSize.width/2, winSize.height/2 + 50))
    self.layer_:addChild(lblProgress)

    self.storagePath_ = cc.FileUtils:getInstance():getWritablePath() .. "/".."game_assets"
    self.am_ = cc.AssetsManagerEx:create("res/project.manifest", self.storagePath_)
    self.am_:retain()

    if self.am_:getLocalManifest():isLoaded() then
        local localManifest = self.am_:getLocalManifest()
        print(localManifest:getVersion())
    end

    local failCount = 0
    if not self.am_:getLocalManifest():isLoaded() then
        print("Fail to update assets, step skipped.")
        self:startGame()
    else
        local function onUpdateEvent(event)
            local eventCode = event:getEventCode()
            if eventCode == cc.EventAssetsManagerEx.EventCode.ERROR\_NO\_LOCAL_MANIFEST then
                print("No local manifest file found, skip assets update.")
                self:startGame()
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.NEW\_VERSION\_FOUND then

            elseif eventCode == cc.EventAssetsManagerEx.EventCode.UPDATE_PROGRESSION then
                local assetId = event:getAssetId()
                local percent = event:getPercent()
                local percentByFile = event:getPercentByFile()
                local strInfo = ""

                if assetId == cc.AssetsManagerExStatic.VERSION_ID then
                    strInfo = string.format("Version file: %d%%", percent)
                elseif assetId == cc.AssetsManagerExStatic.MANIFEST_ID then
                    strInfo = string.format("Manifest file: %d%%", percent)
                else
                    strInfo = string.format("%d%%", percent)
                end
                lblProgress:setString(strInfo)
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.ERROR\_DOWNLOAD\_MANIFEST or
                eventCode == cc.EventAssetsManagerEx.EventCode.ERROR\_PARSE\_MANIFEST then
                print("Fail to download manifest file, update skipped.")
                self:startGame()
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.ALREADY\_UP\_TO_DATE or
                eventCode == cc.EventAssetsManagerEx.EventCode.UPDATE_FINISHED then
                print("Update finished.")
                self:startGame()
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.UPDATE_FAILED then
                print("Update failed. ", event:getMessage())

                failCount = failCount + 1
                if (failCount < 5) then
                    that.am_:downloadFailedAssets()
                else
                    print("Reach maximum fail count, exit update process")
                    failCount = 0
                    self:startGame()
                end
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.ERROR_UPDATING then
                print("Asset ", event:getAssetId(), ", ", event:getMessage())
                self:startGame()
            elseif eventCode == cc.EventAssetsManagerEx.EventCode.ERROR_DECOMPRESS then
                print(event:getMessage())
                self:startGame()
            end
        end

        local listener = cc.EventListenerAssetsManagerEx:create(self.am_, onUpdateEvent)
        cc.Director:getInstance():getEventDispatcher():addEventListenerWithFixedPriority(listener, 1)

        self.am_:update()
    end

    if cc.Director:getInstance():getRunningScene() then
        cc.Director:getInstance():replaceScene(self)
    else
        cc.Director:getInstance():runWithScene(self)
    end
end

function UpdateScene:startGame()
    local searchPaths = cc.FileUtils:getInstance():getSearchPaths()
    table.insert(searchPaths, 1, self.storagePath_.."/res")
    table.insert(searchPaths, 1, self.storagePath_.."/src")
    cc.FileUtils:getInstance():setSearchPaths(searchPaths)
   
    require("app.MyApp").new():run()
end

return UpdateScene

启动方法：

local UpdateScene = require "app.scenes.UpdateScene"
local scene = UpdateScene.new()
scene:run()

QQ群：239759131 技术交流群欢迎您