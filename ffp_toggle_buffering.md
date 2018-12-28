# ffp\_toggle\_buffering

方法声明在 ijkmedia/ijkplayer/ff_ffplay.h 中：

```
void ffp_toggle_buffering_l(FFPlayer *ffp, int start_buffering);
```

实现也在同文件中：

```
void ffp_toggle_buffering_l(FFPlayer *ffp, int buffering_on)
{
    if (!ffp->packet_buffering)
        return;

    VideoState *is = ffp->is;
    if (buffering_on && !is->buffering_on) {
        av_log(ffp, AV_LOG_DEBUG, "ffp_toggle_buffering_l: start\n");
        is->buffering_on = 1;
        stream_update_pause_l(ffp);
        if (is->seek_req) {
            is->seek_buffering = 1;
            ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_START, 1);
        } else {
            ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_START, 0);
        }
    } else if (!buffering_on && is->buffering_on){
        av_log(ffp, AV_LOG_DEBUG, "ffp_toggle_buffering_l: end\n");
        is->buffering_on = 0;
        stream_update_pause_l(ffp);
        if (is->seek_buffering) {
            is->seek_buffering = 0;
            ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_END, 1);
        } else {
            ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_END, 0);
        }
    }
}
```

```
if (!ffp->packet_buffering)
    return;
```

若不支持缓存，返回。

```
if (buffering_on && !is->buffering_on) {
	...
}
```

判断是开始还是停止，这里是当请求 ```buffering_on``` 但还未开始时的逻辑。

```
is->buffering_on = 1;
```

标记播放器状态 ```buffering_on``` 为 true。

```
stream_update_pause_l(ffp);
```

视频流更新暂停状态。[方法详解](stream_update_pause_l.md)

```
if (is->seek_req) {
	is->seek_buffering = 1;
	ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_START, 1);
}
```

如果有 seek 请求，标记播放器 ```seek_buffering``` 状态为 true，并发送 ```FFP_MSG_BUFFERING_START``` 通知信息。

```
else {
	ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_START, 0);
}
```

若没有 seek 请求，直接发送 ```FFP_MSG_BUFFERING_START``` 通知信息。

```
else if (!buffering_on && is->buffering_on) {
	...
}
```

这里是视频缓存暂停的逻辑。

```
is->buffering_on = 0;
```

标记播放器状态 ```buffering_on``` 为 false。

```
stream_update_pause_l(ffp);
```

更新视频流的暂停状态。

```
if (is->seek_buffering) {
    is->seek_buffering = 0;
    ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_END, 1);
}
```

如果视频正在 seek，标记 ```seek_buffering``` 为 false，之后发送 ```FFP_MSG_BUFFERING_END``` 消息。

```
else {
	ffp_notify_msg2(ffp, FFP_MSG_BUFFERING_END, 0);
}
```

若没有 seek，则直接发送 ```FFP_MSG_BUFFERING_END``` 消息。

[返回](ffp_start_from_l.md)