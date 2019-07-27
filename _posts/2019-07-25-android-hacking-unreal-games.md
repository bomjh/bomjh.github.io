---
layout: post
title: "Android Hacking: Unreal Games"
author: "Bomjh"
categories: "Reversing"
---

Unreal 엔진은 C++ 기반의 엔진으로 FPS, MMORPG 등 실사 비주얼의 표현이 중요한 게임에 주로 사용됩니다. 개발에 필요한 고성능의 도구들이 엔진에 잘 통합되어 있고, 블루프린트를 통해 쉽게 프로토 타입을 제작할 수 있습니다. 모바일 게임 히트 이후로 언리얼 엔진이 많이 등장하고 있지만, 유니티 엔진에 비해서 차지하는 비중이 낮습니다. 대체로 빌드 파일이 엄청 크기 때문에 해커들이 접근하는 것을 꺼려하게 만들 수 있습니다.

![unreal1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/unreal1.png)
_unreal ungine_

심볼명을 쉽게 찾아볼 수 있어서 IDA를 통해서 코드 분석이 가능하고, Hex 수정 및 메모리 코드 패치가 가능합니다. GameGuardian과 같은 치트 도구를 사용하여 메모리 조작도 가능하나, 후킹으로는 변조가 불가능한 것으로 알고 있습니다.
