# stream\_open

方法实现在 ijkmedia/ijkplayer/ff_ffplay.c 中：

```
static VideoState *stream_open(FFPlayer *ffp, const char *filename, AVInputFormat *iformat)
{
    assert(!ffp->is);
    VideoState *is;

    is = av_mallocz(sizeof(VideoState));
    if (!is)
        return NULL;
    is->filename = av_strdup(filename);
    if (!is->filename)
        goto fail;
    is->iformat = iformat;
    is->ytop    = 0;
    is->xleft   = 0;
#if defined(__ANDROID__)
    if (ffp->soundtouch_enable) {
        is->handle = ijk_soundtouch_create();
    }
#endif

    /* start video display */
    if (frame_queue_init(&is->pictq, &is->videoq, ffp->pictq_size, 1) < 0)
        goto fail;
    if (frame_queue_init(&is->subpq, &is->subtitleq, SUBPICTURE_QUEUE_SIZE, 0) < 0)
        goto fail;
    if (frame_queue_init(&is->sampq, &is->audioq, SAMPLE_QUEUE_SIZE, 1) < 0)
        goto fail;

    if (packet_queue_init(&is->videoq) < 0 ||
        packet_queue_init(&is->audioq) < 0 ||
        packet_queue_init(&is->subtitleq) < 0)
        goto fail;

    if (!(is->continue_read_thread = SDL_CreateCond())) {
        av_log(NULL, AV_LOG_FATAL, "SDL_CreateCond(): %s\n", SDL_GetError());
        goto fail;
    }

    if (!(is->video_accurate_seek_cond = SDL_CreateCond())) {
        av_log(NULL, AV_LOG_FATAL, "SDL_CreateCond(): %s\n", SDL_GetError());
        ffp->enable_accurate_seek = 0;
    }

    if (!(is->audio_accurate_seek_cond = SDL_CreateCond())) {
        av_log(NULL, AV_LOG_FATAL, "SDL_CreateCond(): %s\n", SDL_GetError());
        ffp->enable_accurate_seek = 0;
    }

    init_clock(&is->vidclk, &is->videoq.serial);
    init_clock(&is->audclk, &is->audioq.serial);
    init_clock(&is->extclk, &is->extclk.serial);
    is->audio_clock_serial = -1;
    if (ffp->startup_volume < 0)
        av_log(NULL, AV_LOG_WARNING, "-volume=%d < 0, setting to 0\n", ffp->startup_volume);
    if (ffp->startup_volume > 100)
        av_log(NULL, AV_LOG_WARNING, "-volume=%d > 100, setting to 100\n", ffp->startup_volume);
    ffp->startup_volume = av_clip(ffp->startup_volume, 0, 100);
    ffp->startup_volume = av_clip(SDL_MIX_MAXVOLUME * ffp->startup_volume / 100, 0, SDL_MIX_MAXVOLUME);
    is->audio_volume = ffp->startup_volume;
    is->muted = 0;
    is->av_sync_type = ffp->av_sync_type;

    is->play_mutex = SDL_CreateMutex();
    is->accurate_seek_mutex = SDL_CreateMutex();
    ffp->is = is;
    is->pause_req = !ffp->start_on_prepared;

    is->video_refresh_tid = SDL_CreateThreadEx(&is->_video_refresh_tid, video_refresh_thread, ffp, "ff_vout");
    if (!is->video_refresh_tid) {
        av_freep(&ffp->is);
        return NULL;
    }

    is->initialized_decoder = 0;
    is->read_tid = SDL_CreateThreadEx(&is->_read_tid, read_thread, ffp, "ff_read");
    if (!is->read_tid) {
        av_log(NULL, AV_LOG_FATAL, "SDL_CreateThread(): %s\n", SDL_GetError());
        goto fail;
    }

    if (ffp->async_init_decoder && !ffp->video_disable && ffp->video_mime_type && strlen(ffp->video_mime_type) > 0
                    && ffp->mediacodec_default_name && strlen(ffp->mediacodec_default_name) > 0) {
        if (ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2) {
            decoder_init(&is->viddec, NULL, &is->videoq, is->continue_read_thread);
            ffp->node_vdec = ffpipeline_init_video_decoder(ffp->pipeline, ffp);
        }
    }
    is->initialized_decoder = 1;

    return is;
fail:
    is->initialized_decoder = 1;
    is->abort_request = true;
    if (is->video_refresh_tid)
        SDL_WaitThread(is->video_refresh_tid, NULL);
    stream_close(ffp);
    return NULL;
}
```

[返回](ijkplayer_main.md)