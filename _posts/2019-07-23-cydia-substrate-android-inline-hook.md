---
layout: post
title:  "Cydia Substrate Android Inline Hook"
author: "Bomjh"
categories: "Hooking"
---

[Cydia Substrate](http://www.cydiasubstrate.com/)(이하 Cydia)는 Android와 iOS에서 인라인 후킹을 지원해주는 플랫폼입니다. 현재 공식적으로 Android 버전 4.3까지, iOS 버전 9.1까지 지원하고 있습니다. 하지만 공개된 소스코드가 있기 때문에 더 높은 Android 버전에서도 사용할 수 있고, armeabi, armeabi-v7a, x86 등 원하는 아키텍쳐 별로 라이브러리를 직접 만들 수도 있습니다.

### 동작 원리

인라인 후킹이란 대상 함수에 대한 호출을 가로채어 자신이 원하는 동작을 수행한 다음 다시 대상 함수를 실행하는 것으로, 일반적으로 사용되는 후킹 방법입니다.

![cydia1]("https://github.com/bomjh/bomjh.github.io/blob/master/assets/cydia1.png")
_arm code in ida_

![cydia2]("https://github.com/bomjh/bomjh.github.io/blob/master/assets/cydia2.png")
_pseudo code in ida_

위 사진과 같이 대상 함수에서 자신이 만든 코드의 주소로 점프하여 원하는 동작을 수행하도록 합니다. Cydia에서는 `MSHookFunction` 함수를 사용하여 쉽게 구현할 수 있습니다.

### 후킹 방법

1. AndroidManifest 설정
코드를 로드하려면 패키지에 `cydia.permission.SUBSTRATE` 권한이 있어야합니다.
&nbsp;
{% highlight xml %}
&lt;manifest xmlns:android="http://schemas.android.com/apk/res/android"
    android:installLocation="internalOnly"&gt;
&nbsp;
    &lt;application android:hasCode="false"&gt;
    &lt;/application&gt;
&nbsp;
    &lt;uses-permission android:name="cydia.permission.SUBSTRATE"/&gt;
&lt;/manifest&gt;
{% endhighlight %}
&nbsp;
2. 라이브러리 설정
`MSConfig`로 코드를 로드할 위치를 지정하고, 라이브러리를 초기화하는 함수를 선언해야 합니다.
&nbsp;
{% highlight cpp %}
\#include <substrate.h>
&nbsp;
MSConfig(MSFilterExecutable, "/system/bin/app_process")
&nbsp;
// this is a macro that uses __attribute__((__constructor__))
MSInitialize {
    // ... code to run when extension is loaded
}
{% endhighlight %}
3. 코드 작성
&nbsp;
{% highlight cpp %}
void MSHookFunction(void \*symbol, void \*hook, void \**old);
{% endhighlight %}
symbol은 대상 함수의 주소이고, hook는 대체할 함수의 주소이고, old는 대상 함수를 호출할 때 사용할 포인터입니다.
&nbsp;
{% highlight cpp %}
void \*(\*oldConnect)(int, const sockaddr \*, socklen_t);
&nbsp;
void \*newConnect(int socket, const sockaddr \*address, socklen_t length) {
    if (address->sa_family == AF_INET) {
          sockaddr_in \*address_in = address;
        if (address_in->sin_port == htons(6667)) {
            sockaddr_in copy = \*address_in;
            address_in->sin_port = htons(7001);
            return oldConnect(socket, &copy, length);
        }
    }
&nbsp;
    return oldConnect(socket, address, length);
}
&nbsp;
MSHookFunction(&connect, &newConnect, &oldConnect);
{% endhighlight %}
