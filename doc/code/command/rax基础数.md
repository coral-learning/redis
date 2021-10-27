# 简介
## RAX叫做基数树（前缀压缩树），就是有相同前缀的字符串，其前缀可以作为一个公共的父节点

redis源码中对应的是rax.c和rax.h 源码中的说明：

```
假设要存三个字符串：foo, footer, foobar
这是一个没有压缩的结构

             (f) ""
               \
               (o) "f"
                 \
                 (o) "fo"
                   \
                 [t   b] "foo"
                 /     \
        "foot" (e)     (a) "foob"
               /         \
     "foote" (r)         (r) "fooba"
             /             \
   "footer" []             [] "foobar"

 我们进行一下压缩：
                 ["foo"] ""
                    |
                 [t   b] "foo"
                 /     \
       "foot" ("er")    ("ar") "foob"
                /          \
      "footer" []          [] "foobar"
redis中基本就是这个样子

如果我们再插入一个first:

                   (f) ""
                   /
                (i o) "f"
                /   \
   "firs"  ("rst")  (o) "fo"
             /        \
   "first" []       [t   b] "foo"
                    /     \
          "foot" ("er")    ("ar") "foob"
                   /          \
         "footer" []          [] "foobar"
那就变成了这个样子
  基本了解之后，来看一下基本概念
```
结构 看一下，一个节点是什么样的

![](https://img2020.cnblogs.com/blog/2160054/202009/2160054-20200919094648133-2024582986.png)
对应的结构体定义：
```
typedef struct raxNode {
    uint32_t iskey:1;     /Does this node contain a key? */
    uint32_t isnull:1;    /Associated value is NULL (don't store it). */
    uint32_t iscompr:1;   /Node is compressed. */
    uint32_t size:29;     /Number of children, or compressed string len. */
    unsigned char data[];
} raxNode;
```
一个node有5个部分

iskey:如果为1，那表示前面的字符串组成一个完整的Key
isnull:字符串是可以对应有一个value指针的，就像key-value的对应关系；这个value可以是NULL
iscompr:是否压缩节点
size: 该节点含有的字符串长度，注意，并不是data数组的长度

假设现在有一个节点存储了"abc"这3个字符：

![](https://img2020.cnblogs.com/blog/2160054/202009/2160054-20200919095657182-1993847876.png)

看图，因为有abc三个字符，所以size是3

前面三个iskey, isnull, 先不管

iscompr为0，表示非压缩节点

data中绿色的padding，不存放任何数据，其意义在于让数组的大小=指针大小的倍数，内存取整
padding后面的三个指针就是abc这三个字符分别的后继
这3个指针也是raxNode指针

如果这个节点是rax树中的一个节点，那我们可以猜想rax含有：xxxayyyy, xxxbzzzzz, xxxcttttt 这3个key字符串
xxx是共同前缀，后面yyyy,zzzz,ttttt就是分别的后缀

那压缩节点呢？

![](https://img2020.cnblogs.com/blog/2160054/202009/2160054-20200919100644161-130999007.png)

iscompr为1
后面就只有一个raxNode指针

如果是压缩节点，那表示共同的前缀abc，就如同开头的例子中的"foo"

## 插入分析

直接看代码的话，其实挺迷的
因此，我们从实际例子出发
观察一个rax树，从新建到插入数据的变化过程
理解了这个过程，再看代码就会容易很多
（假设指针大小为4字节）

刚新建
rax结构体表示rax树
raxNode结构体表示rax树中的节点
```
typedef struct rax {
    raxNode *head;  //指向新的raxNode
    uint64_t numele; // 0   完整的key的数量
    uint64_t numnodes;  // 1  raxNode的数量
} rax;

rax->head指向下面这个raxNode

typedef struct raxNode {
    uint32_t iskey:1;     /Does this node contain a key? */
    uint32_t isnull:1;    /Associated value is NULL (don't store it). */
    uint32_t iscompr:1;   /Node is compressed. */
    uint32_t size:29;     /Number of children, or compressed string len. */

    unsigned char data[];
} raxNode;
全是0，data没有
```
刚新建的时候，只有一个空节点

插入abcdefg

rax是可以保存key->value的，key是字符串，value任意
因此需要区分有没有value，raxNode->isnull为1表示没有value，为0表示有

rax->head会变成：
```
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 1
uint32_t size        // 7

    unsigned char data[];
    // |a|b|c|d|e|f|g| |ptr raxNode*|
    // 存入的数据需要分配的空间为指针大小（4字节）的倍数，所以会多一个字节
    // 最后是一个指针，指向下一个raxNode
    // 注意，size并不是data真正的长度
} raxNode;
可以看到，data中abcdefg这7个字符后面，还有1个字节的空白，这是为了凑够8字节，刚好是指针占用字节数的2倍

ptr raxNode*：

typedef struct raxNode {
uint32_t iskey       // 1 为1表示前面的组成一个key
uint32_t isnull      // 1 或 0
uint32_t iscompr     // 0
uint32_t size        // 0

    unsigned char data[];
    //如果没有value，那data没有；如果有，那data有一个指针的长度
} raxNode;
注意这个iskey，表示的是前面一串的节点组成一个完整的key，不包括当前节点

这时：
rax->numele=1, rax->numnodes=2

先看一下这个函数，这个函数把一个raxNode变为压缩节点

函数 raxCompressNode (raxNode *n, unsigned char *s, size_t len, raxNode **child):
参数：
child指针 新建Node raxNewNode

函数的执行内容：
n->data的大小变为：raxNode结构体长度+len数据长度+sizeof(raxNode*)，指针因为是压缩节点后面只有一个指针

n->iscompr = 1

n->size = len

s的数据内容复制到n

n的最后的指针为child(刚才新建的Node)

这个函数把n变成压缩节点，s的数据覆盖到n节点；n的最后的指针指向新创建的Node，传入的child指向这个新创建的Node

n的iscompr和size属性发生了变化，iskey和isnull没变

h即rax->head，需要变成压缩节点（调用raxCompressNode）
parentlink = rax->head的最后那个指针
rax->numnodes++

h指向下一个raxNode

如果data有，把h扩大void*

rax->numele++

h->key = 1
如果data没有，h->isnull=1；如果有data，h的最后一个指针指向data

return 1

插入abcxyz
演变过程：

现在rax有2个node，我们把它们叫做node1 和 node2

观察发现，rax当前有字符串abcdefg，和要插入的abcxyz，前3个字符是相同的

因此，新建一个node3 放前3个字符abc

node3
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 1
uint32_t size        // 3

    unsigned char data[];
    //内存对齐，虽然只有3个字符，实际占用了4个字节，最后有一个指针
    //指针指向node4
    // abc_ node*
} raxNode;
然后来到分界点，需要再新建一个node4

node4
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 0
uint32_t size        // 2

    unsigned char data[];
    //内存对齐，有2个字符d和x（按照字符升序排列），实际占用了4个字节，最后有2个指针
    // dx__ nodenode*
} raxNode;
因为这里分出了2条字符串，所以node4 有2个字符，作为2个引导字符，也有2个node指针，指向后续的字符串

先来看d这条分支，这代表的是原来的那个abcdefg，剩下的efg在下一个node，node5

node5
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 1
uint32_t size        // 3

    unsigned char data[];
    //内存对齐，有3个字符efg，实际占用了4个字节，最后有1个指针
    // efg_ node
} raxNode;
最后的这个指针指向原来那个node2，node2:

node2
typedef struct raxNode {
uint32_t iskey       // 1 为1表示前面的组成一个key
uint32_t isnull      // 1 或 0
uint32_t iscompr     // 0
uint32_t size        // 0

    unsigned char data[];
    //如果没有value，那data没有；如果有，那data有一个指针的长度
} raxNode;
现在原来的abcdefg，被拆分成了3个node(abc、d、efg)，加上原来的node1，一共4个node

回到分界点node4，第二条分支，以字符x 开始

新建node6，放yz

node6
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 1
uint32_t size        // 2

    unsigned char data[];
    //内存对齐，有2个字符yz，实际占用了4个字节，最后有1个指针
    // yz__ node
} raxNode;
再新建一个node7，和node2一样，作为结尾；node6最后的指针指向node7

node7
typedef struct raxNode {
uint32_t iskey       // 1 为1表示前面的组成一个key
uint32_t isnull      // 1 或 0
uint32_t iscompr     // 0
uint32_t size        // 0

    unsigned char data[];
    //如果没有value，那data没有；如果有，那data有一个指针的长度
} raxNode;
最后，rax->head = node3

原来的node1 释放

插入完成，图示:

node3(abc) -> node4(dx) d的分支-> node5(efg) -> node2()

＿＿＿＿＿＿＿＿＿＿＿＿x的分支-> node6(yz) -> node7()

插入abcxyzwhat
把node7 放入what，node7变成压缩节点

然后新建节点node8，node8->iskey=1，作为终止节点，和node2一样

node3(abc) -> node4(dx) d的分支-> node5(efg) -> node2()

＿＿＿＿＿＿＿＿＿＿＿＿x的分支-> node6(yz) -> node7(what) -> node8()

插入 ab
由于ab在第一个节点abc中已经存在，因此需要将node3拆成2个node

新建node9，装ab，这个node会成为rax->head

node9
typedef struct raxNode {
uint32_t iskey       // 为原来的node3->iskey
uint32_t isnull      // 为原来的node3->isnull
uint32_t iscompr     // 1
uint32_t size        // 2

    unsigned char data[];
    //ab__ nodevoid*(取决于node3有没有data)
} raxNode;
然后新建node10，node9后面是node10

node10
typedef struct raxNode {
uint32_t iskey       // 1，表明node9组成了key
uint32_t isnull      // 取决于插入ab时，有没有data
uint32_t iscompr     // 1
uint32_t size        // 1

    unsigned char data[];
    //c___ nodevoid*(取决于node3有没有data)
    //node*指向node4，后面保持一致
} raxNode;
node3释放

插入后结构：

node9(ab) -> node10(c) -> node4(dx) d的分支-> node5(efg) -> node2()

＿＿＿＿＿＿＿＿＿＿＿＿＿＿＿＿＿＿x的分支-> node6(yz) -> node7(what) -> node8()

插入abcx
看到node4已经有x 了，这时不需要新增节点

我们要做的是修改node6

node6原来的样子：

node6
typedef struct raxNode {
uint32_t iskey       // 0
uint32_t isnull      // 0
uint32_t iscompr     // 1
uint32_t size        // 2

    unsigned char data[];
    //内存对齐，有2个字符yz，实际占用了4个字节，最后有1个指针
    // yz__ node
} raxNode;
由于前面的组成key，node6->iskey=1，rax->numele++

如果data有，node6需要变大void*，isnull根据情况变化

插入就讲到这里
代码就不讲了，其实主要就是一些内存申请、位移操作，比较繁琐，但也不难