+++
date = "2016-06-25T14:53:48+09:00"
prev = "../"
title = "10분만에 Github Profile 만들기"
toc = true
weight = 11
aliases = [
    "/create-github-profile-in-10-minutes"
]
+++

![](https://raw.githubusercontent.com/1ambda/1ambda.github.io/master/assets/images/oh-my-github/baracktocat-large.jpg?width=200&height=200)

<br/>

Github 데이터를 이용해 프로필을 만들려면 

1. **[Github API](https://developer.github.com/v3/)** 를 이용해 데이터를 긁어옵니다.
2. 데이터를 보여줄 웹 어플리케이션 (*static*) 을 만듭니다.
3. **[Github Page (gh-pages)](https://pages.github.com/)** 를 이용해 남들에게 보여줍니다.

이 때, (1) 에서 만든 데이터의 **포맷을 정형화하면**, 이것을 사용하는 (2) 의 웹 어플리케이션을 일종의 *viewer* 로 생각할 수 있습니다. 포맷이 고정되어 있으므로 데이터를 사용하는 *viewer* 를 **쉽게 교체하거나**, 자신이 원하는대로 **커스터마이징** 할 수 있게 됩니다.

<br/>

### Demo

시작 전에 오늘 만들 결과물의 데모를 보겠습니다.

[Demo (Chrome, Firefox, Safari, IE11+)](http://1ambda.github.io/oh-my-github/)

- **Langauge**: 즐겨 사용하는 프로그래밍 언어
- **Repository**: 레포지토리 정보 (*stargazer*, *fork count* 등)
- **Contribution**: 오픈소스 커밋 내역
- **Activity**: 최근 활동 내역 (*Push*, *PullRequest* 등)

등의 정보를 확인할 수 있습니다. 커스터마이징 등은 아래에서 설명하겠습니다. 

### Prerequisites

이제 Github 프로필을 만들어 보겠습니다. 준비물을 먼저 확인하면,

1. *SSH Key* 가 Github 에 등록이 안되어있을 경우 **[Github: Generating an SSH key](https://help.github.com/articles/generating-an-ssh-key/)** 를 참조해서 등록해주세요.
2. **oh-my-github** 란 이름의 **[Github Repository](https://github.com/new)** 를 만들어주세요. **[Github Page](https://pages.github.com/)** 를 이용해 배포시 사용할 저장소입니다. (이름은 반드시 **oh-my-github** 여야 합니다)
3. **[Github Access Token](https://github.com/settings/tokens/new)** 을 만들어주세요. 50 개 이상의 Github API 호출을 위해선 Access Token 이 꼭 필요합니다. (Write Permission 은 필요 없습니다.)  

데이터를 보여주는 정적 웹 어플리케이션인 [viewer](https://github.com/oh-my-github/viewer) 와 Github API 를 호출해 데이터를 생성하는 [oh-my-github](https://github.com/oh-my-github/oh-my-github) 설치법은 아래서 설명하겠습니다.

### Install: Viewer

먼저 [default viewer](https://github.com/oh-my-github/viewer) 를 클론 받고, `upstream` 업데이트를 위해 remote 를 등록합니다.

```
$ git clone git@github.com:oh-my-github/viewer.git oh-my-github
$ cd oh-my-github

$ git remote add upstream git@github.com:oh-my-github/viewer.git
```

그리고, 위에서 만든 자신의 레포지토리 url 을 `origin` 으로 등록합니다. `[GITHUB_ID]` 대신 자신의 아이디를 사용하면 됩니다.

```
$ git remote remove origin
$ git remote add origin git@github.com:[GITHUB_ID]/oh-my-github
```

### Install: oh-my-github

먼저 [oh-my-github](https://github.com/oh-my-github/oh-my-github) 를 설치하겠습니다. NodeJS 가 없다면, [NVM](https://github.com/creationix/nvm) 설치 후 문서에 나와있는 대로 NodeJS 5.0.0 이상 버전을 설치해 주세요. 이 문서에서는 5.0.0 을 사용하겠습니다.

```
$ nvm use 5.0.0
Now using node v5.0.0

$ nvm ls
   v0.12.9
->  v5.0.0
    v5.4.1
```

이제 [oh-my-github](https://github.com/oh-my-github/oh-my-github) 를 설치합니다. 네트워크 상황에 따라 2분 ~ 4분정도 걸립니다.

```
$ npm install oh-my-github -g
```

만약 Linux 를 사용하고 있고, 위 설치 과정에서 `LIBXXX` 등의 에러를 마주쳤을 경우 [Linux Install Guide](https://github.com/oh-my-github/oh-my-github/wiki/Installation-Guide-for-Linux) 를 참조해주세요.

이제 *viewer* 를 클론 받은 디렉토리로 이동한 뒤 *oh-my-github* 를 실행합니다. 여기서 `[GITHUB_TOKEN]` 은 위에서 만든 [Github Access Token](https://github.com/settings/tokens/new) 값이고, `[GITHUB_ID]` 는 자신의 Github ID 입니다.

```
$ oh-my-github

$ omg init [GITHUB_ID] oh-my-github       # (e.g) omg init 1ambda oh-my-github
$ omg generate [GITHUB_TOKEN]             # (e.g) omg generate 394fbad49191aca
```

`omg generate` 를 실행하면, 현재 디렉토리에 `oh-my-github.json` (프로필 데이터) 가 생성됩니다. [Github Page](https://pages.github.com/) 에 배포하기 전에 먼저 로컬에 띄워 볼 수 있습니다.

```
$ omg preview
```

이제 `gh-pages` 브랜치를 만들고 push 를 해야하는데, 아래의 명령어를 이용해 한번에 해결할 수 있습니다.

```
$ omg publish
```

만약 `omg publish` 명령어가 동작하지 않는다면, 직접 Git 커맨드를 사용하면 됩니다.

```
$ git add --all
$ git commit -m "feat: Update Profile"
$ git checkout -b gh-pages
$ git push origin HEAD
```

이제 30초 정도 기다리고, 자신의 oh-my-github 레포지토리 [Github Page](https://pages.github.com/) URL 을 확인해 봅니다. 예를 들어 Github ID 가 `1ambda` 라면, [http://1ambda.github.io/oh-my-github](http://1ambda.github.io/oh-my-github) 에 프로필이 생성됩니다.

<br/>

### Update

#### Profile

프로필 데이터 `oh-my-github.json` 내용은 크게 분류하면 두가지로 나뉩니다.

- `activities`: 사용자의 활동 정보로, 이전 정보에 새로운 값이 추가됨(*append*)
- `repositories`, `languages` 등: 최신 정보로 덮어 씌워짐 (*overwrite*)

`omg generate` 를 실행할 때 마다, 새로운 이벤트가 있다면 `activities` 값이 추가 (*append*) 됩니다.

> 더 정확히는, Github API 는 최대 10개월 혹은 최대 300개의 event 만 제공하기 때문에, 이것보다 더 많은 양의 event 를 프로필 데이터에 저장하고자 event id 값으로 중복 제거를 한 뒤 *append* 방식으로 데이터를 쌓습니다. 

따라서 프로필 데이터를 업데이트 하고, Github 에 푸시하려면 다음의 명령어를 실행하면 됩니다.

```
$ cd oh-my-github         # oh-my-github.json 이 위치한 곳

$ omg generate [GITHUB_TOKEN]
$ omg publish
```

#### Viewer

*viewer* 를 [upstream](https://github.com/oh-my-github/viewer) 에서 다음처럼 업데이트 할 수 있습니다.

```
$ cd oh-my-github         # oh-my-github.json 이 위치한 곳

$ git checkout master
$ git pull upstream master --rebase

$ git checkout gh-pages
$ git rebase master

$ git push origin HEAD
```

<br/>

### Customizing

만약 [default viewer](https://github.com/oh-my-github/viewer) 가 맘에 들지 않는다거나, 새로운 기능 (e.g 그래프) 을 추가하고 싶다면 클론받아 `app/src` 아래의 코드를 수정할 수 있습니다.

```
app
├── LICENSE.md
├── package.json
├── src (웹 애플리케이션 소스)
│   ├── actions
│   ├── components
│   ├── constants
│   ├── containers
│   ├── reducers
│   ├── store
│   ├── theme
│   └── util
└── tools (빌드도구 관련)
```

`app` 디렉토리로 이동 후 `npm start -s` 를 실행하고 코드를 수정한 뒤, `npm run build` 를 실행하면 루트 디렉토리에 `bundle.js` 와 `index.html` 이 업데이트 됩니다. 이 두 파일을 자신의 **oh-my-github** 에 업데이트 하면 됩니다.

추가로, 다른 사람들이 자신이 수정한 *viewer* 를 찾을 수 있게 [NPM](https://www.npmjs.com/search?q=oh-my-github%2C+viewer) 에 등록하고 싶다면 `package.json` 을 수정 후 `app` 디렉토리에서 `npm publish` 명령을 실행하면 됩니다.

아래의 내용을 수정하고, 배포하면 NPM 에서 `oh-my-github, viewer` 키워드로 검색할 수 있습니다. [(NPM: oh-my-github, viewer)](https://www.npmjs.com/search?q=oh-my-github%2C+viewer)

```json
{
  ...

  "name": "oh-my-github-viewer-default",
  "version": "0.0.1",
  "author": "1ambda",
  "description": "",
  "homepage": "https://github.com/oh-my-github/viewer#readme",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/oh-my-github/viewer.git"
  },
  "bugs": {
    "url": "https://github.com/oh-my-github/viewer/issues"
  },

  ...
}
```

### References

- [@baracktocat on octodex (Title Iamge)](https://octodex.github.com/baracktocat)
- [oh-my-github: generator](https://github.com/oh-my-github/oh-my-github)
- [oh-my-github: viewer](https://github.com/oh-my-github/viewer)
