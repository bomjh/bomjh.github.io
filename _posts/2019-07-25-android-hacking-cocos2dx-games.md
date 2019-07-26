---
layout: post
title: "Android Hacking: Cocos2dx Games"
author: "Bomjh"
categories: "Reversing"
---

모바일 게임 엔진 중에 유니티 다음으로 가장 많이 사용되는 엔진이 Cocos2dx입니다. Cocos2dx는 게임에 사용되는 여러 기능을 지원해주고, 오픈 소스이기 때문에 자신이 원하는 기능을 수정할 수 있습니다. 그리고 C++로 제작하여 하나의 소스로 Android에서 사용하는 APK, iOS에서 사용하는 IPA 파일을 만들 수 있습니다. 아래 글은 자주 볼 수 있는 cocos2dcpp, cocos2dlua에 대한 내용입니다.

## cocos2dcpp

cocos2dcpp은 C++로 작성된 일반적인 Native Library와 비슷합니다. 보안이 적용되지 않은 라이브러리를 IDA로 열어보면 게임 내 코드들의 심볼 이름을 볼 수 있어서 어셈블리어를 알고 있다면 아주 쉽게 변조를 할 수 있습니다. 만약 심볼명이 숨겨져 있다면 IDA의 동적 디버깅 기능을 사용하여 추적해 나갈 수 있습니다.

![cocos1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/cocos1.png)
_libcocos2dcpp.so_

Hex Editor를 사용하여 파일을 수정하는 방법, 메모리 코드 패치, 후킹 등을 통해서 해킹이 이루어집니다.

## cocos2dlua

cocos2dlua는 cocos2dx에 확장 언어로 쓰이는 Lua Script를 사용하였습니다. 게임의 주요 코드들을 스크립트에서 읽어옵니다. 개발자들은 게임 업데이트 시 스크립트만 수정해주면 프로젝트를 다시 컴파일 할 필요가 없습니다. APK 파일에 담겨있는 Lua 파일들은 컴파일과 난독화가 되어 있기 때문에 일반 텍스트로 열어보면 알아볼 수가 없습니다. 모드를 제작하기 위한 난이도가 있는 편이어서 공유되고 있는 APK를 찾아보기 어렵습니다. 해킹을 하기 위해서는 복호화된 파일이 필요한데, Lua 메뉴얼 사이트에서 주요 함수에 대해서 찾아볼 수 있습니다.

![cocos2](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/cocos2.png)
_lua_load_

![cocos3](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/cocos3.png)
_luaL_loadbuffer_

Lua가 실행하는 코드의 각 부분을 chunk라고 합니다. 해당 함수를 후킹하여 복호화된 파일을 얻어낼 수 있고, 수정된 스크립트를 삽입할 수도 있습니다.
