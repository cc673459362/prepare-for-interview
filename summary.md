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
    - 1. 首先，在编译的时候加上 -g，将调试信息加入进去。
    - 2. 然后用 gdb （程序名）开始调试。用list可以让gdb打出代码来看。
    - 3. 调试首先加入几个断点：用 break （行数/函数名等）加入断点，用info break命令可以查看当前断点。同样用 clear （。。）就可以删除断点。
    - 4. run  将程序运行起来，程序会运行到第一个断点把控制权交还给用户。
    - 5. next  就是一步步调试程序，和stepi不同的是，step当遇到程序调用的时候，会进入该调用程序，而next不会。
    - 6. 用info frame可以查看当前栈帧信息。 info registers查看当前寄存器信息。使用 backtrace 或者 bt 可以打印出当前栈的信息。
    - 7. print （变量名或者表达式）可以打印出当前该变量或表达式的内容。
    - 8. kill  和run相反，停止程序
    - 9. quit 推出gdb调试。

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

##txem  
1. 有三个字符串a,b,c，从a字符串找出b的串，并用c代替。（大小写相等）。
    - example：a="abcfabcee";b="abc";c="dd",那么最终返回字符串"ddfddee"。

    - 思考：首先，是一个字符串匹配，那么有暴力的O（m*n）解法和KMP解法，但是只有十五分钟，来不及想那么多，暴力匹配。然后就是替换，但是由于要替换所有找到的匹配，而不仅仅是替换掉第一个找到的。所以需要递归。先找一个，把他替换了，然后对之后的字符串递归这个操作。那么程序就是：
     ```C++
     int findstring(string a,string b){
         int n=a.size();
         int m=b.size();
         int i=0;j=0;
         while(i<n&&j<m){
             if(a[i]==b[j]||abs(a[i]-b[j])==32){
                 i++;
                 j++;
             }
             else{
                 i=i-j+1;
                 j=0;
             }
         }
         if(j==m){
             return i-j+1;
         }
         else{
             return -1;
         }
     }

     string replacestring(string a,string b,string c){
         int l1=a.size();
         int l2=b.size();
         if(l1<l2) return a;
         string res;
         int k=findstring(a,b);
         if(k==-1){
             return a;
         }
         else{
             res+=a.substr(0,k-1);
             res+=b;
             res+=replacestring(a.substr(k-1+l2,l1-l2-k+1),b,c);
             return res;
         }
     }  
     ```  
     - 上面有个问题没解决就是如果字符里有非字母的话，应该还是会有问题。

13. KMP算法
    - KMP算法中，对于字符串匹配，总结出一个规律，就是子串有时候根本不需要回去那么多直接回到最开头。而母串也根本不需要回溯。为什么呢？因为子串自己的结构其实就暗示了母串已经匹配过的结构，所以从自己出发就可以知道那些已经匹配过的字符是什么，就不用再去匹配一次了。所以KMP算法的计划是：
      - 1. 母串的匹配位置i不用回溯。
      - 2. 子串的匹配位置j适量回溯。
    - 那么如何确定j应该回溯到哪呢？这个问题前面就说了，根据子串自己其实就可以确定，跟母串无关。如何确定？
    - 假设子串是abaabcac，那么j是1，2，3，4，5，6，7，8.假设现在在j=6的地方失去了匹配，那么也就是说abaab这俩是已经匹配的，那么此时j应该怎么回溯呢？我们发现abaab的前缀ab和后缀ab相等，那么我们把这个子串往后移，可以把前面的ab放到后面的ab处就行了，这样前两个ab就已经匹配了，直接从j=3的a开始再次和母串匹配就行了。所以j=6的next[j]=3。从中我们发现，要去计算j的next[j]的值，只需要去观察在j之前的子串的前缀和后缀相等的长度（假设为k），有多长的前后缀相等（k），那么这个长度就不用再次匹配了，直接从k+1的那个位置开始再次开启匹配过程即可。所以next[j]=k+1；（k是指j位置之前的串的前后缀相等长度。）
    - 知道了next[j]之后，KMP就很简单了，和前面暴力匹配一样，相等就i++，j++。只不过不想等的时候，i不用动，j=next[j]（-1）就行啦！