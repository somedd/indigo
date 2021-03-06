---
title: "[TIL] Git & Github의 기초사용법"
layout: post
date: 2017-10-20 15:00
image: false
headerImage: false
tag:
- Git
- Github

category: blog
author: jack
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---
# 2017/10/20
## Git & Github의 기초사용법

## #1. Git 시작하기
- 2005년 Linus Torvals가 리눅스의 소스 코드 버전 관리용으로 만듬
- VCS(Version Control System)
  - 왜 VCS를 사용할까?
    - 소스를 어디에 저장할까?
    - 협업시 충돌 해결은?
    - 문제가 생겼을 때 어떻게 되돌릴까?
    - 오픈 소스?

### Git의 장단점
- 장점
  - 오프라인 작업이 가능.
  - 풍부한 브랜치 기능
  - WorkFlow가 좋다!!!
- 단점
  - 어려움.

### 워킹 트리(디렉토리) = 작업 디렉토리
- 내가 현재 작업하고 있는 디렉토리
- .git이 있는 디렉토리를 워킹 트리라 한다.

### Git 명령어
1. clone: 프로젝트를 원격 저장소 -> 로컬 저장소로 불러옴
2. init
3. add: file들을 Stage에 올린다.
4. commit: Stage에 있는 ***파일들을 묶어서 로컬 저장소(.git)에 저장*** 한다. 내 워킹트리에 SnapShot이라고 한다.
5. reset: 이전 커밋이 참조 리스트에서 사라짐.  이전 커밋으로 되돌릴 때 사용
6. revert: 이전 커밋은 남아있고 새로운 커밋을 만듬.  이전 커밋으로 되돌릴 때 사용
7. checkout: 이전 커밋으로 되돌릴 때 사용. 헤드만 가져옴. (detached HEAD)
  - branch는 commit의 참조변수임.
8. rebase: 현재 브랜치에서 대상 브랜치와 차이가 나는 커밋을 가져옴
9. push: 로컬 저장소를 원격저장소로 올리는 것. 한 저장소의 변경사항만 올린다.
10. pull: 원격저장소를 로컬 저장소애 복사하는는 것. (fetch + merge)
  - 현재 작업 디렉토리에 마지막 커밋 내용을 반영
11. fetch: 저장소에 없는 커밋들을 가져온다.

## #2. Branch
### Branch
- 여러 커밋을 트리 형태로 관리할 수 있게 해줌
- 브랜치를 이용해 워크 플로우 관리를 쉽게 할 수 있음
- branch는 단순히 커밋의 참조이다
- 현재 작업중인 브랜치 = HEAD

### HEAD
- 현재 작업중인 브랜치의 참조
- 결국에는 마지막 커밋을 가르키게 된다.
- HEAD가 직접 커밋을 가르킬수도 있다. = detached HEAD [비추!]

### HEAD와 커밋
- 커밋은 반드시 부모를 가지고 있음
- 새로 생성하는 커밋의 부모는 HEAD임

### 커밋의 절차
- 커밋 객체 생성
- 새로운 커밋의 부모를 HEAD로 변경
- 새로운 커밋이 HEAD가 됨

## #3. Tag
```
git tag 태그이름 [커밋]
git push origin 태그이름
```
- 태그도 참조의 일종, 보통 버전을 태그해준다.
- 태그는 갱신되지 않는다.
- 태그와 브랜치 이름을 같게 하면 위험
