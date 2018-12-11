# ijkmp\_global\_set\_inject\_callback

该方法声明在 ijkplayer.h 中，实现在 ijkplayer.c 中：

```
void ijkmp_global_set_inject_callback(ijk_inject_callback cb)
{
    ffp_global_set_inject_callback(cb);
}
```

```void ffp_global_set_inject_callback(ijk_inject_callback cb);``` 方法声明在 ff_ffplay.h 中，实现在 ff_ffplay.c 中：

```
static ijk_inject_callback s_inject_callback;
void ffp_global_set_inject_callback(ijk_inject_callback cb)
{
    s_inject_callback = cb;
}
```

```s_inject_callback``` 会有另一个方法，封装它的调用：

```
int inject_callback(void *opaque, int type, void *data, size_t data_size)
{
    if (s_inject_callback)
        return s_inject_callback(opaque, type, data, data_size);
    return 0;
}
```

调用则只有一处：

```
static int app_func_event(AVApplicationContext *h, int message ,void *data, size_t size)
{
    if (!h || !h->opaque || !data)
        return 0;

    FFPlayer *ffp = (FFPlayer *)h->opaque;
    if (!ffp->inject_opaque)
        return 0;
    if (message == AVAPP_EVENT_IO_TRAFFIC && sizeof(AVAppIOTraffic) == size) {
        AVAppIOTraffic *event = (AVAppIOTraffic *)(intptr_t)data;
        if (event->bytes > 0) {
            ffp->stat.byte_count += event->bytes;
            SDL_SpeedSampler2Add(&ffp->stat.tcp_read_sampler, event->bytes);
        }
    } else if (message == AVAPP_EVENT_ASYNC_STATISTIC && sizeof(AVAppAsyncStatistic) == size) {
        AVAppAsyncStatistic *statistic =  (AVAppAsyncStatistic *) (intptr_t)data;
        ffp->stat.buf_backwards = statistic->buf_backwards;
        ffp->stat.buf_forwards = statistic->buf_forwards;
        ffp->stat.buf_capacity = statistic->buf_capacity;
    }
    return inject_callback(ffp->inject_opaque, message , data, size);
}
```

最后是下面的代码：

```
void *ffp_set_inject_opaque(FFPlayer *ffp, void *opaque)
{
    if (!ffp)
        return NULL;
    void *prev_weak_thiz = ffp->inject_opaque;
    ffp->inject_opaque = opaque;

    av_application_closep(&ffp->app_ctx);
    av_application_open(&ffp->app_ctx, ffp);
    ffp_set_option_int(ffp, FFP_OPT_CATEGORY_FORMAT, "ijkapplication", (int64_t)(intptr_t)ffp->app_ctx);

    ffp->app_ctx->func_on_app_event = app_func_event;
    return prev_weak_thiz;
}
```

将之赋予类型为 FFPlayer 的 app_ctx->func_on_app_event 上，在对应 app_event 发生变化时便可及时获取回调。

其中 ```ijk_inject_callback``` 回调定义在 ff_ffinc.h 中：

```
typedef int (*ijk_inject_callback)(void *opaque, int type, void *data, size_t data_size);
```

[返回](ijkplayer_main.md)