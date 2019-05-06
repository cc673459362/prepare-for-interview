# CMU-RETE算法论文阅读笔记
1. AI领域的发展，导致机器需要学习的规则变得越来越多，导致整个系统的规则匹配速度下降，当规则数量到一定规模，则该专家系统会瘫痪，所以减少规则匹配的时间很重要。
2. （1.1）减少匹配时间的方法主要有两种：
   1. 遗忘模式：专家系统拥有“遗忘”功能，根据一定策略去遗忘系统内的规则，这样会让系统内的规则规模一直保持在可以让系统运行比较高效的状态下。（这种方式是对规则的各类放弃方式的统称，也有一些做法是对进来的规则进行过滤，只对特定的规则加入到专家系统，防止大量规则无限扩大系统规模。）但这种方法有一定的局限性：暂时没看懂，见P16。第二段
   2. 降低单个规则匹配消耗：比如防止形成一些非常“昂贵”的规则。系统在形成规则网络的同时，优化规则网络，降低单个规则的匹配费用。这种方法的局限性很明显，它能减少单条规则的匹配时间，但是随着AI的发展，规则的数量将会迅速增长，这必定不是一个长远的解决方案。
3. （1.2）本篇论文的主要研究点是针对大规模规则下的专家系统运行效率的研究。一般规则数量大于100000条。并且实验使用了大量的学习系统，经研究发现，RETE和TREAT算法的表现最佳，但两者还是随着规则数量增加，效率有线性的降低。这种线性的降低导致了它们对于大规模的学习系统的支持还是有问题。
4. （1.2）本篇论文将RETE算法修改为RETE/UL算法，大幅增强了对于大规模规则系统的支持，比普通算法快2个量级。
5. （1.3）影响学习系统的效率的因素其实不仅仅是规则的规模，还有其他问题：例如一些庞大规则，这样一条非常庞大的单体规则可能也会影响学习系统。但是本论文不讨论这些问题因素。只关注规则数量对专家系统效率的影响。
6. （1.3）本篇论文无法完全解决大规模规则匹配昂贵问题，且这个问题理论上是永远无法彻底解决的。
7. （1.3）本篇论文的方法针对大规模匹配，1000-100000，对于小规模规则10-100数量可能没有效果。
8. （1.3）本篇论文只针对解决大数量的问题，不解决算法本身的匹配效率问题。
9. （1.3）本篇论文假设所有数据可以存在内存中，而不会考虑硬盘问题。有一些研究已经在研究高效的数据库存取方法。
10. （1.3）本篇论文研究的是序列化匹配系统，不考虑并行匹配。虽然并发匹配时解决大规模规则匹配的一个很好的方法，但是在单CPU下序列化匹配的优化对于并行匹配的设计也有很好的指导性。
11. （1.3）本篇论文解决的时全局匹配，而不是部分匹配。有论文提出在匹配的时候去发现在大型规则库内的部分规则去匹配的方法降低匹配消耗。这种方法缺乏在不同情况下的适用性，但也是一个很好的点子。  

## Basic Rete
1. Rete算法包含一个PM（Production Memory）和WM（Working Memory）。WM里存的元素是外部世界的事实（fact）或者是内部处理后的状态（state）。我们将它们称之为WME（Working Memory Element）。
下面的例子就是典型的WMEs：
  ```
  w1:(B1 ^on B2)                 w6:(B2 ^color blue)
  w2:(B1 ^on B3)                 w7:(B3 ^left-of B4)
  w3:(B1 ^color red)             w8:(B3 ^on table)
  w4:(B2 ^on table)              w9:(B3 ^color red)
  w5:(B2 ^left-of B3)
  ```
2. 从上述例子中可以看到，我们的WMEs由一个三元组构成（three-tuples）,用（identifier ^attribute value）的形式来描述。这三个英文名称对于匹配没有特殊的含义。这三者的约束只要是一个常量不是变量就行。
3. PM是产品（rules）的集合。产品通常由一系列的条件，我们称之为LHS，以及一系列的动作，我们称之为RHS组成。Productions通常以下列形式写就：
   ```
   name-of-this-production
   LHS   /*one or more conditions*/
   -->
   RHS   /*one or more actions*/
   ```

4. 匹配算法只针对LHS，它遍历系统去决定哪些规则被完全匹配了。另外的模块负责去执行这些规则的RHS。条件里可以包含变量。下面的例子表示了寻找两个或更多的在红色箱子左边的箱子堆。 
    ```
    find-stack-of-two-blocks-to-the-left-of-a-red-block
     (<x> ^on <y>)
     (<y> ^left-of <z>)
     (<z> *color red)
    -->
    RHS
    ```
5. 匹配算法决定哪些规则可以被当前的WM中的事实所完全匹配。由于我们假定WME由一个三元组构成，所以我们也可以假定每个条件由一个三元组构成。否则两者不可能匹配。
6. Rete算法可以被看成是一个黑盒子，他有两个输入，一个是PM，一个是WM，它只负责将PM中的规则与当前WM中的WME去匹配，如果完全匹配，则交给另外的模块去执行该规则的动作。
   ![RETE黑盒子](http://ww1.sinaimg.cn/large/005xfSxkly1g1yq2rev60j30r10ggq58.jpg)
7. Rete使用数据流网络去代表规则的条件。网络分为两个部分。alpha网络用来测试WME。它将符合条件的WME存储到AM(alpha memories)中。
![RETE网络图](http://ww1.sinaimg.cn/mw690/005xfSxkly1g1yqmo0c6bj30yq0l1aj7.jpg)

  从上图中可以看到。beta网络是由join节点和beta memories构成的。（可能还会有其他一些节点，后面讲。）Join节点的功能是测试两个条件的一致性，也就是说，它的左右输入都是已经符合某个或者某类条件了，那么这个join节点的功能就是将左右已经符合某类条件的事实库再次测试其两边都符合的事实。将它们存到自己的beta memories中。所以beta memories中存储的是符合该规则的部分条件的WMEs，只有走到根节点才是符合该规则所有条件的WMEs。

8. 严格来说，大部分Rete网络的alpha网络不仅仅测试常量比如在单个条件中出现 \<x\> ^ self \<x>.但是这里，论文里很少出现这种条件，且本篇论文的大部分条件都是“相等”判断，连“大于”、“小于”等判断都很少。总之，在基本情况下，alpha网络表现为测试单个WME，而beta啊网络表现为测试两个及两个以上的WMEs。
9. 上述过程也可以用数据库去类比。Working Memory作为关系型数据库的表，而Production则是作为Query。不同的条件匹配就可以看作是对数据库内容的select，将select得到的结果存储与alpha memory中。最终符合规则P的working memory 将会是把前面分别select得到的结果通过join select联合起来。join operation由Beta网络的Join节点完成。每个beta memory里存储了每次join操作得到的结果。每当working memory发生改变，我们便通过alpha网络传送这些改变，并更新alpha memory。这些更新将会被传播到相关的join节点。如果join节点中有实例被改变，那么我们便更新这个beta memory，并将其改变往下传播，知道到达最终的执行节点。
   - 例如，有n个condition c(n)在系统内，那么我们就将他们各自的select结果r(cn)存储在相应的alpha memory中，现在如果规则P由C1...C2...Ck构成。那么如果P被匹配，则过程就是r(c1)Xr(c2)X...xr(ck).其中的X代表着JOIN操作。
10. 能够让RETE算法比普通匹配算法块的原因在于Rete算法可以保存状态。每次改变WM，那么在alpha和beta网络中得到的匹配结果都会保存，那么下次改变WM的时候，就不用所有WMEs都进行重新计算，那么就减少了很多重复计算。（Rete算法的状态保存特性使得它在那种小规模变化的WMEs中表现良好，但并不适用与大规模WMEs的变化。）
11. 第二点则是节点的共享，当production中由一些conditions是相同的，那么这两个production就可以共享这些节点。例如上图中的alpha memoryC3被production P1和P3共享。再例如在beta网络中，如果由一些production的部分condition相同，则可以共用一些beta节点，而不必重新复制一个重复的节点。由于这些节点共享，所以beta网络看上去像是一颗树。
12. Rete模型系统分为四个切入点：add-wme，remove-wme，add-production,remove-production。
13. add-wme。当一个wme被加入到working memory中，那么这个alpha网络将对他进行必要的常量测试然后将他存储与合适的alpha memories中。有几种方法可以去找到合适的alpha memories。
    - Dataflow Network    
    ![123](http://ww1.sinaimg.cn/mw690/005xfSxkly1g25nu6a2mlj30zp0pm0zv.jpg)
    - 如上图，这是一个简单的规则系统网络图，含有C1-C10 10条conditions。对于每个condition，我们做出k个节点做常量测试，用这k个节点作为路径使得WME在路径上进行测试流动。这k个节点的建立过程中，如果本condition的某个节点和另外一个condition的某个节点常量测试一样，则共享该节点。最后在节点末尾增加一个alpha memory。作为wme完成常量测试的输出。
    - 从上图中可以看到，我们只关注了它的constants，而没有关注variable name。因此，C2和C10共享了节点和alpha memory即便他们有不同的variable names。而且condition也有可能不包含任何节点，直接作为top节点的子节点，犹如C9。
14. 我们可以把上述节点用数据结构的形式表示：
    ```
    structure constant-test-node
      field-to-test:"identifier","attribute","value",or"no-test"
      thing-the-field-must-equal:SYMBOL
      output-memory:alpha-memory or NULL
      children: LIST of constant-test-node
    end
    ```
    no-test是用在top节点的。当一个WME被加入到working memory，我们将把它放入到dataflow的top节点中。
    ```
    procedure add-wme(w:WME){dataflow version}
      constant-test-node-activation(the-top-node-of-the-alpha-network,w)
    end
    ```
    该add-wme的procedure将调用constant-test-node-activation这个递归的procedure，递归完成这个WME在alpha网络中的传播。
    ```
    procedure constant-test-node-activation(node: constant-test-node;w:WME)
      if node.field-to-test≠'no-test' then
        v=w.[field-to-test]
        if v≠node.thing-the-field-must-equal then
          return {failed the test, so don't propagate any further}
      if node.output-memory ≠ NULL then
        alpha-memory-activation(node.output-memory,w)
      for each c in node.children do constant-test-node-activation (c,w)
    end
    ```
    通过上面这个递归调用，这个WME要么被中途抛弃，要么被放入到合适的alpha memory中了。但要注意上面的伪代码只示例的=和≠这种判断方式，一般的condition应该包含＞、＜、≥、≤等各种不等关系的判断。这样也许在 node的struct中需要再添加一项判断类型。并且修改之后的constant-test-node-activation这个procedure了。(但原理是一样的。)

  15. Dataflow Network with Hashing
      - 从上面的网络图中，我们可以看到当节点符合attr==color?的时候，它将会有5个子节点，且这5个子节点是互斥的。随着系统的不断学习和增大，那么这些节点的数量将会越来越多，那么匹配的速率就会越来越慢。（从上面的伪代码可以看到，对于子节点的activation是采用for循环去遍历做的。）
      - 一种显而易见的改进方式就是将这些大扇出的节点用一个特殊的节点代替，这个节点存储有一张hash表，它可以快速决定应该执行那个子节点。所以上图中的5个color的子节点可以不用遍历，直接在hash表中查找这个WME的“value”值，然后根据查找到的内容执行接下去的传播。事实上，这些子节点完全可以省略了，因为这个hash节点已经可以做到完全一样的效果。如下图优化所示。
  ![hash](http://ww1.sinaimg.cn/mw690/005xfSxkly1g25pgz5f4tj31330qr7ak.jpg)
      - 这张图中还是只包含了“相等”这种判断模式。从这张图，我们还可以观察得到，由于我们已经假设condition都有三元组构成（v1 ^v2 v3），那么，任何WME都最多只有8个alpha memory可以进入。如果一个WME w=（v1 ^v2 v3）进入了alpha memory a，那么a肯定得拥有下列8种形式之一：
      ```
      (* ^* *)               (v1 ^* *)
      (* ^v2 *)              (* ^* v3)
      (v1 ^v2 *)             (v1 ^* v3)
      (* ^v2 v3)             (v1 ^v2 v3)
      ```
      现在alpha memory只有8个，也就是说当一个WME：m进入的时候，只需要在这8个中找应该进入哪个memory就行了。（甚至更少，因为可能不存在符合上述8种之一的条件。）根据上述观察，我们可以无论如何，把alpha网络的top节点之后，跟8个hash节点，分别对应上述情况，然后当一个wme进来之后，在这8个hash节点里查找是否存在这个wme匹配的条件，如果存在，则加入到响应的memory中。否则，查找另一个hash节点，直到所有都查找完。
      ```
      procedure add-wme(w:WME){exhausive hash table version}
        let v1,v2,v3 be the symbls in the three fields of w
        alpha-mem = lookup-in-hash-table(v1,v2,v3)
        if alpha-mem ≠ "not-found" then alpha-memory-activation (alpha-mem,w)
        alpha-mem = lookup-in-hash-table(v1,v2,*)
        if alpha-mem ≠ "not-found" then alpha-memory-activation (alpha-mem,w)
        alpha-mem = lookup-in-hash-table(v1,*,v3)
        if alpha-mem ≠ "not-found" then alpha-memory-activation (alpha-mem,w)
        alpha-mem = lookup-in-hash-table(*,v2,v3)
        if alpha-mem ≠ "not-found" then alpha-memory-activation (alpha-mem,w)
        .
        .
        .
        alpha-mem = lookup-in-hash-table(*,*,*)
        if alpha-mem ≠ "not-found" then alpha-memory-activation (alpha-mem,w)
      end
      ```
      上述算法基于两个假设：1）WMEs是三元组2）常量测试条件都是相等。其中第一个假设可以被放宽为r元组。当WME是r元组的时候，那么我们可以使用2的r次方个hash表查找。当然了，r不宜过大。对于第二个假设，我们可以有两种方法去消除：
        - 第一种方法，前面和上面相等测试一样，做出8个hash表查找，但是找到之后，WME并不直接进入alpha memory而是再经过像上面曾经用过的数据流网络那样，把不同的“不等式”测试条件分开，进入不同的alpha memory。所以这种解决方法是将alpha net分为两层，上面一层还是hash查找，后面一层是数据流流动。
        - 第二种方法则是将不等判断交给beta网络。其实beta网络也可以做一些常量判断，而且并不会增加很大的beta网络的负担。  
  
      总体来说，alpha网络的效率都是很高的，对于每次WM的变化，都是常数级的复杂度。RETE算法的主要耗费都是在beta网络。  
  16.  Memory Node 实现
         - 回忆一下alpha memory里存储的是WMEs的集合。而beta memory存的是tokens的集合。token指的是WME序列。比如既满足C1又满足C2的事实序列（W1&W2,从C1 AM取出一个WME W1，从C2 AM取出一个WME W2，拼成一个token）。  
        根据下面两个标准，有2种实现memory node的方式：
            - 这个（WME或者token的）集合是用什么数据结构如何构建的？
            - token-WME序列是如何指代的？
        - 还是使用上面讲过的production例子：
          ```
          find-stack-of-two-blocks-to-the-left-of-a-red-block
            (<x> ^on <y>)
            (<y> ^left-of <z>)
            (<z> *color red)
            -->
            RHS
          ```
          ![betanode](http://ww1.sinaimg.cn/mw690/005xfSxkly1g2b8o426m5j30et0j1wiy.jpg)  
        - 假设有一个新的WME(B7 ^color red)被加入到working memory中，那么它最终会进入C3的alpha memory中，那么从图中可以看到最下面的beta节点将会被右激活。那么这个被右激活的节点将会检查左边的token中的内容是否有匹配将\<z>绑定到了B7的token。在没有索引的情况下，这种查找将会遍历所有内容。当然了这个过程可以通过添加索引加速（比如通过\<z>绑定的内容作为索引选项）。类似的，当一个新的token被加入到beta memory中，那么下个join 节点就会被左激活然后将会查找它的alphamemory中是否有匹配的项。同样的这个查找过程可以使用索引加速。
        - 一般来说，最好的索引方式就是hash表，而且已有论文证明RETE目前来说比其他手段比如二叉排序树等效果要好。但是，使用hash表索引有一个两难选项，就是对于hash表大小的选择。因为也许WMEs和tokens会变得很大，那么如果hash表很小，就会造成大量的hash冲突，这样的话，hash表的意义就会失去，无法提升效率。但如果hash表很大，那么会造成大量的内存浪费。我们可以动态管理哈希表的大小，但是这种方法再编码上造成了很大的复杂性。
        - 解决上述困境的方法是可以通过为所有alpha memory和beta memory构建一个大的hash表，而不是给每个节点构建一个hash表，这样的话，其实就是相当于所有节点共用这个hash表，由于其巨大性，可以减少hash冲突，又由于共享性，可以减少内存的浪费。（有点类似于非阻塞socket编程的输入输出缓冲区，一个大的hash表内部，动态的分为几个区域，内容少的memory分到较少的hash表内存，多的就分到多的。。）
        - 虽然如前面所说，给memory加索引可以加快匹配速度。但是也有2个缺点：第一就是每次更新memory的时候，都需要更新索引，所以会比较慢。第二就是会降低节点共享率，有些时候，不得不给重复的memory去重新构建，而无法共享（因为可能存储的内容一样，但是索引方式不一样）。例如，我们增加下列这个和上述差不多的production：
          ```
          (slightly-modified-version-of-previous-production
            (<x> ^<y>)           /*C1*/
            (<y> ^left-of <z>)   /*C2*/
            (<y> ^color red)     /*C3,but tests <y> instead of <z>*/
          -->
          ...RHS...
          )
          ```
        上述production中，如果前面的memory nodes没有索引，也即匹配的时候是通过遍历方法的，那么C1,C2节点就可以共享。如果是存在索引的，那么由于此时要对\<y>建立索引去匹配最佳，那么前面建立的索引可能不是这样的，所以就不能共用了。
        尽管上述两个问题的存在，但是hash memory的设计还是可以整体提升性能的。

        （注意，我们并不是每次都能找到合适的变量去作为索引的，例如以下条件：
          ```
          (<x> ^on <y>)   /*C1*/
          (<a> ^left-of <b>)   /*C2*/
          ```
        在这个例子里，C2中的变量和C1完全无关，所以找不到一个合适的变量作为索引，那么此时我们可以使用简单的非索引memory节点。
        - 现在来说第二个问题——token是如何表示的？这里会出现两种分歧，是用数组存储还是用列表存储。用数组的优点是显而易见的，数组迭代器是随机迭代器，通过下标i可以直接获取第i个元素，但是list则是需要遍历。
        - 但是数组存储方法会有很多冗余。如果\<w1,w2..wi>是匹配前i个condition的一个token，那么它的父节点必定存储了\<w1..wi-1>匹配前i-1个condition的token。所以子节点的token可以表示为<parent,wi>,这样就可以将token表示为列表，其中parent是指向父节点内匹配前i-1个condition的token的指针。可以看到如果用数组存储，匹配i个条件的token需要O(i)的空间，但是用链表只需要O(1).所以如果有C个条件，那么数组的存储需要的空间是O(C^2)，但是链表是O(C)。而且更多的空间开销还意味着更多的时间开销去填补这些空间。例如复制时间开销。
        ## 总结来说，memory节点的设计，可以用hash索引加快匹配速度，最好使用链表形式表示token。
  17. Alpha Memory Implementation
      这里用伪代码实现memory节点，为了方便起见，使用了链表表示，且不含有索引。
      - 首先是WME结构，WME已经约定用三元组表示：
        ```
        structure WME:
          fields:array[0..2]of symbol
        end
        ```
      - Alpha memory 存储了WMEs的列表和它的后继节点（Join 节点）的列表：
        ```
        structure alpha-memory:
          item:list of WME
          successors:list of rete-node
        end
        ```
      - 当一个WME通过了alpha网络的筛选进入了alpha memory，那么我们就简单的将它放入WME列表中，并通知后继节点列表中的所有节点。
        ```
        procedure alpha-memory-activation(node:alpha-memory,w:WME)
          insert w at the head of node.items
          for each child in node.successors do right-activation (child,w)
        end
  18. Beta Memory Implementation
      - 按照前面约定的用list方式代表token：
        ```
        structure token:
          parent:token{points to the higher token,for items 1...i-1}
          wme:WME{gives item i}
        end
        ```
      - beta memory node存储了它含有的tokens的列表和它的子节点列表。在给出beta memory节点结构之前，我们回忆到要确定beta节点到底是左激活还是右激活需要确定已经被激活的节点类型。所以我们必须可以去判断节点类型。我们可以用一种直接的方式，在结构中加入节点类型，这样根据节点内结构类型就可以知道这个节点的类型了。beta网络中的节点都是由下列形式表示：
      ```
      structure token:
        type:"beta-memory","join-node",or"p-node"
        children:list of rete-node
        parent:rete-node
        ...(variant part ----other data depending on node type)...
      end
      ```
      - 所有beta网络内的节点都有类型，子节点列表，父节点这些部分。因此，我们可以快速写出左激活函数和右激活函数。
      - 回到beta memory 节点。唯一额外内容就是beta memory存储的token。
      ```
      structure beta-memory:
        items:list of token
      end
      ```
      - 每当一个beta memory得到了一次新的匹配（由已存在的token和一些WMEs构成），我们都建立一个新的token，把他加入beta memory中，然后通知这个beta memory节点的子节点们：
      ```
      procedure beta-memory-left-activation(node: beta-memory,t: token,w: WME)
        new-token=allocate-memory()
        new-token.parent=t
        new-token.wme=w
        insert new-token at the head of node.items
        for each child in node.children do left-activation(child,new-token)
      end
      ```

