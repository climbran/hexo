title: 数据库四种设计模式
date: 2015-08-29 21:17:26
tags: [数据库]
---
1、主扩展模式<br>
主扩展模式，通常用来将几个相似的对象的共有属性抽取出来，形成一个“公共属性表”；其余属性则分别形成“专有属性表”，且“公共属性表”与“专有属性表”都是“一对一”的关系。<br>
感觉主扩展模式适合用于继承关系，“公共属性表”即父类表，“专有属性表”即各个子类各自独有的属性形成的表。主扩展模式用来描述对象内的关系。<br><br>
2、主从模式<br>
即对象间的一对多关系，例如学校和班级即一种主从模式。<br><br>
3、名值模式
名值模式，通常用来描述在系统设计阶段不能完全确定属性的对象，这些对象的属性在系统运行时会有很大的变更，或者是多个对象之间的属性存在很大的差异。<br>
我的理解名值模式就是主扩展模式和多对多模式的混合，将主从模式中的“专有属性表”中列变为一张“属性列表“”中的行。“公共属性表”和“属性列表”为多对多关系，属性值可在映射表中。用来描述对象内的关系。
eg:
<img width="600 "src="/img/database-design-model/1.png"/><br><br>
4、多对多模式<br>
这个没什么好说的，可分为关联表有独立的业务处理需求（映射表除映射功能还有其他属性）和关联表没有独立的业务处理需求（映射表只有映射功能）。多对多模式描述对象间的关系。<br><br>
总结：<br>
以上4种模式其实就是分别描述了对象内的一对一关系、对象内的多对多关系、对象间的一对多关系，对象间的多对多关系，基本能应用所有情况，其他关系有：
1、对象内的一对多关系：多方会存在冗余信息，将冗余信息提取到一张表后即转换为名值模式；
2、对象间的一对一关系：貌似不存在这种关系，本想如婚姻关系是对象间的一对一关系，但考虑离婚等情况其实也是多对多关系。