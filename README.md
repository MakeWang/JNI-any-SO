# JNI的使用
## 目录<br/>
* [java与jni相互调用](#java与jni相互调用)
* [android动态加载so库](#android动态加载so库)

#### java与jni相互调用
----------------------------------------------------
##### 1、java访问jni字符串
```java
JNIEXPORT jstring JNICALL Java_com_wy_test_Test_getTest
(JNIEnv * jenv, jclass jclass){
	return (*jenv)->NewStringUTF(jenv, "C string");
}
```
##### 2、访问java非静态方法
```java
JNIEXPORT void JNICALL Java_com_wy_test_Test_getTest2
(JNIEnv * env, jobject jobj){
	//得到jobject得到jclass
	jclass jcls = (*env)->GetObjectClass(env, jobj);
	
	jmethodID jmid = (*env)->GetMethodID(env, jcls, "getA", "(Ljava/lang/String;)V");
	char* name = "nihao";
	jstring jstr = (*env)->NewStringUTF(env,name);
	(*env)->CallVoidMethod(env, jobj, jmid, jstr);
}
```
##### 3、访问java静态方法
```java
JNIEXPORT void JNICALL Java_com_wy_test_Test_getTest3
(JNIEnv * env, jobject jobj){
	jclass jcls = (*env)->GetObjectClass(env, jobj);

	jmethodID jmid = (*env)->GetStaticMethodID(env,jcls,"getB","(Ljava/lang/String;)V");
	(*env)->CallStaticVoidMethod(env, jcls, jmid,(*env)->NewStringUTF(env,"hahah"));
}
```
##### 4、访问java构造方法
```java
JNIEXPORT jobject JNICALL Java_com_wy_test_Test_getTest4
(JNIEnv * env, jobject jobj) {
	//通过类的路径来从JVM 里面找到对应的类
	jclass jclz = (*env)->FindClass(env, "java/util/Date");
	//jmethodid
	jmethodID jmid = (*env)->GetMethodID(env, jclz, "<init>", "()V");

	//调用 newObject 实例化Date 对象，返回值是一个jobjcct
	jobject date_obj = (*env)->NewObject(env, jclz, jmid);

	//得到对应对象的方法，前提是，我们访问了相关对象的构造函数创建了这个对象
	jmethodID time_mid = (*env)->GetMethodID(env, jclz, "getTime", "()J");
	jlong time = (*env)->CallLongMethod(env, date_obj, time_mid);

	printf("time: %lld \n", time);
	return date_obj;
}
```
##### 5、java与jni的中文乱码问题
```java
JNIEXPORT jobject JNICALL Java_com_dongnao_alvin_Jni_1Test_chineseChars
(JNIEnv * env, jobject jobj, jstring in) {
	jboolean iscp;
	char * c_str = (*env)->GetStringChars(env, in, &iscp);
	if (iscp == JNI_TRUE)
	{
		printf("is copy: JNI_TRUE\n");
	}
	else if (iscp == JNI_FALSE)
	{
		printf("is copy: JNI_FALSE\n");
	}

	int length = (*env)->GetStringLength(env, in);
	const jchar * jcstr = (*env)->GetStringChars(env, in, NULL);
	if (jcstr == NULL) {
		return NULL;
	}
	//jchar -> char
	char * rtn = (char *)malloc(sizeof(char) *2 * length + 3);
	memset(rtn, 0, sizeof(char) * 2 * length + 3);
	int size = 0;
	size = WideCharToMultiByte(CP_ACP, 0, (LPCWSTR)jcstr, length, rtn, sizeof(char) * 2*length + 3, NULL, NULL);
	
	if (rtn != NULL) {
		free(rtn);
		rtn = NULL;
	}
	(*env)->ReleaseStringChars(env, in, c_str);// JVM 使用。通知JVM c_str 所指的空间可以释放了
	printf("string: %s\n", rtn);
	
	return NULL;
	//方法二：
	//char *c_str = "年后";
	//jclass str_cls = (*env)->FindClass(env, "java/lang/String");
	//jmethodID jmid = (*env)->GetMethodID(env, str_cls, "<init>", "([BLjava/lang/String;)V");
	//
	////jstring -> jbyteArray
	//jbyteArray bytes = (*env)->NewByteArray(env, strlen(c_str));
	//// 将Char * 赋值到 bytes
	//(*env)->SetByteArrayRegion(env, bytes, 0, strlen(c_str), c_str);
	//jstring charsetName = (*env)->NewStringUTF(env, "GB2312");
	//return (*env)->NewObject(env, str_cls, jmid, bytes, charsetName);
}
```

#### android动态加载so库
----------------------------------------------------
```java

#include <jni.h>
#include <android/log.h>
#include <assert.h>

#define TAG "wangyin"
#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, TAG, __VA_ARGS__)
# define NELEM(x) ((int) (sizeof(x) / sizeof((x)[0])))

/*JNIEXPORT void JNICALL Java_com_wy_test_MainActivity_stringFromJNI(JNIEnv* env,jobject jObj) {
    LOGI("这是JNI的日志");
}*/

JNIEXPORT jint JNICALL native_diff(JNIEnv *env, jclass type, jint a, jint b) {
    return a + b;
}

static const JNINativeMethod getMethod_FileUtil[] = {
        {
                "diff","(II)I",(int*)native_diff
        }
};

static int registerNative_FileUtil(JNIEnv* env){
    LOGI("registerNative begin");
    jclass jcls;
    jcls = (*env)->FindClass(env,"com/wy/test/FileUtil");
    if(jcls == NULL){
        LOGI("jclass is NULL");
        return JNI_FALSE;
    }
    if((*env)->RegisterNatives(env,jcls,getMethod_FileUtil,NELEM(getMethod_FileUtil)) < 0){
        LOGI("RegisterNatives error");
        return JNI_FALSE;
    }
    return JNI_TRUE;
}
JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    LOGI("jni_OnLoad begin");

    JNIEnv* env = NULL;
    jint result = -1;
    if ((*vm)->GetEnv(vm,(void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        LOGI("ERROR: GetEnv failed\n");
        return -1;
    }
    assert(env != NULL);
    registerNative_FileUtil(env);
    return JNI_VERSION_1_4;
}


```
