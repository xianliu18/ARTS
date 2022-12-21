### 规则引擎 Drools 入门

### 1. 业务背景
公司营销人员为了吸引更多的新客，和提高会员的活跃度，做了一期推广活动。抽奖活动奖品包括现金红包、积分、IPhone、坚果礼盒、京东 100 元（30 元、10 元）购物卡。

<table>
    <tr>
        <th>客户类型</th><th>条件</th><th>可参与的抽奖活动</th>
    </tr>
    <tr>
        <td rowspan="4">新客</td><td>首次绑定微信/支付宝</td><td>抽取 16 元现金红包</td>
    </tr>
    <tr>
        <td>首次关注公司指定公众号</td><td>抽取 16 元现金红包</td>
    </tr>
    <tr>
        <td>首次金融资产月日均 1000 元</td><td>抽取 188 元现金红包</td>
    </tr>
	<tr>
        <td>首次使用借记卡签约周周存/季季存</td><td>抽取 188 元现金红包</td>
    </tr>
	<tr>
        <td rowspan="3">老客</td><td>支付宝绑定银行卡，在线购物</td><td>抽取 8.18 元现金红包</td>
    </tr>
    <tr>
        <td>支付宝绑定银行卡，点外卖</td><td>抽取 8.18 元现金红包</td>
    </tr>
    <tr>
        <td>支付宝绑定银行卡，购买机票</td><td>抽取 8.18 元现金红包</td>
    </tr>
</table>

### 1. 什么是规则引擎
规则引擎由推理引擎发展而来，是一种嵌入在应用程序中的组件， 实现了将业务决策从应用程序代码中分离出来，并使用预定义的语义模块编写业务决策。

### 2. Drools
- KIE(Knowledge Is Everything)是一个综合项目。包括 Drools、jBPM、OptaPlanner、Business Central、UberFire。

![kie组件](https://github.com/xianliu18/ARTS/tree/master/tools/drools/images/kie组件.png)

#### 2.1 Drools 介绍
- Drools 的基本功能是将传入的数据或事实与规则的条件进行匹配，并确定是否以及如何执行规则。
- Drools 的基本组件：
  - `Rules`: 用户定义的业务规则，所有的规则必须至少包含触发规则的条件和触发规则后执行的操作；
  - `Facts`: 在 Drools 规则应用当中，将一个普通的 JavaBean 插入到`Working Memory`后的对象就是`Facts`对象，`Facts`对象是我们的应用和规则引擎进行数据交互的桥梁或通道；
  - `Production Memory`: 用于存放规则数据；
  - `Working Memory`: 用于存放 `Facts` 数据；
  - `Agenda`: 存放激活成功的规则；

![Drools组件](https://github.com/xianliu18/ARTS/tree/master/tools/drools/images/Drools组件.png)

#### 2.2 Drools 规则文件语法
- 主要在`.drl`文件中定义业务规则，关键字包含：`package`、`import`、`function`、`global`、`query`、`rule...end`等

```drl
package  //包名，这个包名只是逻辑上的包名，不必与物理包路径一致。
 
import   //导入类 同java
 
function  //  自定义函数
 
query  //  查询
 
global   //  全局变量
 
rule "rule name"  //  定义规则名称，名称唯一不能重复
    attribute //  规则属性
    when
        //  规则条件
    then
        //  触发行为
end
```

<br/>

**参考资料：**
- [Drools Documents](https://docs.drools.org/7.73.0.Final/drools-docs/html_single/index.html#_welcome)
- [规则引擎 Drools 在贷后催收业务中的应用](https://mp.weixin.qq.com/s/32O2KwQxlQodac0IpYClLQ)
