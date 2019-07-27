---
layout: post
title: "Android Hook: LD_PRELOAD"
author: "Bomjh"
categories: "Hooking"
---

&nbsp;
[LD_PRELOAD](http://man7.org/linux/man-pages/man8/ld.so.8.html)는 Linux에서 사용되는 환경변수로, 이 환경변수에 등록된 라이브러리들은 프로세스 실행 시 다른 라이브러리들이 로드되기 전에 우선적으로 로드됩니다. 만약 우선적으로 로딩된 라이브러리의 함수 중에 이 후에 로드된 라이브러리의 함수와 이름이 동일한 함수가 있다면, 먼저 로딩된 라이브러리의 함수를 호출하게 됩니다. 따라서 후킹과 같은 결과를 얻을 수 있습니다.

Android에서 모든 프로세스는 zygote의 fork를 통해서 생성되므로 zygote를 부모 프로세스로 가집니다. 그리고 zygote는 init 프로세스를 부모 프로세스로 가집니다. 부모 프로세스의 환경변수는 자식 프로세스에 그대로 물려주기 때문에, zygote에 LD_PRELOAD 환경변수를 설정해주면 그 후로 생성되는 모든 프로세스는 환경변수에 등록된 라이브러리가 삽입된 채로 실행됩니다.

## 적용 방법

1.`init` 프로세스에 attach
* ptrace를 사용하여 init 프로세스에 attach 합니다.

2.`zygote` 프로세스 kill
* SIGKILL 신호를 보냅니다. zygote이 죽으면 init에서 새로운 zygote을 생성합니다.

3.init 프로세스에서 fork 하는 순간을 포착
* 새로운 zygote 프로세스를 찾습니다. 이 때, arm과 x86에서 사용되는 레지스터를 잘 확인하여 코드를 작성합니다.

4.새로운 zygote에 attach
* ptrace를 사용하여 zygote 프로세스에 attach 합니다.

5.`execve` 시스템 콜을 호출하는 순간을 포착하여 환경변수 배열에 LD_PRELOAD 추가
* execve의 세 번째 인자가 환경변수 배열입니다.

## 실제 적용

환경변수에 LD_PRELOAD를 추가하는 과정입니다.

![ldpreload1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/ldpreload1.png)
_inject environ_

새로운 zygote이 생성되었고 환경변수에 LD_PRELOAD가 추가된 것을 확인할 수 있습니다.

![ldpreload2](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/ldpreload2.png)
_cat environ_

환경변수에 등록된 라이브러리가 로딩된 것을 확인할 수 있습니다.

![ldpreload3](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/ldpreload3.png)
_ld_preload_
