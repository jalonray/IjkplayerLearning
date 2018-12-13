# ijkmp\_set\_data\_source

方法声明在 ijkmedia/ijkplayer/ijkplayer.h 中：

```
int ijkmp_set_data_source(IjkMediaPlayer *mp, const char *url);
```

实现在 ijkmedia/ijkplayer/ijkplayer.c 中：

```
#define MP_RET_IF_FAILED(ret) \
    do { \
        int retval = ret; \
        if (retval != 0) return (retval); \
    } while(0)

#define MPST_RET_IF_EQ_INT(real, expected, errcode) \
    do { \
        if ((real) == (expected)) return (errcode); \
    } while(0)

#define MPST_RET_IF_EQ(real, expected) \
     MPST_RET_IF_EQ_INT(real, expected, EIJK_INVALID_STATE)
     
int ijkmp_set_data_source(IjkMediaPlayer *mp, const char *url)
{
    assert(mp);
    assert(url);
    MPTRACE("ijkmp_set_data_source(url=\"%s\")\n", url);
    pthread_mutex_lock(&mp->mutex);
    int retval = ijkmp_set_data_source_l(mp, url);
    pthread_mutex_unlock(&mp->mutex);
    MPTRACE("ijkmp_set_data_source(url=\"%s\")=%d\n", url, retval);
    return retval;
}

static int ijkmp_set_data_source_l(IjkMediaPlayer *mp, const char *url)
{
    assert(mp);
    assert(url);

    // MPST_RET_IF_EQ(mp->mp_state, MP_STATE_IDLE);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_INITIALIZED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ASYNC_PREPARING);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PREPARED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STARTED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_PAUSED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_COMPLETED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_STOPPED);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_ERROR);
    MPST_RET_IF_EQ(mp->mp_state, MP_STATE_END);

    freep((void**)&mp->data_source);
    mp->data_source = strdup(url);
    if (!mp->data_source)
        return EIJK_OUT_OF_MEMORY;

    ijkmp_change_state_l(mp, MP_STATE_INITIALIZED);
    return 0;
}

void ijkmp_change_state_l(IjkMediaPlayer *mp, int new_state)
{
    mp->mp_state = new_state;
    ffp_notify_msg1(mp->ffplayer, FFP_MSG_PLAYBACK_STATE_CHANGED);
}

```

分析一下，先 ```pthread_mutex_lock(&mp->mutex);``` 上锁，再 ```int retval = ijkmp_set_data_source_l(mp, url);``` 设置 dataSource，可以看到里面的实现是对当前的 mp_state  作状态判别，在设置 dataSource 时，Ijkplayer 的 mp_state 应当是 MP_STATE_IDLE。之后清空原有的 data_source 内存，再将传入的 url 拷贝至 data_source。接着是溢出判断。最后发送状态更新至 MP_STATE_INITIALIZED 的通知。下面 ```pthread_mutex_unlock(&mp->mutex);``` 释放锁，最后 ```return retval;``` 返回操作结果，若成功则为 0，状态错误是 ```EIJK_INVALID_STATE```，溢出是 ```EIJK_OUT_OF_MEMORY```。

[返回](ijkplayer_main.md)