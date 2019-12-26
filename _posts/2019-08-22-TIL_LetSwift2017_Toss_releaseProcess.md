---
title: "LetSwift 2017 토스의 개발/배포 환경 - 손민탁님"
layout: post
date: 2019-08-22 19:34
image: false
headerImage: false
tag:
-

category: blog
author: jack
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

# 2019/08/22
## LetSwift 2017
### 토스의 개발/배포 환경 - 손민탁님
- [영상링크](https://youtu.be/338FdLzGhhY)
- [발표자료](https://www.slideshare.net/MintakSon/ios-80115427)

#### 풀고자 한 문제
- 스킴, 프로젝트, 워크스페이스 파일관리는 생각만해도 머리가 아프고, 치가 떨림
- 프로젝트 파일(ex. -objc, -force_load, $(interited), OTHER_LDFLAGS)을 볼 때마다 로딩시간이 너무 김
- 베타서버용 테스트 앱을 빌드해주는 경우.
  - 현재 토스는, Toss, Toss Alpha, Toss Staging, Toss Dogspaw 4가지 로 구성되어 있음.
    - Toss: 실서버. 앱스토어용과 실서버 사내배포
    - Toss Alpha: 알파서버
    - Toss Staging: 스테이징 서버
    - Toss Dogspaw: 개발 서버
  - 위 버전들은 각각 Debug & Relase로 나뉘고, 보안모듈 여부가 다르다.
  - 위 문제를 어떻게 해결할까?를 고민했음.
    1. 앱 쪼개기: 4개의 서벼별로 앱 만들기, 관리, 유지보수하기 쉽게
    2. 제휴 SDK 관리하기: .xcpdeproj 최대한 덜 건드리기, 제휴 SK 이력 업데이트 ㅗ간리 쉽게
    3. 배포 쉽게하기: fastlane, fabric 베타

#### .xcconfig로 프로젝트 설정하기
- Target vs .xcscheme?
  - WWDC 2014 Videos - `sharing code between iOS and OS X`을 확인!
  - .xcconfig를 subclass + composition을 사용한 것임.
  - 비공식 문서도 많은 부분이었음. 많은 삽질 끝에 알아낸 방법을 공유하고자 함.
- 방법: *발표자료를 꼭 확인하기*
  1. 반영 방식 파악하기
  2. 기존 설정 뽑아내기
     - buildingSettingExtractor라는 오프소스 활용.
  3. 파일 이해하기
     - `#include`가 가능한 계층구조임
  4. xcconfig 반영하기
     - pods를 바꿔주는게 핵심!
     - pod이 만들어준 xcconfig 말고, 직접 만든 xcconfig 지정
     - 구체적인 파일명은?
  5. 주의할 점
     - cocoapods을 썼으면 꼭 .xcconfig에 Pods의 .xcconfig를 `#include` 해야한다.
     - //가 주석으로 인식이 된다. 그래서 hack을 써야함.
     - 뽑아낸 설정 중에서 필요한 것만 써야함.

#### private cocoa pods 설정하기
- cocoapods을 쓰지 않는 제휴사 SDK/framework를 활용하기 위해 private cocoapods을 만들었음.
- 방법 *발표자료 확인

#### fastlane + beta 설정하기
- Fastlane - Felix Krause가 만듬.
- 방법 *발표자료 확인

#### Reference
- 발표자료 확인
