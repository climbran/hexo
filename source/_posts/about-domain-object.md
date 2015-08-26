title: 关于领域模型（贫血模型，充血模型）
date: 2015-07-30 19:31:51
tags: [web]
---
领域模型(domain object)分为4类:<br>
失血模型<br>
贫血模型 service --> dao --> entity <br>
充血模型 service --> entity <--> dao <br>
胀血模型<br>

简单来说，失血模型就是纯数据POJO，业务逻辑完全与entity分离；贫血模型domain object中含有与持久化无关的逻辑，不依赖于dao层；充血模型把domain object和business object合二为一，service层不依赖dao层，而entity层与dao层形成双向依赖；胀血模型直接取消service层，只剩下entity层与dao层。

由于今年才开始做web开发，之前对各种模型完全没有了解，项目中的模型是由自己瞎摸索出来的。现在总结来看，应该属于贫血模型，但是把大量service层的业务逻辑推到controller层实现，这种方式前期开发相当迅速，但是后期发现严重影响代码重用，不同controller里存在大量重复代码，导致需求更改时需要修改多个controller，影响项目维护。

我感觉目前使用的模型适合小项目的快速开发，但对于大点的项目还是传统的贫血模型更合适。

附：不同模型的优缺点<br>
一、贫血模型<br>
优点：<br>
1、各层单向依赖，结构清楚，易于实现和维护<br>
2、设计简单易行，底层模型非常稳定<br>
3、非常适合于软件外包和大规模软件团队的协作。每个编程个体只需要负责单一职责的小对象模块编写，不会互相影响。<br>
缺点：<br>
1、domain object的部分比较紧密依赖的持久化domain logic被分离到Service层，显得不够OO<br>
2、Service层过于厚重<br>
3、可复用的颗粒度比较 小，代码量膨胀的很厉害，最重要的是业务逻辑的描述能力比较差，一个稍微复杂的业务逻辑，就需要太多类和太多代码去表达<br>
4、对象协作依赖于外部容器的组装，因此裸写代码是不可能的了，必须借助于外部的IoC容器。<br>

二、充血模型<br>
优点：<br>
1、更加符合OO的原则。
2、对象自洽程度很高，表达能力很强，因此非常适合于复杂的企业业务逻辑的实现，以及可复用程度比较高。<br>
3、Service层很薄，只充当Facade的角色，不和DAO打交道。<br>
4、不必依赖外部容器的组装，所以RoR没有IoC的概念。<br>
缺点：<br>
1、DAO和domain object形成了双向依赖，复杂的双向依赖会导致很多潜在的问题。<br>
2、如何划分Service层逻辑和domain层逻辑是非常含混的，在实际项目中，由于设计和开发人员的水平差异，可能导致整个结构的混乱无序。<br>
3、考虑到Service层的事务封装特性，Service层必须对所有的domain object的逻辑提供相应的事务封装方法，其结果就是Service完全重定义一遍所有的domain logic，非常烦琐，而且Service的事务化封装其意义就等于把OO的domain logic转换为过程的Service TransactionScript。该充血模型辛辛苦苦在domain层实现的OO在Service层又变成了过程式，对于Web层程序员的角度来看，和贫血模型没有什么区别了。<br>
4、对象高度自洽的结果是不利于大规模团队分工协作。一个编程个体至少要完成一个完整业务逻辑的功能。对于单个完整业务逻辑，无法再细分下去了。<br>

三、胀血模型<br>
优点：<br>
1、简化了分层<br>
2、也算符合OO<br>
缺点：<br>
1、很多不是domain logic的service逻辑也被强行放入domain object ，引起了domain ojbect模型的不稳定<br>
2、domain object暴露给web层过多的信息，可能引起意想不到的副作用。<br>

参考：<br>
<a href="http://www.cnblogs.com/magiccode1023/archive/2012/11/06/2756234.html">关于架构设计的“贫血模型”与“充血模型”</a><br>
<a href="http://www.oschina.net/question/54100_10400">［转帖］贫血、充血模型的解释以及一些经验</a>