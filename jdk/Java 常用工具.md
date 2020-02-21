# Java 常用工具

## 新认识的工具

* BTace,动态修改java字节码文件的工具；
    - Arthas，基于BTace实现的Java诊断工具；


## 值得借鉴的设计

* entity 与 DTO转换，查看guava的com.google.common.base.Convert
* Lombok的设计
    - lombok高阶使用方法：
        + @Accessors(chain = true)，链式调用方式；
        + 静态构造方法：@RequiredArgsConstructor(staticName = "ofName")
        + @Builder，lombok的建造者模式；
        + @Delegate，lombok的代理模式；





## 值得借鉴的思想

* 拥抱变化，持续重构；







## 值得借鉴的做法

* 多看成熟框架的源码
* 多回头看自己的代码
* 勤于重构




## 值得学习的书

* 《Clean Code》(代码整洁之道)，Robert C. Martin
* 《IMPLEMENTING DOMAIN-DRIVEN DESIGN》(实现领域驱动设计)，Vaughn Vernon
* 《Refactoring Imporving the Design of Existing Code》(重构 改善既有代码的设计)，Martin Fowler
* 《Linux私房菜》

## 值得深入学习的高阶知识

* ASM
* cglib
* spring aop动态代理


