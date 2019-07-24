---
layout: post
title: "Android Hook: Frida"
author: "Bomjh"
categories: "Hooking"
---

&nbsp;
[Frida](https://www.frida.re/)는 개발자, 리버스 엔지니어 및 보안 연구원을 위한 동적 분석 도구입니다. 모듈 제작이나 컴파일 과정 없이 스크립트만 작성하면 바로 동작하기 때문에 편리하고 빠른 분석이 가능합니다. Windows, macOS, Linux, Android 등의 네이티브 앱에 JavaScript를 삽입하여 Java Code 및 Native Code를 후킹할 수 있습니다.

## 작동 방식

Frida는 zygote를 후킹하여 작동되고, 양방향 통신 채널을 통해 대화식으로 후킹을 진행할 수 있습니다.

![frida](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/frida.png)
_frida architecture_

## 사용법 및 예제

* Python 3.x 버전 설치

* frida 설치

{% highlight bash %}
$ pip install frida-tools
{% endhighlight %}

* [frida-server](https://github.com/frida/frida/releases) 다운로드

* frida-server 실행

{% highlight bash %}
$ adb root # might be required
$ adb push frida-server /data/local/tmp/
$ adb shell "chmod 755 /data/local/tmp/frida-server"
$ adb shell "/data/local/tmp/frida-server &"
{% endhighlight %}

* frida가 작동하는지 확인

{% highlight bash %}
$ frida-ps -U
{% endhighlight %}

{% highlight bash %}
PID NAME
1590 com.facebook.katana
13194 com.facebook.katana:providers
12326 com.facebook.orca
13282 com.twitter.android
…
{% endhighlight %}

* 코드 작성

Java Code Hook

{% highlight python %}
import frida, sys

def on_message(message, data):
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

jscode = """
Java.perform(function () {
    // 후킹할 함수입니다.
    var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity');

    // 버튼이 클릭되면
    MainActivity.onClick.implementation = function (v) {
        // 함수가 호출되었음을 알리는 메세지를 보여줍니다.
        send('onClick');

        // 기존의 onClick을 호출합니다.
        this.onClick(v);

        // 기존의 onClick을 실행한 후에 값을 설정합니다.
        this.m.value = 0;
        this.n.value = 1;
        this.cnt.value = 999;

        // 콘솔에서 로그를 볼 수 있습니다.
        console.log('Done:' + JSON.stringify(this.cnt));
    };
});
"""

process = frida.get_usb_device().attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(jscode)
script.on('message', on_message)
print('[*] Running CTF')
script.load()
sys.stdin.read()
{% endhighlight %}

Native Code Hook

{% highlight python %}
import frida, sys

def on_message(message, data):
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

jscode = """
Interceptor.attach(Module.findExportByName("libc.so", "strstr"), {

    onEnter: function(args) {

        // args를 사용하여 함수 인자 값을 확인할 수 있습니다
        // this를 사용하면 onLeave에서도 사용할 수 있습니다.
        this.haystack = args[0];
        this.needle = args[1];
        this.isRoot = Boolean(0);
    },

    onLeave: function(retval) {

        if (this.isRoot) {
            // 반환 값을 조작합니다.
            retval.replace(0);
        }
        return retval;
    }
});
"""

process = frida.get_usb_device().attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(jscode)
script.on('message', on_message)
print('[*] Running CTF')
script.load()
sys.stdin.read()
{% endhighlight %}
