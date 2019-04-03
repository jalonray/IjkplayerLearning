# cache.c

文件位于 extra/ffmpeg/libavformat/cache.c 中。

先看两个头部的结构体定义：

```
typedef struct CacheEntry {
    int64_t logical_pos;
    int64_t physical_pos;
    int size;
} CacheEntry;

typedef struct Context {
    AVClass *class;
    int fd;
    struct AVTreeNode *root;
    int64_t logical_pos;
    int64_t cache_pos;
    int64_t inner_pos;
    int64_t end;
    int is_true_eof;
    URLContext *inner;
    int64_t cache_hit, cache_miss;
    int read_ahead_limit;
} Context;
```

```CacheEntry``` 定义了 cache 的单元类型。```Context``` 定义了 cache 的环境。

```
static int cmp(const void *key, const void *node)
{
    return FFDIFFSIGN(*(const int64_t *)key, ((const CacheEntry *) node)->logical_pos);
}
```

看下 ```FFDIFFSIGN``` 的定义：

```
/**
 * Comparator.
 * For two numerical expressions x and y, gives 1 if x > y, -1 if x < y, and 0
 * if x == y. This is useful for instance in a qsort comparator callback.
 * Furthermore, compilers are able to optimize this to branchless code, and
 * there is no risk of overflow with signed types.
 * As with many macros, this evaluates its argument multiple times, it thus
 * must not have a side-effect.
 */
#define FFDIFFSIGN(x,y) (((x)>(y)) - ((x)<(y)))
```

所以这个方法是比较 ```key``` 和 ```node``` 的大小。

```
static int cache_open(URLContext *h, const char *arg, int flags, AVDictionary **options)
{
    char *buffername;
    Context *c= h->priv_data;

    av_strstart(arg, "cache:", &arg);

    c->fd = avpriv_tempfile("ffcache", &buffername, 0, h);
    if (c->fd < 0){
        av_log(h, AV_LOG_ERROR, "Failed to create tempfile\n");
        return c->fd;
    }

    unlink(buffername);
    av_freep(&buffername);

    return ffurl_open_whitelist(&c->inner, arg, flags, &h->interrupt_callback,
                                options, h->protocol_whitelist, h->protocol_blacklist, h);
}
```

