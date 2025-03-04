---
title: "通过Tampermonkey脚本抓取指定dom内容变化并通过wxpusher推送微信通知"
subtitle: " "
description: " "
date: 2025-03-04T06:42:06Z
author:      "Wayne"
image: ""
tags: ["Tampermonkey", "wxpusher", "微信事件通知"]
categories: ["Tech"]
---

## js

```
// ==UserScript==
// @name         IM Chat Message Notifier with wxpusher (Vue.js + Element UI)
// @namespace    http://tampermonkey.net/
// @version      1.0
// @description  监控Vue.js + Element UI开发的IM聊天窗口的新消息并通过wxpusher发送微信通知
// @author       Wayne
// @match        https://lechat.oa.fenqile.com/*
// @grant        GM_xmlhttpRequest
// ==/UserScript==

(function () {
    'use strict';

    // 配置
    const WXPUSHER_API_URL = "https://wxpusher.zjiecode.com/api/send/message";
    const APP_TOKEN = "AT_VATxxxxxxxxxxx"; // 替换为你的wxpusher AppToken
    const UIDS = ["UID_eoy2xxxxxx","UID_eTaIE1gCHlxxxxxxxN"]; // 替换为你的wxpusher UID，注意是数组形式

    // 存储上一次的消息内容
    let lastMessage = "";

    // 获取最新的消息内容
    function getLatestMessage() {
        const container = document.querySelector("#activeuserbox");
        const messages = container.querySelectorAll("div.activeuserinner  div.onemsg");
        //const messages = document.querySelectorAll(".onemsg"); // 替换为IM聊天窗口消息的选择器
        console.log("新消息长度:", messages.length);
        if (messages.length > 0) {
            return messages[0].innerText.trim(); // 获取最后一条消息的内容
        }
        return "";
    }

    // 发送通知到wxpusher
    function sendWxpusherNotification(message) {
        const payload = {
            appToken: APP_TOKEN,
            content: message,
            uids: UIDS, // 注意这里是 uids，且是一个数组
        };

        GM_xmlhttpRequest({
            method: "POST",
            url: WXPUSHER_API_URL,
            headers: {
                "Content-Type": "application/json",
            },
            data: JSON.stringify(payload),
            onload: function (response) {
                const result = JSON.parse(response.responseText);
                if (result.code === 1000) {
                    console.log("通知发送成功:", result.msg);
                } else {
                    console.error("通知发送失败:", result.msg);
                }
            },
            onerror: function (error) {
                console.error("请求失败:", error);
            },
        });
    }

    // 监听页面内容变化
    function observeChanges() {
        const targetNode = document.querySelector("body"); // 监听整个页面的变化
        const config = { childList: true, subtree: true };

        const callback = function (mutationsList, observer) {
            for (let mutation of mutationsList) {
                if (mutation.type === "childList") {
                    console.log("页面内容发生变化");
                    const latestMessage = getLatestMessage();
                    if (latestMessage && latestMessage !== lastMessage) {
                        console.log("检测到新消息:", latestMessage);
                        lastMessage = latestMessage; // 更新 lastMessage
                        sendWxpusherNotification(latestMessage); // 发送通知
                    }
                }
            }
        };

        const observer = new MutationObserver(callback);
        observer.observe(targetNode, config);
    }

    // 启动监听
    observeChanges();
    console.log("油猴子加载成功。");
})();
```
