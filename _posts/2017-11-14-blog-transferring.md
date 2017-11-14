---
layout:     post
title:      test
summary:    윈도우에 지킬을 사용해 깃헙 페이지 블로그 설치하기
categories: jekyll pixyll 깃헙
---

# 윈도우 사용자의 깃헙 페이지 블로그 설치기

블로깅을 꾸준히 하고 싶은 마음은 있었지만, 게으른 천성으로 인해 HTML 포매팅을 한다던지 하는건 절대 싫고 가능하면 코드와 텍스트를 날것 그대로 쓰고 싶었다. 깃헙 페이지 블로그에 마크다운으로 블로깅을 하려고 생각은 진작 하고 있었지만 게으름으로 인해... 

깃헙 페이이지를 그냥 써도 되겠지만 대세를 따라 지킬을 사용하기로 했다. 자 지킬을 깔아보자!

## 지킬을 깔자

[지킬 홈피](https://jekyllrb.com)에 들어갔다. 지킬 홈피 초기 화면에 보니 빠른 설치에 `gem`이 필요하단다. 십년만에 루비시스템이군. 그럼 일단 루비부터 깔아야 하는건가? 난 회사 사정상 윈도우 7 사용자다. 예전 기억으로 윈도즈에서 지킬 설지가 뭔가 불편했던 것 같다. [지킬 온 윈도우즈](https://jekyllrb.com/docs/windows/)로 들어가본다. "루비 인스톨러를 사용한 설치"를 따라해보자. 루비 인스톨러가 필요하다.

### 루비부터 깔자

[루비 인스톨러 홈페이지](https://rubyinstaller.org/)에 들어간다. 빨간 다운로드 버튼이 보인다. 과감히 클릭한다!

![루비 인스톨러 다운로드]({{ site.url }}/images/rubyInstaller.png)

잘 모르니 최신판으로 Ruby 2.4.2-2 x64를 클릭하자.

설치해보면 MSYS2가 필요하단다. 잘 모르겠으니 엔터를 누르자.

![루비인스톨러 MSYST2]({{ site.url }}/images/rubyInstallerMsysCheck.png)

열심히 기도하다 보면 설치가 끝난다. 네트워크 상황에 따라 걸리는 시간은 다르다. 문제가 생겼다고? 나도 잘 모른다. 스택오버플로우나 구글에게 물어보라.

### 드디어 지킬을 깔아보자!

루비를 설치했다면 루비 패키지 관리 프로그램인 `gem`도 설치됐을 것이다. `gem`으로 지킬을 깔자.

```
gem install jekyll bundler
```

설치가 다 됐다면 한번 지킬을 실행해 본다.

```
C:\Users\Hyunsok> jekyll -v
jekyll 3.6.2
```

## 깃헙 페이지를 만들자

깃헙 페이지에서 블로그를 사용하려면 자신의 깃헙에 *게정이름.github.io*라는 리포지터리를 만들어야 한다. 깃헙 사용법은 따로 적지 않겠다. 

만들고 나면 다음과 같은 절차를 거쳐서 로컬 컴퓨터에 블로그용 깃헙 리포를 만들고 원격과 연결한다. 난 깃 사용시 *cygwin*의 깃을 사용하기 때문에 다음과 같이 했지만 윈도우 깃 사용자는 GUI를 쓰던 명령줄에서 하던 기본적인 절차는 같다.

```
$ cd /cygdrive/e/blog/frank-oh.github.io
$ git init
$ cat > index.html
Hello World!
$ git add *
$ git commit -m "initial"
(메시지 생략)
$ git remote add origin https://frank-oh@github.com/frank-oh/frank-oh.github.io.git
$ git push -u origin master
Password for 'https://frank-oh@github.com':
(메시지 생략)
```

이제 잘 되나 브라우저에서 `https://frank-oh.github.io/`를 열어보자.

![Hello World 인덱스페이지]({{ site.url }}/images/helloworld.png)

좋다. 이제 본격적으로 지킬을 사용해 홈피를 구축할 때다.

## 지킬 테마를 찾자

[지킬 테마 페이지](https://github.com/jekyll/jekyll/wiki/Themes)에 가보면 엄청난 양의 지킬 테마를 볼 수 있다. 난 그 중에서 john Otander의 [pixyll](https://github.com/johnotander/pixyll)을 택했다.

포크후 설치하는 방법도 있지만 리포를 덮어쓰는 방법을 택하기로 했다. pixyll 깃 페이지에서 "Clone or Download" 버튼을 눌러 소스를 다운로드한다. 

![다운로드]({{ site.url }}/images/pixyll_download.png)

블로그 디렉터리의 모든 파일을 지우고 다운로드한 소스 파일을 블로그 디렉터리에 복사한다.

디렉터리에는 다음과 같은 파일이 생긴다.

![복사후 디렉터리 내용]({{ site.url }}/images/afterPixyllCopy.png)

### `_config.yml` 설정

`_config.yml` 설정을 적당히 바꾼다.

```
# Site settings
title: 사이트 타이틀
email: 전자우편 주소
author: 이름
description: "설명"
baseurl: ""
url: "블로그 URL"

# Build settings
markdown: kramdown
permalink: pretty
paginate: 5
```

### 로컬에서 실행

지키릉 실행해 로컬 서버를 돌리면서 블로그를 미리 살펴볼 수 있다. `--watch`를 설정하면 소스코드 변경시 자동으로 html이 갱신된다.

```
$ jekyll serve --watch
```


혹시 윈도우즈에서 jekyll이 오류를 낸다면 `bundle install`이나 `bundle update`로 번들을 업데이트한 다음에 `bundle exec jekyll serve --watch`로 실행하라.

`http://localhost:4000/`를 브라우저에서 열면 다음과 같은 페이지를 볼 수 있다.

![]({{ site.url }}/images/pixyll_initial.png)



