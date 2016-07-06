# TDD入门

作为目前被广泛接收和采用的开发实践，TDD是任何有品位的开发者必须掌握的技能之一。

## 概念

TDD = Test-Driven Development，它重点强调是**测试驱动**，而不仅仅是测试。区别：
- 测试先行，而非先写产品代码再测试
    - 先写测试代码 - 失败 - 写产品代码 - 运行测试代码 - 成功 - 进入下一轮
- 先写测试代码有助于明确目标，站在调用者角度思考，利于设计出更友好的接口
- 为了书写测试，会强迫开发者思考可测性，有助于拿出更好的设计

至于自动化测试代码的其他好处，其实并非TDD专有，这里略过不谈，可自行google。

同时，即便是测试先行，其前提依然是对需求要理解清楚，并非是指没有文档就直接开写。虽然有些书籍上的例子给人感觉是这样，但在实际过程中，并非这么回事。

而且，对于初入行或刚加入项目组或刚接手代码的同事，有一份文档（需求和设计）也有助于快速理解，把握大方向。而自动化测试代码，更胜任那些需求细节的处理，充当代码保护网。因此，文档和自动化测试代码的分工在于：
- 文档：帮助建立框架性概念，适合描述高层知识
- 测试代码：充当代码的保护网，适合描述技术细节

## 实做

TDD的流程很简单，从大的方面来讲就是由大量“失败 - 成功”小循环来推动开发前行。实际过程就是（以书写一个Java类【计算器】为例）：
- 创建计算器对应的测试类
- 从待实现列表中抽取一个功能，如二元加法函数，实现其测试方法：testAdd
    - 以最快形式实现：先new，再直接调用add
- 由于计算器类并不存在，编译肯定会失败
- 创建计算器类，编译通过
- 运行测试代码，测试必然失败
- 为了让测试通过，必须实现add方法
- 运行测试代码，测试通过
- 修改测试代码，添加更多的测试场景，直至add方法达到要求
- 取下一个实现项目，重复此过程

这里需要重点强调的地方：
- 类的实现由测试推动逐步完成
- 有经验之后，会发现类的结构会依据测试慢慢演化：有可能会出现类的合并，也有可能会出现类的分拆。
- 功能实现之时，即是测试完成之时

## TDD != Test

在结束之前，需要强调一点：TDD并不能替代测试，TDD只能涵盖可自动化的部分，而测试的外延要广得多，如探索性测试，安全性测试，随机测试等等。

同时，也不要因为TDD中有T(est)就猛写测试不已。即便采用TDD，重点还是要给用户交活，交付产品代码。只不过是采用测试的方式驱动开发和设计的前行。而不是在此阶段就立马写出覆盖率很高的代码来，在实际中并不现实，无法一蹴而就。

鉴于TDD给人带来误解，让T(est)过于突出，如今更高阶的叫法是BDD，即Behavior-Driven-Development（行为驱动的开发）。强调通过用代码描述系统的行为方式，逐步让系统成型。当然，在技术上还是用测试代码来描述行为的。但此时，测试代码只不过是实现手段，不再属于概念本身的范畴。