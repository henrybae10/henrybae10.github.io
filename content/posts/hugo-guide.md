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