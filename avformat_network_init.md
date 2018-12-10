# avformat\_network\_init

```avformat_network_init()``` 的定义在 ijkplayer-x86_64/src/main/jni/ffmpeg/include/libavformat/avformat.h 中：

```
/**
 * @defgroup libavf libavformat
 * I/O and Muxing/Demuxing Library
 *
 * Libavformat (lavf) is a library for dealing with various media container
 * formats. Its main two purposes are demuxing - i.e. splitting a media file
 * into component streams, and the reverse process of muxing - writing supplied
 * data in a specified container format. It also has an @ref lavf_io
 * "I/O module" which supports a number of protocols for accessing the data (e.g.
 * file, tcp, http and others). Before using lavf, you need to call
 * av_register_all() to register all compiled muxers, demuxers and protocols.
 * Unless you are absolutely sure you won't use libavformat's network
 * capabilities, you should also call avformat_network_init().
 * ...
 */
 
 /**
 * Do global initialization of network components. This is optional,
 * but recommended, since it avoids the overhead of implicitly
 * doing the setup for each session.
 *
 * Calling this function will become mandatory if using network
 * protocols at some major version bump.
 */
int avformat_network_init(void);

/**
 * Undo the initialization done by avformat_network_init.
 */
int avformat_network_deinit(void);
```

实现在 extra/ffmpeg/libavformat/utils.c 中：

```
int avformat_network_init(void)
{
#if CONFIG_NETWORK
    int ret;
    ff_network_inited_globally = 1;
    if ((ret = ff_network_init()) < 0)
        return ret;
    if ((ret = ff_tls_init()) < 0)
        return ret;
#endif
    return 0;
}
```

```ff_network_init()``` 和 ```ff_tls_init()``` 两个方法均在 extra/ffmpeg/libavformat/network.c 中：

```
int ff_tls_init(void)
{
#if CONFIG_TLS_OPENSSL_PROTOCOL
    int ret;
    if ((ret = ff_openssl_init()) < 0)
        return ret;
#endif
#if CONFIG_TLS_GNUTLS_PROTOCOL
    ff_gnutls_init();
#endif
    return 0;
}

int ff_network_init(void)
{
#if HAVE_WINSOCK2_H
    WSADATA wsaData;
#endif

    if (!ff_network_inited_globally)
        av_log(NULL, AV_LOG_WARNING, "Using network protocols without global "
                                     "network initialization. Please use "
                                     "avformat_network_init(), this will "
                                     "become mandatory later.\n");
#if HAVE_WINSOCK2_H
    if (WSAStartup(MAKEWORD(1,1), &wsaData))
        return 0;
#endif
    return 1;
}
```

可见分为两类初始化方法，要么是 [tls](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)，要么是 [WSA](https://www.cisco.com/c/en/us/products/security/web-security-appliance/index.html)，这是两种网络加密方式。

[返回](ijkplayer_main.md)