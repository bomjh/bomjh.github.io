---
layout: post
title:  "Android Inline Hook: Cydia Substrate"
author: "Bomjh"
categories: "Hooking"
---

&nbsp;
[Cydia Substrate](http://www.cydiasubstrate.com/)(이하 Cydia)는 Android와 iOS에서 인라인 후킹을 지원해주는 플랫폼입니다. 현재 공식적으로 Android 버전 4.3까지, iOS 버전 9.1까지 지원하고 있습니다. 하지만 공개된 소스코드가 있기 때문에 더 높은 Android 버전에서도 사용할 수 있고, armeabi, armeabi-v7a, x86 등 원하는 아키텍쳐 별로 라이브러리를 직접 만들 수도 있습니다.

인라인 후킹은 일반적으로 사용되는 후킹 방법으로, 대상 함수에 대한 호출을 가로채어 자신이 원하는 동작을 수행한 다음 다시 대상 함수를 실행하는 기술입니다. 실제로 BlackMod, PlatinMods, GameModPro 등 여러 사이트에서 인라인 후킹을 사용하여 모드를 제작 및 배포하고 있습니다.

## 작동 방식

![cydia1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/cydia1.png)
_arm code in ida_

![cydia2](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/cydia2.png)
_pseudo code in ida_

위 사진과 같이, 대상 함수에서 자신이 만든 코드의 주소로 점프하여 원하는 동작을 수행하도록 합니다. Cydia에서는 이러한 기능을 `MSHookFunction` 함수를 사용하여 쉽게 구현할 수 있습니다. 아래는 JNI에서 작성한 예제입니다.

## Hook Native Code

* AndroidManifest 설정

코드를 로드하려면 패키지에 `cydia.permission.SUBSTRATE` 권한이 있어야합니다.

{% highlight xml %}
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    android:installLocation="internalOnly">

    <application android:hasCode="false">
    </application>

    <uses-permission android:name="cydia.permission.SUBSTRATE"/>
</manifest>
{% endhighlight %}
* 라이브러리 설정

`MSConfig`로 코드를 로드할 위치를 지정하고, 라이브러리를 초기화하는 함수를 선언해야 합니다.

{% highlight cpp %}
#include <substrate.h>

MSConfig(MSFilterExecutable, "/system/bin/app_process")

// this is a macro that uses __attribute__((__constructor__))
MSInitialize {
    // ... code to run when extension is loaded
}
{% endhighlight %}
* 코드 작성

{% highlight cpp %}
// symbol: 대상 함수 주소
// hook: 대체 함수 주소
// old: 대상 함수를 호출할 때 사용할 포인터
void MSHookFunction(void *symbol, void *hook, void **old);
{% endhighlight %}

{% highlight cpp %}
void *(*oldConnect)(int, const sockaddr *, socklen_t);

void *newConnect(int socket, const sockaddr *address, socklen_t length) {
    if (address->sa_family == AF_INET) {
          sockaddr_in *address_in = address;
        if (address_in->sin_port == htons(6667)) {
            sockaddr_in copy = *address_in;
            address_in->sin_port = htons(7001);
            return oldConnect(socket, &copy, length);
        }
    }

    return oldConnect(socket, address, length);
}

MSHookFunction(&connect, &newConnect, &oldConnect);
{% endhighlight %}

## Hook Java Code

* AndroidManifest 설정

Native Code와 같습니다.

* 라이브러리 설정

Native Code와 같습니다.

* 클래스 참조

먼저 해당 클래스를 참조해야 합니다. 해당 클래스가 로드될 때 실행할 수 있는 콜백이 제공됩니다.

{% highlight cpp %}
static void OnResources(JNIEnv *jni, jclass resources, void *data) {
    // ... code to modify the class when loaded
}

MSInitialize {
    MSJavaHookClassLoad(NULL, "android/content/res/Resources", &OnResources);
}
{% endhighlight %}

* 코드 작성

JNI 포인터를 사용하여 구현합니다.

{% highlight cpp %}
static jint (*_Resources$getColor)(JNIEnv *jni, jobject _this, ...);

static jint $Resources$getColor(JNIEnv *jni, jobject _this, jint rid) {
    jint color = _Resources$getColor(jni, _this, rid);
    return color & ~0x0000ff00 | 0x00ff0000;
}

static void OnResources(JNIEnv *jni, jclass resources, void *data) {
    jmethodID method = jni->GetMethodID(resources, "getColor", "(I)I");
    if (method != NULL)
        MSJavaHookMethod(jni, resources, method,
            &$Resources$getColor, &_Resources$getColor);
}
{% endhighlight %}
