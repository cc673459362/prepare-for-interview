## txym

1. 用C函数写一个字符串模式匹配的，同一字母大小写相等，如果找不到返回NULL，找到了返回母串中该字串的第一个字符的位置。
    - 注意点：
        1. 大小写先统一。
        2. 找不到返回的是NULL，说明该函数返回值是一个指针。
        3. 有一个KMP算法。
```C
    char* findsub(const char* src,const char* sub){
        char* indexofsrc=src;
        char* indexofsub=sub;
        while(*indexofsrc!='\0'){
            *indexofsrc=tolower(*indexofsrc);//转为小写
            ++indexofsrc;
        }
        while(*indexofsrc!='\0'){
            *indexofsub=tolower(*indexofsub);//转为小写
            ++indexofsub;
        }
        while(*indexofsrc!='\0'&&*indexofsub!='\0'){  //父串和子串都没有匹配完
            if(*indexofsrc==*indexofsub){
                indexofsrc++;
                indexofsub++;
            }
            else{
                indexofsrc=indexofsrc-(indexofsub-sub)+1;
                indexofsub=sub;
            }
        }
        if(*indexofsub='\0') return indexofsrc-(indexofsub-sub);
        else{
            return NULL;
        }
    }
```  

2. struct和union的区别
    - struct会为结构体里面所有成员都分配空间，而union则只会分配一个里面最大的空间。
    - union的赋值会覆盖上一个内容。
      
3. struct和class的区别
    - struct可以包含成员函数，可以继承，可以多态，这些都和class一样。
    - struct默认访问控制是public，而class是private。（包括继承权限的默认也是如此。）
    - class还可以定义模板，类似于“typename”，而struct不能。
      
4. C++的多态的作用和实现方式
    - C++多态可以实现一个接口，多种方法。实现接口的重用。
    - C++多态通过虚函数实现。
    - 虚函数的机制是晚捆绑。
    - 晚捆绑通过编译器为含有虚函数的类在构造对象时，首先创建一个vptr，该指针指向这个类的虚函数表，虚函数表里存放了该类虚函数的实际地址。那么调用该对象虚函数的时候，就会在真正运行时，通过vpt指向的表里的地址去调用那个真正想要调用的函数，实现晚捆绑。
      
5. 字节对齐
    - struct字节对齐：首先内部，对每种类型都应该首先保证自己的字节对齐，比如int4个字节，那么这个空间应该保证4字节对齐，内部每种类型对齐后，对于整体struct，也应该保证对内部最大字节对齐的方式对齐，比如一个struct类里面最大的数据是double，里面还有char，int其他小的，那么这个struct自己也应该保证8字节对齐。
      
6. select函数的原理作用和缺点。
    - select函数实现IO多路复用，通过把文件描述符加入到select的文件描述符集里，调用select函数后，阻塞与select，每当其中文件描述符发生可响应的事件，就会返回这个fd。
    - 缺点：1.每次调用select，都会把fd_set从用户态拷贝为内核态，这个开销在fd_set很大时很大。    
            2. 每次调用select，内核都会遍历整个fd_set，开销很大。  
            3.fd_set很小，默认时1024.  

7. linux系统查看内存命令
    - cat /proc/meminfo 这个虚拟文件就行，里面显示了各种内存信息。
    ![cat /pro/meminfo](http://ww4.sinaimg.cn/large/005xfSxkly1g11frvfwanj30ko0dxwj9.jpg)
    - free 命令将/proc/meminfo里的信息概述出来。
    ![free](http://ww1.sinaimg.cn/large/005xfSxkly1g11ftm1t5oj30kq03cmy4.jpg)

8. 用gdb查看堆栈信息
    - **有待研究**

9. mysql有哪些引擎
    - 有MyISAM、Innodb、MEMORY等。
    - MyISAM支持表锁，INSERT和SELECT比较快，但是并发性没有Innodb好。
    - Innodb支持行锁，并发性更好。
    - **行锁的实现原理（有待研究）**


10. TCP、UDP区别
      TCP|UDP
      -|-
      TCP是面向连接的|UDP是不面向连接的
      TCP是可靠的，拥有确认重传、流量控制、拥塞控制等|UDP是不可靠的
      TCP是面向流的|UDP是面向数据包的
      TCP包头从20字节到60字节不等|UDP是8个字节包头
      TCP不支持多播|UDP支持多播
        
11. 堆的实现方式
    - 堆直接使用数组存储即可。因为堆是一个完全二叉树，用数组存足以保证其互相之间的关系。子节点n的[n/2]节点就是父节点了。
    - 堆的建立通过筛选法。就是从最后一个节点开始实行下面那个堆的调整的算法。
    - 堆的调整：把最后一个节点放到第一个来，也就是说现在根节点是不符合堆的性质的，但是下面都是符合的，那么假设是大根堆，就把这个根节点跟左右子树的根节点比较，如果有比他大，那么就和他交换，然后递归建立大根堆。
12. 快速排序
    - 设置一个哨兵，将哨兵与s[0]交换，之后设置前后两个游标，先从后往前，当游标所指的数字小于哨兵，那么就将哨兵和这个数字交换，然后从前往后，如果前游标所指数字大于哨兵，就把刚才后面游标的数字和现在前游标数字交换，知道左右游标相等。这个时候，游标左边都比哨兵小，右边都比哨兵大。然后对左右两边进行刚才算法的递归就完成了快排。  
