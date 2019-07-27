---
layout: post
title: "Android Hook: Xposed"
author: "Bomjh"
categories: "Hooking"
---

&nbsp;
[Xposed](https://repo.xposed.info/)는 APK를 건드리지 않고 시스템 및 앱의 동작을 변경할 수 있는 인라인 후킹 모듈 제작을 지원해주는 프레임 워크입니다. 모듈로 제작한다는 것은 대상의 코드가 너무 많이 변경되지 않는 한 계속 사용할 수 있다는 것을 의미합니다. 공식적으로 Android Oero 버전까지 지원하고 있고, 이미 제작된 많은 모듈들이 공유되고 있습니다. 모듈을 관리하기 위해서 `Xposed Framework`를 설치해야하며, 프레임 워크가 없으면 모듈이 작동하지 않습니다.

## 작동 방식

zygote라고 불리는 Android Runtime의 핵심 프로세스가 있습니다. zygote은 `/system/bin/app_process`에서 시작되며, 모든 프로세스는 zygote의 fork를 통하여 생성됩니다. Xposed Framework를 설치하게 되면 app_process가 수정된 파일로 대체됩니다. 대체된 app_process는 zygote가 시작될 때 classpath에 Java 후킹을 지원하는 XposedBridge.jar를 추가합니다. jar에 포함된 `findAndHookMethod` 함수를 사용하여 후킹을 실행합니다.

## 예제

* AndroidManifest 수정

meta-data를 등록하여 Xposed Framework에 의해 관리받을 수 있도록 합니다.

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="de.robv.android.xposed.mods.tutorial"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk android:minSdkVersion="15" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="Easy example which makes the status bar clock red and adds a smiley" />
        <meta-data
            android:name="xposedminversion"
            android:value="53" />
    </application>
</manifest>
{% endhighlight %}

* 코드 작성

지정된 패키지가 로드될 때만 후킹이 실행되도록 합니다. `beforeHookedMethod`, `afterHookedMethod`를 사용하여 대상 함수 호출의 전, 후에서 실행할 함수를 작성합니다.

{% highlight java %}
package de.robv.android.xposed.mods.tutorial;

import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
    	if (!lpparam.packageName.equals("com.android.systemui"))
            return;

    	findAndHookMethod("com.android.systemui.statusbar.policy.Clock", lpparam.classLoader, "updateClock", new XC_MethodHook() {
    		@Override
    		protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
    			// this will be called before the clock was updated by the original method
    		}
    		@Override
    		protected void afterHookedMethod(MethodHookParam param) throws Throwable {
    			// this will be called after the clock was updated by the original method
    			TextView tv = (TextView) param.thisObject;
    			String text = tv.getText().toString();
    			tv.setText(text + " :)");
    			tv.setTextColor(Color.RED);
    		}
	});
    }
}
{% endhighlight %}
