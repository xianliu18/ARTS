### DDD 入门

领域驱动设计(DDD，Domain-Driven Design)是一种架构设计方法论。其核心思想是通过领域驱动设计方法定义领域模型，从而确定业务和应用边界，保证业务模型和代码模型的一致性。
DDD 包括战略设计和战术设计两部分：
- **战略设计主要从业务视角出发**，建立业务领域模型，划分领域边界，建立通用语言的限界上下文，
限界上下文可以作为微服务设计的参考边界。
- **战术设计则从技术视角出发**，侧重于领域模型的技术实现，完成软件开发和落地，包括：聚合根、实体、
值对象、领域服务、应用服务和资源库等代码逻辑的设计和实现。

### 1. 名词介绍
- DDD 的核心知识体系，具体包括
![核心知识体系](./images/核心知识体系.jpg)

### 2. 建立领域模型
- 事件风暴是建立领域模型的主要方法，它是一个从发散到收敛的过程。它通常采用用例分析、场景分析和用户旅程分析，尽可能全面不遗漏地分解业务领域，并梳理领域对象之间的关系，这是一个发散的过程。事件风暴过程会产生很多的实体、命令、事件等领域对象，我们将这些领域对象从不同的维度进行聚类，形成如聚合、限界上下文等边界，建立领域模型，这就是一个收敛的过程。

![领域模型](./images/领域模型.jpg)

#### 2.1 领域划分
- DDD 的领域就是这个边界内要解决的业务问题域；
- 领域会细分为不同的子域，子域可以根据自身重要性和功能属性划分为三类子域，分别是核心域、支撑域和通用域
  - **核心域：**决定产品和公司核心竞争力的子域是核心域，它是业务成功的主要因素和公司的核心竞争力；
  - **支撑域：**支撑核心域正常运行的领域，具备一定的独特性。
  - **通用域：**没有太多个性化的诉求，同时被多个子域使用的通用功能子域是通用域；如认证、权限、分布式唯一 ID 生成等；
  
#### 2.2 限界上下文（Bounded Context）
- 领域专家、架构师和开发人员的主要工作就是通过事件风暴来划分限界上下文。限界上下文确定了微服务的设计和拆分方向，是微服务设计和拆分的主要依据。
- 通用语言：在事件风暴过程中，通过团队交流达成共识的，能够简单、清晰、准确描述业务含义和规则的语言。它可以解决交流障碍，使领域专家和开发人员能够协同合作，从而确保业务需求的正确表达；
- 限界上下文：限界就是领域的边界，上下文则是语义环境；用来封装通用语言和领域对象，提供上下文环境，保证在领域之内的一些术语、业务相关对象等（通用语言）有一个确切的含义，没有二义性。

#### 2.3 实体（Entity）和值对象（ValueObject）
- 贫血模型：指仅用作数据载体，而没有行为和动作的领域对象；
- 充血模型：数据和对应的业务逻辑被封装到同一个类中。

实体和值对象是组成领域模型的基础单元。实体具有业务属性、业务行为和业务逻辑。而值对象只是若干个属性的集合，只有数据初始化操作和有限的不涉及修改数据的行为，基本不包含业务逻辑。
- 实体：
  - 拥有唯一标识符，且标识符在经历各种状态变更后仍能保持一致；
  - 重要的不是其属性，而是其延续性和标识。如：客户基础信息等。
- 值对象：
  - 通过对象属性值来识别的对象，它将多个相关属性组合为一个概念整体；
  - 不可修改；如地址（省市区街道）等。

#### 2.4 聚合和聚合根
- 聚合：是由业务和逻辑紧密关联的实体和值对象组合而成。聚合是数据修改和持久化的基本单元，每一个聚合对应一个仓储，实现数据的持久化。
  - 聚合在 DDD 分层架构里属于领域层，领域层包含了多个聚合，共同实现核心业务逻辑。聚合内实体以充血模型实现个体业务能力，以及业务逻辑的高内聚。跨多个实体的业务逻辑通过领域服务来实现，跨多个聚合的业务逻辑通过应用服务来实现。
- 聚合根：如果把聚合比作组织，那聚合根就是这个组织的负责人。聚合根也称为根实体，它不仅是实体，还是聚合的管理者。
  - 聚合根作为实体，拥有实体的属性和业务行为，实现自身的业务逻辑；
  - 聚合根作为聚合的管理者，在聚合内部负责协调实体和值对象按照固定的业务规则协同完成共同的业务逻辑；
  - 外部对象不能直接访问聚合内实体，需要先访问聚合根，由聚合根导航到聚合内部实体。

#### 2.5 康威定律
- 任何组织在设计一套系统（广义概念上的系统）时，所交付的设计方案在结构上都与该组织的沟通结构保持一致。
	- 即系统结构应尽量与组织结构保持一致。



<br/>

**参考资料：**
- [DDD 实战课](https://zq99299.github.io/note-book2/ddd/)
- [领域驱动设计在互联网业务开发中的实践](https://tech.meituan.com/2017/12/22/ddd-in-practice.html)