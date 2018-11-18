# test-02分支用于Activiti原理测试
> * 无状态测试
> * 内存模型解析时间
> * 流程实例请求集中式执行

------

## 1. 无状态测试
### 目的
测试Activiti是否支持无状态

### 实验过程
测试Activiti是否支持无状态需要测试一个流程实例是否可以由两个或两个以上流程引擎执行，测试实验包括以下几种情况：
1. 先启动engine1，执行流程实例，关闭engine1，再启动engine1，测试是否可以继续执行流程实例
2. 先启动engine1，执行流程实例，关闭engine1，再启动engine2，测试是否可以继续执行流程实例
3. 先启动engine1，执行流程实例，再启动engine2，关闭engine1，测试是否可以继续执行流程实例

### 实验结果
流程实例都正常运行，Activiti支持无状态。


## 2. 内存模型解析时间
### 目的
测试将一个流程定义文件从xml转化成对应引擎的内存模型需要消耗的时间

### 实验过程
一个流程文件从xml转化成对应引擎的内存模型需要经过两部分：元素解析和对象解析。

* 元素解析：xml -> BpmnModel。
* 对象解析：BpmnModel - > ActivitiImpl/TransitionImpl

内存模型解析是通过调用org.activiti.engine.impl.bpmn.parser.BpmnParse.execute()方法实现，该方法包括元素解析和对象解析，因此只需要在执行解析代码前后加上时间戳就可以计算得到元素解析、对象解析以及整体解析相应的时间。

### 实验结果
启动流程实例会执行一次解析过程，因此可用于实验。使用travel-booking流程多次实验平均结果如下：

* 元素解析平均时间：**103ms**
* 对象解析平均时间：**38ms**
* 整体解析平均时间：**141ms**

## 3. 流程实例请求集中式执行
### 目的
测试将流程实例请求集中在少数引擎上执行的必要性；主要考虑两个方面，一个是模型的解析时间和内存占用；另一个是引擎缓存对于这种集中式执行的效果；（一般是如果缓存找不到才去外在找）；测试的结果可以体现在引擎的负载和流程实例的执行时长上；

### 实验过程与结果
1) 去掉模型解析时间，测试一个引擎与多个引擎下流程实例执行时长
引擎执行流程实例时只会在没有内存模型时进行解析流程定义文件操作，因此在第二次执行流程实例时就不会再次解析，此后的流程实例执行时长就相当去掉流程模型解析时间。在此基础上在一个引擎与两个个引擎上多次试验执行流程实例travel-booking，得出结果：
一个引擎平均流程实例执行时长：**1943ms**
两个引擎平均流程实例执行时长：**1973ms**
结果接近。

2）添加Redis缓存后，测试多引擎是否会再重新解析内存模型
	通过缓存BpmnModel，当一个引擎完成解析流程文档后将BpmnModel添加到redis，其他引擎不再需要解析同一个流程文档，直接从redis获取。经测试，引擎从redis获取BpmnModel后能继续正常运行，且实验测试引擎从redis获取BpmnModel的平均时间为47ms，比没有实现缓存、需要进行元素解析的平均时间103ms少了很多。

## 4. 流程实例并发请求集中式执行（暂时不做）
### 目的
测试流程实例并发请求在一个引擎上执行和分在多个引擎上执行的差异；

### 实验过程
1）设计包含并发流程的流程定义
2）去掉模型解析时间，测试一个引擎与多个引擎下流程实例执行时长