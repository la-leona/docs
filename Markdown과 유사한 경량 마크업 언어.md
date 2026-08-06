User: markdown 과 비슷한 다른 언어는 무엇이 있고 각각을 비교해줘
Kimi: Markdown과 유사한 경량 마크업 언어(Lightweight Markup Language)에는 여러 가지가 있습니다. 주요 언어들의 특징을 비교해 드리겠습니다.

---

## 📋 주요 경량 마크업 언어 비교

| 언어 | 탄생 | 주요 사용처 | 문법 난이도 | 강점 | 약점 |
|------|------|------------|------------|------|------|
| **Markdown** | 2004 | 블로그, README, 문서 | ⭐ 쉬움 | 직관적, 널리 사용됨 | 표준 불일치(플레이버 많음) |
| **reStructuredText** | 2002 | Python 문서, 기술 문서 | ⭐⭐⭐ 어려움 | 강력한 확장성, 문서 구조화 | 진입 장벽 높음 |
| **AsciiDoc** | 2002 | 기술 문서, 책, 매뉴얼 | ⭐⭐ 보통 | Markdown보다 풍부한 기능 | 상대적으로 생태계 작음 |
| **Org-mode** | 2003 | Emacs 사용자, 할 일/노트 | ⭐⭐ 보통 | 할 일 관리, 일정, 테이블 계산 | Emacs 의존성 높음 |
| **Textile** | 2002 | 웹 콘텐츠, 블로그 | ⭐⭐ 보통 | HTML과 유사한 직관성 | 최근 사용 감소 |
| **MediaWiki** | 2002 | 위키백과, 위키 | ⭐⭐ 보통 | 링크, 템플릿 시스템 강력 | 위키 전용에 특화 |
| **BBCode** | 1998 | 포럼, 게시판 | ⭐⭐ 보통 | 안전한 HTML 대체 | 기능 제한적, 구식 느낌 |

---

## 🔍 상세 비교

### 1. **Markdown**
```
# 제목
**굵게** *기울임*
[링크](https://example.com)
```
- **장점**: 가장 직관적, GitHub/GitLab 등에서 기본 지원
- **단점**: CommonMark 외에 GitHub Flavored Markdown, MultiMarkdown 등 **표준 분열**

### 2. **reStructuredText (reST)**
```rst
제목
====
**굵게** *기울임*
`링크 <https://example.com>`_
```
- **장점**: Sphinx 문서화 도구와 결합, 복잡한 문서 구조(목차, 인덱스, 교차 참조)에 최적
- **단점**: 문법이 Markdown보다 복잡하고 엄격함

### 3. **AsciiDoc**
```asciidoc
= 제목
*굵게* _기울임_
https://example.com[링크]
```
- **장점**: Markdown 문법을 거의 그대로 쓰면서 **테이블, Footnote, Include, 조건부 처리** 등 고급 기능 제공
- **단점**: 도구 체인(Asciidoctor) 추가 설치 필요

### 4. **Org-mode**
```org
* 제목
*굵게* /기울임/
[[https://example.com][링크]]
```
- **장점**: **할 일(TODO), 일정(Agenda), 테이블 계산, 코드 실행** 등 노트+프로젝트 관리 통합
- **단점**: Emacs 외 에디터에서 완벽한 지원 어려움

### 5. **MediaWiki**
```mediawiki
= 제목 =
'''굵게''' ''기울임''
[https://example.com 링크]
```
- **장점**: 위키 내부 링크 `[[문서명]]`, 템플릿, 분류 시스템이 매우 강력
- **단점**: 일반 문서 작성보다는 위키 전용

---

## 🎯 선택 가이드

| 상황 | 추천 언어 |
|------|----------|
| GitHub README, 간단한 문서 | **Markdown** |
| Python 프로젝트 문서화 | **reStructuredText** + Sphinx |
| 기술 서적, 복잡한 매뉴얼 | **AsciiDoc** |
| Emacs 사용자의 노트/할 일 관리 | **Org-mode** |
| 위키 시스템 구축 | **MediaWiki** |
| 포럼/게시판 글쓰기 | **BBCode** |

Markdown이 가장 보편적이지만, **복잡한 문서 구조**가 필요하면 AsciiDoc이나 reStructuredText를, **Emacs 생태계**에 있다면 Org-mode가 훨씬 강력한 대안이 됩니다.
