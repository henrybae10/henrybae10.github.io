---
title: 'Hugo Guide'
date: '2026-04-15T10:08:32+09:00'
draft: false
categories: ["개발"]
tags: ["개발", "블로그"]
---

## Hugo
 GitHub Pages는 HTML 파일만 서비스하는 웹 호스팅 서비스다. GitHub Pages(이하 GHP)와 연동된 Repository를 Public으로 세팅한다면 무료로 사용할 수 있어 사용하는 사람이 많다.
 GHP의 기본 동작 방식은 사용자가 md 파일을 작성하여 GH에 Push하면 내장된 Jekyll(지킬, 일종의 정적 사이트 생성기)이 md 파일을 HTML로 컴파일하여 GHP 웹서버로 배포해준다.
 하지만 최근에는 Jekyll 대신 Hugo를 많이 사용한다고 한다. Jekyll에 비해 장점이 많기 때문이다. 하지만 Hugo는 GHP에 내장돼있지 않기 때문에 GitHub Actions라는 지시어를 통해 Push가 실행됐을 때 Hugo를 통해 md 파일을 HTML 파일로 컴파일하고 GHP 웹 서버로 배포하라는 지시를 해줘야한다.
 (이런 면에서 md는 HTML을 위한 일종의 날것의 파일이라 생각이 된다.)

|  | Hugo | Jekyll |
| 기반 언어 | Go | Ruby |
| 빌드 속도 | 빠름 | 글이 수백개면 느려짐 |
| 의존성 | 단일 바이너리라 실행 파일 하나면 세팅이 끝남 | Gem 의존성 등 설치할게 많다 |
| 기능 | 이미지 리사이징, 다국어 지원 등 자체 기능이 많음 | 플러그인으로 다 해결해야 함 |

### 로컬 환경에서 실행
Hugo 블로그 코드가 있는 경로로 간 후 (hugo.toml 있는 경로) 아래 명령어를 실행하면 미리보기를 위해 Hugo에 내장된 내장 웹 서버가 실행된다. 그러면서 HTML 파일도 생성됨.

```bash
hugo server -D
```

### Git에 글 올리기
 Repository의 깔끔한 관리를 위해 public은 제외처리한다. (GitHub Actions를 활용하지 않으려면 그냥 public 파일들만 배포해도 될듯.)

```bash
# 1. git 초기화 (이미 .git 폴더가 있다면 생략 가능)
git init
git add .
git commit -m "commit message"
git branch -M main
git remote add origin https://github.com/henrybae10/henrybae10.github.io.git
git push -u origin main
```

### Git에 글 올리면 자동 배포되게 설정
 위와 같이 Git에 md 파일로 글을 작성해서 push 하면 마크다운이나 설정 파일 같은 원본 소스 파일만 업로드가 된다. 이러면 HTML이 생성되지 않았기 때문에 내 주소로 접근할 경우 블로그처럼 보여지지 않고 날 것의 md 파일을 보여준다. GitHub Pages는 기본적으로 저장소 루트에 있는 index.html을 찾아서 보여주는건데 이 HTML이 없어서 문제가 발생하는 것이다. 이 문제를 해결하기 위해 아래와 같은 과정을 따른다.

#### 1. 로컬 환경에서 Hugo 빌드를 한다. (HTML을 바로 배포하고 싶을 때)
빌드를 통해 public 경로가 생기며 여기에 md 파일들이 HTML로 변환된 결과물이 생긴다.
```bash
hugo
```

#### 2-1. GitHub Pages 설정 변경
GitHub Actions를 킨다. GitHub ACtions란 특정 이벤트(여기서는 GitHub Repository에 Push)가 발생했을 때 GitHub 서버가 미리 정해둔 작업(빌드, 테스트, 배포)를 자동으로 실행해주는 기능이다. 
Repository 메뉴 -> Settings -> Pages -> Build and deployment -> Source -> GitHub Actions 선택


#### 2-2. Hugo 자동 배포 설정
위에서 언급한 GitHub Actions에서 어떤걸 해야할지 지시해줘야 할 파일이 필요하다. 그걸 만들어야 한다. GitHub이 md 파일을 받으면 자동으로 HTML로 변환하여 배포하도록 설정 파일을 만든다.
1. .github/workflows/hugo.yaml 파일 생성
2. hugo.yaml
```YAML
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        run: hugo --minify
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

#### 4. Push