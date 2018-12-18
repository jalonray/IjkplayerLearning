# SDL\_CreateThreadEx

声明在 ijkmedia/ijksdl/ijksdl_thread.h 中：

```
SDL_Thread *SDL_CreateThreadEx(SDL_Thread *thread, int (*fn)(void *), void *data, const char *name);
```

用具体实现分别实现于几个不同的 c 文件中，用于对系统进行适配。

首先是 ijkmedia/ijksdl/ijksdl_thread.c 中：

```
SDL_Thread *SDL_CreateThreadEx(SDL_Thread *thread, int (*fn)(void *), void *data, const char *name)
{
    thread->func = fn;
    thread->data = data;
    strlcpy(thread->name, name, sizeof(thread->name) - 1);
    int retval = pthread_create(&thread->id, NULL, SDL_RunThread, thread);
    if (retval)
        return NULL;

    return thread;
/}
```

[返回](ijkplayer_main.md)