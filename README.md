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
