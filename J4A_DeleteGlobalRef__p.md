# J4A\_DeleteGlobalRef\_\_p

定义在 ijkmedia/ijkj4a/j4a/j4a_base.h 中：

```
void   J4A_DeleteGlobalRef__p(JNIEnv *env, jobject *obj);
```

实现在 ijkmedia/ijkj4a/j4a/j4a_base.c 中：

```
void J4A_DeleteGlobalRef(JNIEnv *env, jobject obj)
{
    if (!obj)
        return;
    (*env)->DeleteGlobalRef(env, obj);
}

void J4A_DeleteGlobalRef__p(JNIEnv *env, jobject *obj)
{
    if (!obj)
        return;
    J4A_DeleteGlobalRef(env, *obj);
    *obj = NULL;
}
```

删除全局变量。

[返回](jni_set_media_data_source.md)