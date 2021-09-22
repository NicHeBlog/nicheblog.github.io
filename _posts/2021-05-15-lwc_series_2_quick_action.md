---
title: LWC Series2 - LWC Quick Action Release
categories:
- Tech 
date: 2021-05-15
excerpt: "LWC Series 2"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---

### 背景介绍
LWC框架已经发布一年多了，框架也一直也演进优化中。<br/>
这次的Summer 21 Release终于直接支持了Global Action直接调用LWC component.<br/>
本文做一个简单的Demo，快速的看一下这个新鲜出炉的功能。
<br/>
<br/>
### Screen Action：正常的popup模态弹框
新功能发布之前，要想使用LWC框架写一个popup弹框，需要在LWC之外包裹一层Aura component.
然后通过Object中创建一个Action来调用Aura才行。举个例子： 

```
<aura:component implements="force:lightningQuickActionWithoutHeader, force:hasRecordId" access="public">
    <aura:attribute name="recordId" type="String" />
    <c:testLwcComp recordId="{!v.recordId}" onclose="{!c.handleClose}" />
</aura:component>

```
其中testLwcComp是一个LWC Component.<br/>
事实上，Aura层什么也不做。甚至有时候Aura还会带来一些副作用，比如有时候需要一个loading画面来等待数据加载，设计遮罩层的时候，还要考虑Aura层的问题。<br/>
这样的事情一去不复返了，新版本release之后，可以如下直接使用。举个例子：

**HTML Part: lwcQuickActionDemo.html**
```
<template>
    <lightning-card title="Hello" icon-name="standard:contact">
        <div class="slds-var-m-around_medium">Hello {yourname}!</div>
        <div class="slds-var-m-around_medium">Hello {yourname}!</div>
        <div class="slds-var-m-around_medium">Hello {yourname}!</div>
    </lightning-card>
</template>
```

**JS Part: lwcQuickActionDemo.js**
```
import { LightningElement,api } from 'lwc';

export default class TestLWCAction extends LightningElement {
    @api yourname;
    yourname = 'Nic';
}
```

**Metadata Part: lwcQuickActionDemo.js-meta.xml**
<br/>
这其中需要在targetConfigs中指定actionType为**ScreenAction**

```
<?xml version="1.0" encoding="UTF-8" ?>
<LightningComponentBundle xmlns="http://soap.sforce.com/2006/04/metadata">
    <apiVersion>51.0</apiVersion>
    <isExposed>true</isExposed>
    <targets>
        <target>lightning__AppPage</target>
        <target>lightning__RecordPage</target>
        <target>lightning__HomePage</target>
        <target>lightning__RecordAction</target>
    </targets>
    <targetConfigs>
        <targetConfig targets="lightning__RecordAction">
        <actionType>ScreenAction</actionType>
   </targetConfig>
 </targetConfigs>
</LightningComponentBundle>
```
效果如下图,和用Aura包裹后的效果一模一样：<br/>
![screenAction](/assets/lwcQuickAction/screenAction.png "screenAction")

<br/>
<br/>

### Headless Action: 直接显示在Quick Action Button下面的弹出框

试了一下，看起来这个功能暂时支持还不是很好，感觉功能好像不是很完备的样子。<br/>

比如，点击出现的弹出框还不能被隐藏掉；内容如果太长的话，整个hightlight panel会被撑开等。<br/>

实现这种效果只需要把上述例子中的xml文件的targetConfigs中actionType改为**Action**即可。

最终效果如下图：<br/>

![headlessAction](/assets/lwcQuickAction/headlessAction.png "headlessAction")

完。

>Reference: https://help.salesforce.com/articleView?id=release-notes.rn_lwc_quick_actions.htm&type=5&release=232
