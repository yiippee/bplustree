# B+Tree
A minimal B+Tree implementation for millions (even billions) of key-value storage based on Posix.

## Branch
[in-memory](https://github.com/begeekmyfriend/bplustree/tree/in-memory) for learning and debugging.

## Demo
```shell
./demo_build.sh
```

## Code Coverage Test

**Note:** You need to delete the existing `/tmp/coverage.index` first for this testing!

```shell
./coverage_build.sh
```
作者：我的上铺叫路遥
链接：https://www.zhihu.com/question/269033066/answer/347585034
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

谢邀！这个问题还是正合我意，因为我自己手把手亲自写了一个B+树最小demo，1K行的C代码，至今已有1K颗星，应该说代码得到了大众认可begeekmyfriend/bplustree你问我怎么实现磁盘存储？我们还是先来看头文件，这里有核心数据结构的描述和接口签名：其中bplus_tree相当于一个handler，由调用接口的用户掌握。里面重要东西包括，文件描述符、节点缓存、根节点位置以及空洞节点链表。
```
typedef struct free_block {
        struct list_head link;
        off_t offset;
} free_block;

struct bplus_tree {
        char *caches;
        int used[MIN_CACHE_NUM];
        char filename[1024];
        int fd;
        int level;
        off_t root;
        off_t file_size;
        struct list_head free_blocks;
};
```
文件描述符表明我的B+树是基于Posix接口实现，也就是说，最终面向用户磁盘存储形态是Unix文件，而不是裸机存储。节点缓存是以内存缓存磁盘存储的节点，将磁盘存储的节点读取到内存，以便改动更新后，写回磁盘。实际上MIN_CACHE_NUM为5，也就是说我们最多需要五个节点大小的缓存足矣。根节点位置是用来是B+树的搜索起点，它的值是文件中的偏移，注意根节点并非一定是文件最开始的地方，因为增删所以经常有变动。空洞是指上次删除节点的标记，可用于下次新插入节点的位置，优先于追加。除此之外，还有一些参数，比如节点大小block size（默认4KB）、非叶子节点的分支数目order、叶子节点的分支数目entries。再来是节点数据结构。可以看到，我们用struct bplusnode表示通用节点，它有四个off_t成员，分别是自身位置、父节点位置和左右兄弟位置，位置表示文件中的偏移。再来是类型（叶子or非叶子），以及孩子数目，对于非叶子节点就是order的值，它的数目比key多一个，因为在增删中需要父子节点旋转；对于叶子节点，就是entry的值，同key的数目一样。
```
typedef struct bplus_node {
        off_t self;
        off_t parent;
        off_t prev;
        off_t next;
        int type;
        /* If leaf node, it specifies  count of entries,
         * if non-leaf node, it specifies count of children(branches) */
        int children;
} bplus_node;

/*
struct bplus_non_leaf {
        off_t self;
        off_t parent;
        off_t prev;
        off_t next;
        int type;
        int children;
        key_t key[BPLUS_MAX_ORDER - 1];
        off_t sub_ptr[BPLUS_MAX_ORDER];
};
struct bplus_leaf {
        off_t self;
        off_t parent;
        off_t prev;
        off_t next;
        int type;
        int entries;
        key_t key[BPLUS_MAX_ENTRIES];
        long data[BPLUS_MAX_ENTRIES];
};
*/
```
在源码里我们只用到通用结构bplus_node，至于bplus_non_leaf和bplus_leaf是隐含的，在后面分析源文件时再说。未完待续……
