# JNI\_CHECK\_GOTO

方法定义在 ijkmedia/ijksdl/android/ijksdl_android_jni.h 中：

```
#define JNI_CHECK_GOTO(condition__, env__, exception__, msg__, label__) \
    do { \
        if (!(condition__)) { \
            if (exception__) { \
                SDL_JNI_ThrowException(env__, exception__, msg__); \
            } \
            goto label__; \
        } \
    }while(0)
```

[返回](ijkplayer_main.md)