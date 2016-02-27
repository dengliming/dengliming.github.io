title: redis之sds动态字符串
date: 2015-07-06 23:30:51
tags:
- redis
- sds
categories: redis

---

# redis之sds动态字符串

------
## redis定义
redis源码中sds的结构是这样的：
```cpp
 /*
 * 保存字符串对象的结构
 */
struct sdshdr {

    // buf 中已占用空间的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 数据空间
    char buf[];
};
```
buf最后一个字节保存了空字符'\0'遵循c字符串以空字符结尾的惯例，保存空字符的1字节空间是不记在len里面

## api之sdslen
```cpp
/*
 * 返回 sds 实际保存的字符串的长度
 *
 * T = O(1)
 */
static inline size_t sdslen(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));
    return sh->len;
}
```
其中要理解这段代码struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));

需要回顾下c的一些基础知识：
```cpp
typedef struct Node {
    int len;
    char str[];
} Node;
sizeof(char*) = 4
sizeof(Node*) = 4
sizeof(Node2) = 4
```
前两个是指针所占的字节是由系统的寻址能力有关，以上是32位的，后面那个
int len占4个字节，str[]暂未分配内存

然后看下sds初始化方法：
```cpp
sds sdsnewlen(const void *init, size_t initlen) {

    struct sdshdr *sh;

    // 根据是否有初始化内容，选择适当的内存分配方式
    // T = O(N)
    if (init) {
        // zmalloc 不初始化所分配的内存
        sh = zmalloc(sizeof(struct sdshdr)+initlen+1);
    } else {
        // zcalloc 将分配的内存全部初始化为 0
        sh = zcalloc(sizeof(struct sdshdr)+initlen+1);
    }

    // 内存分配失败，返回
    if (sh == NULL) return NULL;

    // 设置初始化长度
    sh->len = initlen;
    // 新 sds 不预留任何空间
    sh->free = 0;
    // 如果有指定初始化内容，将它们复制到 sdshdr 的 buf 中
    // T = O(N)
    if (initlen && init)
        memcpy(sh->buf, init, initlen);
    // 以 \0 结尾
    sh->buf[initlen] = '\0';

    // 返回 buf 部分，而不是整个 sdshdr
    return (char*)sh->buf;
}
```

这里面返回sds的地址只返回了buf部分，所以前面如果我们需要获取sds的长度len需要减去(sizeof(struct sdshdr))个字节

## api之sdsfree
释放给定的sds
```cpp
void sdsfree(sds s) {
    if (s == NULL) return;
    zfree(s-sizeof(struct sdshdr));
}
```

## api之sdsclear
在不释放 SDS 的字符串空间的情况下，重置 SDS 所保存的字符串为空字符串。T = O(1)

```cpp
void sdsclear(sds s) {

    // 取出 sdshdr
    struct sdshdr *sh = (void*) (s-(sizeof(struct sdshdr)));

    // 重新计算属性
    sh->free += sh->len;
    sh->len = 0;

    // 将结束符放到最前面（相当于惰性地删除 buf 中的内容）
    sh->buf[0] = '\0';
}
```
之所以sh->free += sh->len是因为redis的惰性空间释放，也就是当sds缩短时程序并没有立即回收缩短的字节，而是使用free属性记录起来，等待将来的使用，这样的好处就是下一次扩展sds时，就可以直接使用这些未使用的空间，不需要重新分配内存。

## api之sdscat

```cpp
sds sdscat(sds s, const char *t) {
    return sdscatlen(s, t, strlen(t));
}

sds sdscatlen(sds s, const void *t, size_t len) {
    
    struct sdshdr *sh;
    
    // 原有字符串长度
    size_t curlen = sdslen(s);

    // 扩展 sds 空间
    // T = O(N)
    s = sdsMakeRoomFor(s,len);

    // 内存不足？直接返回
    if (s == NULL) return NULL;

    // 复制 t 中的内容到字符串后部
    // T = O(N)
    sh = (void*) (s-(sizeof(struct sdshdr)));
    memcpy(s+curlen, t, len);

    // 更新属性
    sh->len = curlen+len;
    sh->free = sh->free-len;

    // 添加新结尾符号
    s[curlen+len] = '\0';

    // 返回新 sds
    return s;
}
```

参考：
> * [redis设计与实现][1]
> * [注释的redis源码][2]



  [1]: http://redisbook.com/
  [2]: https://github.com/huangz1990/redis-3.0-annotated