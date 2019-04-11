4/8/2019  
## Agenda
1. Agenda（议程）
   议程是Rete算法特有的功能，它存有即将被执行的规则集合，它的工作是按照某种特定的顺序去执行规则。
    - 当处于RuleRuntime阶段的时候，规则可能被完全匹配从而准备去执行，一个单规则运行动作可能导致多规则的执行。当一个规则被完全匹配后，对应的规则的被匹配的事实被放入到Agenda（议程）中，Agenda控制匹配规则的执行顺序。  
2. Drools规则引擎循环重复两个阶段：
   1. Rule Runtime Actions：这是大部分工作发生的阶段，包括了Java应用流程和RHS执行阶段。一旦这个流程完成或者说Java应用流程调用fireAllRules()函数，Drools引擎将切换到Agenda Evaluation 阶段。
   2. Agenda Evaluation： 该阶段目的是选择一个想要触发的规则。如果没有触发该规则或者触发该规则，都会转去Rule Runtime Actions阶段。
- ***这两个阶段将会循环执行直到agenda中的存储集合为空，此时将会把控制权转换给应用，当处于第一阶段的时候，没有规则会被触发。***
  
## 冲突解决策略
- 首先，对于一个普通的规则而言，它的效果不依赖于特定的规则触发顺序才是合理和健康的，作者不需要担心“执行流”的问题。但是有时候必须要要求一个特定执行顺序的时候，就会有很多方案可选：salience（优先）、agenda group（议程组）、rule flow group（规则流组）、activation group（激活组）、control/semaphore facts（控制或者信号事实）。
1. Salience（优先级数字）
   - 当我们想要该规则在所有规则执行完后再执行，那么可以将该规则的salience变量置为负数（所有规则的默认salience数字为0），那么该规则就会最后执行。
    ```java
    rule "Print balance for AccountPeriod"
        salience -50
    when
        ap:AccountPeriod()
        acc:Account()
    then
        System.out.println(acc.accountNo + " : " + acc.balance);
    end
    ```
    - 如上述代码，那么名为"Print balance for AccountPeriod"的规则就会在最后执行（假设其他规则的优先级数字为默认值）。Drools6.0后改为设置优先级。（应该和这种差不多）
2. Agenda Groups（议程组）
    - Agenda Groups允许用户将规则放入到一个组内，然后这些组放入到一个栈内，栈拥有入栈和出栈的操作，调用"setFocus"函数可以将组放入栈内。
    ```JAVA
    ksession.getAgenda().getAgendaGroup("Group A").setFocus();
    ```
    - 议程永远首先执行在栈顶的组内的规则，当该组规则全部执行后，该组pop(),然后agenda执行下一个组。
    ```JAVA
    rule "increase balance for credits"
        agenda-group "calculation"
    when
        ap:AccountPeriod()
        acc:ACCount($accountNo:accountNo)
        CashFlow(type==CREDIT,
                 accountNo==$accountNo,
                 date>=ap.start&&<=ap.end,
                 $amount:amount )
    then
        acc.balance += $amount;
    end
    ```
    ```java
    rule "Print balance for AccountPeriod"
        agenda-group "report"
    when
        ap:AccountPeriod()
        acc:Account()
    then
        System.out.println(acc.accountNo + " : " + acc.balance );
    end
    ```
    - 首先，setFocus"report"组，然后focus on "calculation"，那么"calculation"将会首先执行。
    ```JAVA
    Agenda agenda=ksession.getAgenda();
    agenda.getAgendaGroup("report").setFocus();
    agenda.getAgendaGroup("calculation").setFocus();
    ksession.fireAllRules();
    ```
3. Rule Flow（规则流）
   - Drools允许用户用流程图的格式规定规则的执行顺序。（不详写了）

## Infrence（推断）

1. 什么是推断？推断就是根据已有的规则，必定会发生的事实。
```JAVA
rule "Infer Adult"
when
  $p : Person( age >= 18 )
then
  insert( new IsAdult( $p ) )
end
```
当Person实例的age大于等于18的时候，则会insert一个这个Person实例的IsAdult实例。
那么我们可以推断如果有一个IsAdult的实例的Person实例，它的age必定大于等于18.
2. 它的意义是什么？
   - 它可以把规则拆分，那么当规则中的某些条件发生变化的时候，不需要重新改变整个规则网络，只需要改变部分即可。
   - 例如有一个ID卡发行部门。该部门判断一个Person实例的location是否为London以及是否为成年人，若两者皆成立，则发行ID卡给他。那么这个部门就有两个LHS，即Person（locatoin=="London"）和Person（age>=18）,如果法律将成年人定义改成21岁，那么该部门的规则体系将发生重大改变。不利于大型规则的更改。
   - 那么此时，如果我有一个中央政府部门，它管理着成年人这个类，它的规则如上所述，当年龄》=18则insert一个IsAdult的实例。那么现在ID发行部门就不需要去具体管理年龄了，他的LHS变成了location以及这个实例是否拥有IsAdult的实例即可。至于这个人如何才能成为一个adult就不归他管，当法律发生改变时，它不需要做任何改变。只需要中央政府做改变即可。  
  
  ***所以一个好的规则应该是分块的、将具体规则压缩的（把age>=18压缩为IsAdult）、提供语义抽象的。而一个不好的规则表现为巨大的，充满漏洞的。***
  
## 逻辑对象的事实维护系统（TMS）
1. 事实的insert分为两种，一种是普通的插入，就是在java应用阶段的用户主动插入，这种插入我们称之为“说明插入”，也就是字面意思上的“说明某个事实”。使用hashmap和计数器，追踪有几个不同的示例符合该事实。
2. 另一种是逻辑插入，就是在RHS执行的时候的事实插入。只能有一个对象，下次堆该事实的插入只会增加这个事实的计数器。当LHS的条件不再为真的时候，计数器也会自动减一，如果计数器为零，则自动删除该事实。（有点类似智能指针）
3. 逻辑插入还有一个功能就是自动管理，比如p的age<16，那么insertLogical（new IsChild（$p）），那么假设当p的age改变了，大于等于16的时候，那么这个IsChild事实将会自动删除，另一个isAdult将会自动增加。  
   
***这段文字的介绍，我认为是介绍了一种Rete算法中的事实维护优化方式，它能够类似于智能指针一样自动析构事实，并且根据环境的变化能够自动改变事实集。相当于是自动维护，减少了人为操作。***
