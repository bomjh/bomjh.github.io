---
layout: post
title: "Android Hacking: Unity Games"
author: "Bomjh"
categories: "Reversing"
---

2018년 조사 자료에 따르면 40% 이상의 모바일 게임이 유니티 엔진을 사용하여 개발되고 있다고 합니다. 그 이유는 버튼 몇 번만 누르면 다양한 플랫폼으로 빌드가 가능하고, 초보 개발자들도 쉽게 사용할 수 있는 개발 환경을 가지고 있기 때문입니다. 하지만 보안이 적용되지 않은 유니티 게임은, 여러 가지 디컴파일 도구를 사용하여 아주 쉽게 원본 수준의 코드를 볼 수 있고 수정까지 가능합니다.

## Mono

유니티 게임에서 가장 많이 사용하는 mono(jit)는 IL코드로 작성된 dll 파일을 읽어서 Assembly 코드로 변환하는 역할을 합니다. 유니티 게임의 APK에는 게임 내의 모든 코드가 들어있는 dll 파일들이 있고, 이를 .Net Decompiler를 사용하여 IL 또는 C# 코드로 볼 수 있습니다. 디컴파일 도구로는 dnSpy를 가장 많이 사용합니다.

![unity1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/unity1.png)
_dnspy_

디컴파일된 코드는 EditMethod 기능을 사용하여 아주 쉽게 변조가 가능합니다. 컴파일 시 에러가 발생하면 Edit IL Instructions 기능을 사용하여 IL코드로도 수정할 수 있습니다. APK 파일을 직접 열어서 수정된 dll 파일로 교체해주고 재서명을 해주면 MOD APK를 만드는 작업이 완료됩니다.

그리고 mono는 오픈 소스로 공개되어 있기 때문에 내부 동작을 쉽게 파악할 수 있습니다. 오픈 소스를 사용하여 보안이 추가된 mono를 직접 만들 수도 있지만, 해킹에도 많이 이용됩니다. 후킹에 사용되는 주요 API 함수는 mono_image_open_from_data_with_name, mono_runtime_invoke 등이 있습니다.

![unity2](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/unity2.png)
_mono open source_

## Il2cpp

유니티에서 Il2cpp로 선택하여 빌드하게 되면 IL코드가 C++코드로 변환되어 Native Library로 컴파일됩니다. 게임 코드들이 어셈블리어로 바뀌기 때문에 mono처럼 쉽게 해킹할 수 없습니다. 그리고 mono와는 다르게 JIT 컴파일 대신 AOT 컴파일 방식을 사용하여 속도면에서 더 향상되었다고 합니다. 하지만 Il2CppDumper 도구를 사용하여 심볼 이름, 위치 등의 정보를 알아낼 수 있어서 역시 게임 코드의 변조가 가능합니다.

![unity3](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/unity3.png)
_il2cppdumper_

![unity4](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/unity4.png)
_dump.cs_

libil2cpp.so 파일과 global-metadata.dat 파일을 사용하여 DummyDll, dump.cs를 추출할 수 있습니다. 추출한 정보로 IDA를 통해서 코드를 분석하고 Winhex와 같은 Hex Editor를 사용하여 Hex값을 수정하거나, 후킹에 이용할 수 있습니다. 실제로 il2cpp 게임의 모드 APK를 보면 Hex값 수정보다는 후킹 기능 또는 메모리 코드 패치 기능이 있는 라이브러리를 삽입하는 방법을 많이 사용하는데, 직접 파일을 수정하여 공유하게 되면 다른 모드 사이트에서 쉽게 leech할 수 있기 때문이라고 추측하고 있습니다.
