
# 📚 개발 문서

프로젝트 기술 문서 모음입니다.

[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-success)](https://jyshinAnt.github.io/note/)

## 🌐 문서 사이트

**👉 [https://jyshinAnt.github.io/note/](https://jyshinAnt.github.io/note/)**

검색 기능과 깔끔한 레이아웃으로 문서를 쉽게 탐색할 수 있습니다.

---

## 📑 목차

### API 명세서
- [관리자 구매내역 조회](docs/관리자_구매내역_조회.md) - 관리자 주문 조회 API 상세 명세

### 개발 가이드
- [Java Spring FCM 셋팅](docs/java_fcm_setting.md) - Firebase Cloud Messaging 설정 가이드
- [내부정기권 사용량 집계 (v1)](docs/내부정기권_사용량_집계.md) - 내부정기권 사용량 집계 로직
- [내부정기권 사용량 집계 (v2)](docs/내부정기권_사용량_집계_v2.md) - 내부정기권 사용량 집계 로직 v2

---

## 🛠️ 로컬에서 문서 보기

```bash
# Jekyll 설치 (macOS)
brew install ruby
gem install bundler jekyll

# 로컬 서버 실행
bundle install
bundle exec jekyll serve

# 브라우저에서 http://localhost:4000/note/ 접속
```

---

## ✨ 기능

- 📝 **마크다운 지원**: GitHub Flavored Markdown 완벽 지원
- 🔍 **전체 검색**: 모든 문서에서 키워드 검색 가능
- 📱 **반응형 디자인**: 모바일, 태블릿, 데스크톱 모두 지원
- 🎯 **목차 자동 생성**: 각 문서의 헤딩 구조 자동 파싱
- 🌙 **다크 모드**: 자동/수동 다크 모드 전환
