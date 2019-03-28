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
    - Mysql的innodb行锁是通过给索引上的索引项加锁（或者对索引项之间的间隙加锁）实现的，和Oracle不同的是，Oracle是通过给相应的数据行加锁实现的。Innodb这种行锁意味着如果不是通过索引去检索数据，是不能使用行锁的，会使用表锁。
    - 

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
    ```C++
    //i,j表示的意思就是下标。
    string src = "abcccwdoghrlkwkndams,hr";
    string pattern = "abaabcac";
    int m = pattern.size();
    vector<int> next(m, 0);
    int ispattern(string src, string pattern) {
	    int n = src.size();
	    int m = pattern.size();
	    if (m > n || m == 0) return -1;
	    int i = 0;
	    int j =0;
	    while (i < n&&j < m) {
		    if (j == -1 || src[i] == pattern[j]) {
			    i++;
			    j++;
		    }
		    else {
			    j = ::next[j];
		    }
	    }
	    if (j>=m) return i - m ; 
	    else
		    return -1;
    }
    void getnext(string pattern, vector<int>& next) {
	    if (pattern.size() == 0) return;
	    next[0] = -1;
	    for (int i = 1; i<pattern.size(); i++) {
		    int k = i - 1 -1;     
		    int l = 1;                
		    while (pattern.substr(0, k+1/*  从0~k的子串的长度是k+1*/) != pattern.substr(l, k+1) && k+1>0) {
			    k--;
			    l++;
		    }
		    next[i] = k + 1;
	    }
    }

    int main() {
	    getnext(pattern, ::next);
	    int a = ispattern(src, pattern);
	    for (int i = 0; i < m ; i++) {
		    cout << ::next[i] << " ";
	    }
	    cout << endl;
	    cout << a;
    }
    ```
    

## 360ym 
   1. 写一个二分查找。
   ```C++
   //递归实现
   int binarysearch(int* a,int left,int right,int key){
       int mid=(left+right)/2;
       if(left>=right){
           return -1;
       }
       if(a[mid]>key){
           binarysearch(a,left,mid-1,key);
       }
       else if(a[mid]<key){
           binarysearch(mid+1,right,key);
       }
       else if(a[mid]==key){
           return mid;
       }
   }

   //非递归实现
   int binarysearch(int* a,int left,int right,int key){
       while(left<right){
           int mid=(left+right)/2;
           if(a[mid]>key){
               right=mid-1;
           }
           else if(a[mid]<key){
               left=mid+1;
           }
           else{
               return mid;
           }
       }
       return -1;
   }


   ```
2. 有一个数组，找出其中符合下列要求的所有元素：该元素左边的元素都比他小，该元素右边的元素都比他大。
   - 这个数的特点就是他是他左边的最大值，右边数组的最小值。那么我们可以先从右往左把最小值记录下来。然后再从左往右判断这个当前元素的最大值，如果和当前元素的右边最小值相等，则该元素符合要求。
  ```C++
  vector<int> getindex(vector<int> a){
      int num=a.size();
      vector<int> res;
      vector<int> minm;
      int least=INT_MAX;
      for(int i=num-1;i>=0;i--){
          least=min(a[i],least);
          minm.push_back(least);
      }
      int largest=INT_MIN;
      for(int i=0;i<num;i++){
          largest=max(largest,a[i]);
          if(lagest==least[i]){
              res.push_back(i);
          }
      }
      return res;
  }  
  ```
3. Dijkstra(迪杰斯特拉)算法。
   - 算法思想：设G=(V,E)是一个**带权有向图**，把图中顶点集合V分成两组，第一组为已求出最短路径的顶点集合（用S表示，初始时S中只有一个源点，以后每求得一条最短路径 , 就将加入到集合S中，直到全部顶点都加入到S中，算法就结束了），第二组为其余未确定最短路径的顶点集合（用U表示），按最短路径长度的递增次序依次把第二组的顶点加入S中。在加入的过程中，总保持从源点v到S中各顶点的最短路径长度不大于从源点v到U中任何顶点的最短路径长度。此外，每个顶点对应一个距离，S中的顶点的距离就是从v到此顶点的最短路径长度，U中的顶点的距离，是从v到此顶点只包括S中的顶点为中间顶点的当前最短路径长度。
  ![图]()

4. 求两个有序数组的并集（有序数组内有重复元素）不能用unique。
   ```C++
   vector<int> getunionset(vector<int> a,vector<int> b){
       int n=a.size();
       int m=b.size();
       int i=0;
       int j=0;
       vector<int> res;
       while(i<n&&j<m){
           if(a[i]==b[j]){
               if (res.size() == 0) {
				    res.push_back(a[i]);
				    i++;
				    j++;
			    }
               else if(res.back()!=a[i]){
                   res.push_back(a[i]);
                   i++;
                   j++;
               }
               else{
                   i++;j++;
               }
           }
           else if(a[i]<b[j]){
                if (res.size() == 0) {
				    res.push_back(a[i]);
				    i++;
				
			    }
                else if(res.back()!=a[i]){
                   res.push_back(a[i]);
                   i++;
               }
               else{
                   i++;
               }
           }
           else{
                if (res.size() == 0) {
				    res.push_back(b[j]);
				    j++;
				
			    }
               else if(res.back()!=b[j]){
                   res.push_back(b[j]);
                   j++;
               }
               else{
                   j++;
               }
           }
       }
       if(i==n){
           while(j<m){
               if(res.back()!=b[j]){
                   res.push_back(b[j]);
                   j++;
               }
               else{
                   j++;
               }
           }
       }
       else{
           while(i<n){
               if(res.back()!=a[i]){
                   res.push_back(a[i]);
                   i++;
               }
               else{
                   i++;
               }
           }
       }
       return res;
   }
   ```

5. 在Linux系统下的内存泄漏检查方法。
   - 用Valgrind
   - 最简单的就是 valgrind [proname]。但是记得程序编译的时候加上-g 和-O0.
6. 调试运行中的程序。
    - 可以先用ps -a查看进程PID。
    - 然后用 gdb attach [PID].就可以调试这个程序了。



7. 有一个单链表1->2->3->...->n-1->n;把他修改成1->n->2->n-1->3->n-2...-.
   ```C++
   //首先将该链表一分为二，然后将后半部分翻转，然后合并
   ListNode* gethallist(ListNode* head){
       ListNode* fast=head;
       ListNode* slow=head;
       while(fast!=NULLfast->next!=NULL){
           fast=fast->next->next;
           slow=slow->next;
       }
       ListNode* sechead=slow->next;
       slow->next=NULL;
       sechead=reverseList(sechead);
       ListNode* res=merge(head,sechead)
       return res;
   }
   ListNode* reverseList(ListNode* head){
       if(head==NULL){
           return NULL;
       }
       ListNode* temp=New ListNode();
       temp->next=head;
       ListNode* pre=head;
       ListNode* cur=pre->next;
       while(cur){
           pre->next=cur->next;
           cur->next=temp->next;
           temp->next=cur;
           cur=pre->next;
       }
       return temp->next;
   }
    ListNode* merge(ListNode* head,ListNode* sechead) {
	    if (sechead == NULL) return head;
	    if (head == NULL) return sechead;
	    ListNode* res = head;
	    ListNode * p1 = head;
	    ListNode *p2 = sechead;
	    while (p1->next&&p2->next) {
		    p1 = p1->next;
		    p2 = p2->next;
		    head->next = sechead;
		    sechead->next = p1;
		    head = p1;
		    sechead = p2;
	    }
	    if (p1->next) {
		    p1 = p1->next;
		    head->next = sechead;
		    sechead->next = p1;
	    }
	    if (p2->next) {
		    p2 = p2->next;
		    head->next = sechead;
	    }
		    return res;
    }
8. 如何确定远端服务器的端口是否可用。
 - 用telnet功能。telnet是TCP\IP协议族的一员，通过telnet功能，本机可以登录远端服务器，对远端服务器进行远程控制。
 - telnet和ping的区别再去，telnet要求端口，是基于TCP的，而ping只要求IP地址，使用的是网络层的ICMP协议。

9. 函数指针和指针函数
    - 函数指针是指指向函数的指针，例如 int (\*func)(int a,int b), 这个声明阅读顺序是先→后←，然后继续。func先→遇到）就←，遇到*，说明他是一个指针，再往→发现是一个形参表，往左发现是一个返回值。所以这个func是一个函数指针。
    - 指针函数是指返回类型是一个指针。（见13.）
10. 常量指针和指针常量
    - 常量指针指的是指针所指的内容是一个常量，不允许被改变内容。
    - 指针常量指的是指针本身是一个常量，不允许改变指向。
11. 虚析构
    - 虚析构为了能够让类在继承中能够完全释放对象的空间。继承类调用析构函数如果不是虚析构函数，则不一定会调用自己类的析构函数而很有可能去调用基函数的析构函数造成只析构了基类部分空间，造成空间的大量泄漏。
12. 如何理解TCP是面向流的UDP是面向数据报的
    - TCP把应用程序交付下来的数据报看成是一串连续的字节流，它会将这些包存放于自己的发送缓存中，具体每次发送多大长度的字节流出去看网络情况和发送窗口大小而定。但是UDP对应用层交付的数据报则不做处理，如果这个包过大，超过了UDP的发送缓冲大小，则直接丢弃。如果没超过，但是也很大，可能会发生IP分片，降低效率。如果太短，则传输效率也很低。
    - UDP面向报文是因为他就加了个头，转交给了下一层，所以基本不做什么工作。 TCP面向流是由于他要做的很多工作决定了他需要把他当作字节流去对待。（例如流量控制、防止IP分片、拥塞控制、序列前后的保证）。
13. 返回指针有什么要注意的
    - 要注意返回的指针要分配在堆中，不要再栈中，否则函数出栈这个指针所指的对象就被释放了，就造成了指针的悬垂。可以用new 或者static解决。
14. IP分片（）
    - 首先是下层数据链路层有一个MTU的规定，一般是1500，也就是说如果我IP层下交的报文太长，长于MTU，那么应该先切片，然后再传输。
    - 一般IP曾也有自己的最长报文的规定，一般是576字节。当上层给的报文长度超过这个长度的时候，IP曾就直接给他分片了。
    - 一般TCP有一个可选择项里已经规定了TCP下交的报文最长长度MSS，一般是556字节，这样加上IP报文固定头长，也就是576，就一般不会发生IP切片。
    - 所以说，容易发生IP切片的应该是UDP下发给IP的了，因为UDP不会对应用层下发的报文做任何处理。很容易超长。
15. 主键和外键
    - 主键可以唯一区别表中的每一行。主键自动创建一个unique index。
    - 外键可以关联两张表，并使得这两张表的数据的一致性和完整性，并实现一些级联操作。
16. B树和B+树  以及文件实现方式。（外键列必须建立索引）
17. 可重入锁
    - 也就是递归锁，对于同一线程，该锁是可以递归加锁的，可以重入的，对于多线程则是和普通的互斥锁一样。在Linux下可重入锁需要在互斥锁加一个PTHREAD_MUTEX_RECURSIVE属性来说明他是递归锁。
19. 如何查看SQL语句中使用的索引
    - 在SQL语句前加explain，
20. 如何查看一张表里的索引
    - show index from table_name;
21. 