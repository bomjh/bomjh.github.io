---
layout: post
title: "Create GitHub Blog Simple Guide"
author: "Bomjh"
categories: "Git"
---

GitHub Pages, Jekyll Theme를 사용하여 쉽고 빠르게 개인 블로그를 생성하는 방법입니다.

## Git 설치
[Git](https://git-scm.com/) 사이트에서 Git을 설치합니다. 설치 후 Git Bash를 실행하여 사용자 등록을 진행합니다.

{% highlight bash %}
$ git config --global user.name "github 이름"
$ git config --global user.email "github 이메일"
{% endhighlight %}

## 새로운 원격 저장소 생성
GitHub에서 새로운 원격 저장소를 생성합니다. 저장소 이름은 username.github.io로 설정합니다.

![blog1](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/blog1.png)
_create a new repository_

## Theme 선택
[JekyllThemes](http://jekyllthemes.org/) 사이트나 [GitHub](https://github.com/topics/jekyll-theme)에 공개되어 있는 테마 중에 마음에 드는 테마를 하나 선택합니다. 라이센스가 MIT 라이센스인지 확인하고, Homepage 버튼을 눌러서 해당 테마의 원격 저장소로 이동합니다.

![blog2](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/blog2.png)
_select jekyll theme_

## Theme 복사하기
해당 테마의 원격 저장소를 로컬 저장소로 가져옵니다.

{% highlight bash %}
$ git clone https://github.com/thelehhman/texture
{% endhighlight %}

복사가 완료되면 폴더명을 username.github.io로 바꿔주고, 폴더 안에 있는 .git 폴더를 삭제합니다.

![blog3](https://raw.githubusercontent.com/bomjh/bomjh.github.io/master/assets/blog3.png)
_clone jekyll theme_

## Theme 적용하기
필요한 파일들을 수정합니다.

{% highlight markdown %}
- _layouts: 레이아웃 html 폴더
- _posts: 포스트 작성 폴더
- _config.yml: 설정 변경
- index.md: 첫 화면 설정
- README.md: 소개
{% endhighlight %}

username.github.io 폴더를 원격 저장소로 사용하기 위해 초기화를 진행합니다.

{% highlight bash %}
$ git init
{% endhighlight %}

원격 저장소 주소를 등록해줍니다. origin이 아니라 다른 이름을 사용해도 됩니다.

{% highlight bash %}
$ git remote add origin https://github.com/bomjh/bomjh.github.io
{% endhighlight %}

원격 저장소가 연결되었는지 확인합니다. 아래 명령어를 입력하면 연결된 원격 저장소가 출력됩니다.

{% highlight bash %}
$ git remote
{% endhighlight %}

모든 수정 및 추가된 파일을 원격 저장소에 보냅니다.

{% highlight bash %}
$ git add .
$ git commit -m "first commit"
$ git push origin master
{% endhighlight %}

잠시 후 [https://bomjh.github.io/](https://bomjh.github.io/) 주소로 접속하면 테마가 적용된 모습을 확인할 수 있습니다.
