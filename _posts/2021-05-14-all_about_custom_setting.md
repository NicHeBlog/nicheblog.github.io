---
title: About Custom Setting
categories:
- Tech 
date: 2021-05-14
excerpt: "Custom setting memorandum"
feature_image: "https://picsum.photos/id/870/600?image=872"
---
<div id="阅读时长15min" style="color:rgb(168,173,172)">阅读时长：15min</div>
---
### 简介
本文会介绍一下Custom Setting的使用<br/>
因为总是容易忘记，每次使用都要大脑扫描一遍记忆，索性写篇文章记录一下<br/>
* --- Part 1： Custom Setting是个啥<br/>
* --- Part 2： Custom Setting的调用<br/>
* --- Part 3： 小结<br/>


### Part 1: Custom Setting是个啥
<hr/>
Custom Setting的使用和Custom Object几乎一样，可以认为Custom Setting就是一种特殊的Custom Object<br/>

用户可以创建一组的数据，这些数据会被存储在app的缓存之中，方便Apex Trigger, Validation Rule, Flow, Formula等等每次快速调用，而不用再从系统的Database中query数据。<br/>

Salesforce系统提供两种Custom Settings: <br/>
1. List Custom Settings : <br/>
   相当于声明了一组static的数据，里面维护的数据可以被整个orgnizatio调用，<br/>
   一般当需要频繁调用某些数据的时候，会选择使用Custom Setting来承载，因为是cache在系统中的，这样会大大减少系统的消耗，提高运行效率。<br/>
   List类型的Custom Settings里面的数据不会因为Profile或者User的不同而改变。
<br/>

2. Hierarchy Custom Settings : <br/>
   可以根据不同的Profile或者User来控制对于setting里面的data的可见性。其余用法和List类型的一致。


### Part 2: Custom Setting的调用
<hr/>
代码中调用Custom Setting的时候，只需要把它当作是一个特殊的Object使用就行了，但是又不同于object，它有一些自己独有的方法。<br/>
Custom Setting可以支持创建自定义的字段，这就和Object下面创建自定义字段是一样的，用来存储额外的信息。<br/>

1. getInstance()/getValues()
2. getall()
<br/>
举个例子，比如在系统中创建了如下的Custom Setting： <br/>
Label: test custom setting  <br/>
API: test_custom_setting__c <br/>
这个Custom Setting有一个自定义的字段叫做testCustomField__c <br/>

此时该Custom Setting在系统中其实就是一个mapping关系，key是Name这个standard字段，Value对应的就是整个Custom Setting的记录值。<br/>

点击manage可以创建该Custom Setting的具体记录，比如创建一条记录，Name值给定‘123box’<br/>
还可以像一般的object页面一样创建一个list view来展示Custom Setting中的字段，方便查看。<br/>

**1. getInstance()** <br/>
  test_custom_setting__c cus = test_custom_setting__c.getInstance('123box');
  system.debug('cus:'+ cus);
  通过‘key’值就可以直接的取到对应的value，如下：
  ```
  cus:
  test_custom_setting__c:{
          LastModifiedDate=2021-05-14 08:59:33, 
          IsDeleted=false,  
          CreatedById=0051T000008y8RMQAY, 
          CreatedDate=2021-05-14 08:59:33, 
          Id=a9D05000000YU6tEAG, 
          CurrencyIsoCode=CNY, 
          LastModifiedById=0051T000009u9RMQAY, 
          SetupOwnerId=00D050000007gCDEAY, 
          Name=123box, 
          testCustomField__c=test,
          SystemModstamp=2021-05-14 08:59:33
  } 
  ```
  其中LastModifiedDate，IsDeleted，CreatedById等都是系统的标准字段。
  也可以通过链式关系直接取到你自定的字段的值。如下：
  system.debug('custom field value: '+ cus.testCustomField__c);


**2. getall()** <br/>
  通过getall()方法则可以一次性的把该Custom Setting中所有的记录都取出来，当然如上所述，这些记录会在页面系统加载的时候，已经被存放在系统内存中了。
  比如又往test_custom_setting__c中存储了几条数据，
  执行test_custom_setting__c.getall() :
  ```
   { 
    1033edg = test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:58:30, 
                     IsDeleted=false,  
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:58:30, 
                     Id=a9D05000000YU6eEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000009u9RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=1033edg, 
                     testCustomField__c=test1,
                     SystemModstamp=2021-05-14 08:58:30
                    }, 
    123box = test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:59:26, 
                     IsDeleted=false, 
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:59:26, 
                     Id=a9D05000000Yu6oEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000008y8RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=123box,
                     testCustomField__c=test2, 
                     SystemModstamp=2021-05-14 08:59:26
                    }, 
    456test = test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:59:40, 
                     IsDeleted=false,  
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:59:40, 
                     Id=a9D05000000YU6yEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000009u9RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=456test, 
                     testCustomField__c=test3,
                     SystemModstamp=2021-05-14 08:59:40
                    }
  }
```
可以看到getall()放大执行之后，得到的其实是一个Map (<Name,CustomSetting>),
Map的Key是该CustomSetting的Name Field，Value就是该CustomSetting记录本身。

所以可以通过test_custom_setting__c.getall().keySet()获取如下结果： 
{1033edg, 123.com, 123box, 456test}
  
通过test_custom_setting__c.getall().Values() 获取如下数据： 

```
（ 
   test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:58:30, 
                     IsDeleted=false,  
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:58:30, 
                     Id=a9D05000000YU6eEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000009u9RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=1033edg, 
                     testCustomField__c=test1,
                     SystemModstamp=2021-05-14 08:58:30
                    }, 
   test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:59:26, 
                     IsDeleted=false, 
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:59:26, 
                     Id=a9D05000000Yu6oEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000008y8RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=123box,
                     testCustomField__c=test2, 
                     SystemModstamp=2021-05-14 08:59:26
                    },
   test_custom_setting__c:{
                     LastModifiedDate=2021-05-14 08:59:40, 
                     IsDeleted=false,  
                     CreatedById=0051T000008y8RMQAY, 
                     CreatedDate=2021-05-14 08:59:40, 
                     Id=a9D05000000YU6yEAG, 
                     CurrencyIsoCode=CNY, 
                     LastModifiedById=0051T000009u9RMQAY, 
                     SetupOwnerId=00D050000007gCDEAY, 
                     Name=456test, 
                     testCustomField__c=test3,
                     SystemModstamp=2021-05-14 08:59:40
                    }
）
```

获取到Values值之后，可以方便的进行链式调用。

