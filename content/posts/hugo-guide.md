---
title: 'Hugo Guide'
date: '2026-04-15T10:08:32+09:00'
draft: false
categories: ["개발"]
tags: ["개발", "블로그"]
---

### 로컬 환경에서 실행
Hugo 블로그 코드가 있는 경로로 간 후 (hugo.toml 있는 경로)

```bash
hugo server -D
```

### Git에 글 올리기
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
 Git에 md 파일로 글을 작성해서 push 하면 마크다운이나 설정 파일 같은 원본 소스 파일만 업로드가 된다. 이러면 HTML이 활성화되지 않았기 때문에 내 주소로 접근할 경우 블로그처럼 나타나지 않는다. GitHub Pages는 기본적으로 저장소 루트에 있는 index.html을 찾아서 보여주는건데 이 HTML이 없어서 문제가 발생하는 것이다.

 이 문제를 해결하기 위해 아래와 같은 과정을 따른다.

#### 1. 로컬 환경에서 Hugo 빌드를 한다.
빌드를 통해 public 경로가 생기며 여기에 md 파일들이 HTML로 변환된 결과물이 생긴다.
```bash
hugo
```

#### 2. GitHub Pages 설정 변경
GitHub Actions를 킨다.
Repository 메뉴 -> Settings -> Pages -> Build and deployment -> Source -> GitHub Actions 선택


#### 3. Hugo 자동 배포 설정
GitHub이 md 파일을 받으면 자동으로 HTML로 변환하여 배포하도록 설정 파일을 만든다.
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