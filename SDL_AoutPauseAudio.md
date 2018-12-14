# SDL\_AoutPauseAudio

方法声明在 ijkmedia/ijksdl/ijksdl_aout.h 中：

```
typedef struct SDL_Aout SDL_Aout;
struct SDL_Aout {
    SDL_mutex *mutex;
    double     minimal_latency_seconds;

    SDL_Class       *opaque_class;
    SDL_Aout_Opaque *opaque;
    void (*free_l)(SDL_Aout *vout);
    int (*open_audio)(SDL_Aout *aout, const SDL_AudioSpec *desired, SDL_AudioSpec *obtained);
    void (*pause_audio)(SDL_Aout *aout, int pause_on);
    void (*flush_audio)(SDL_Aout *aout);
    void (*set_volume)(SDL_Aout *aout, float left, float right);
    void (*close_audio)(SDL_Aout *aout);

    double (*func_get_latency_seconds)(SDL_Aout *aout);
    void   (*func_set_default_latency_seconds)(SDL_Aout *aout, double latency);

    // optional
    void   (*func_set_playback_rate)(SDL_Aout *aout, float playbackRate);
    void   (*func_set_playback_volume)(SDL_Aout *aout, float playbackVolume);
    int    (*func_get_audio_persecond_callbacks)(SDL_Aout *aout);

    // Android only
    int    (*func_get_audio_session_id)(SDL_Aout *aout);
};

void SDL_AoutPauseAudio(SDL_Aout *aout, int pause_on);
```

实现在 ijkmedia/ijksdl/ijksdl_aout.c 中：

```
void SDL_AoutPauseAudio(SDL_Aout *aout, int pause_on)
{
    if (aout && aout->pause_audio)
        aout->pause_audio(aout, pause_on);
}
```

[返回](ffp_stop_l.md)