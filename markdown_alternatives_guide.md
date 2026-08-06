# Markdown과 유사한 마크업 언어 비교 가이드

Markdown은 가장 널리 사용되는 경량 마크업 언어이지만, 더 강력한 기능이 필요한 경우 다른 마크업 언어들을 고려할 수 있습니다. 이 가이드에서는 Markdown과 유사하면서도 각각의 특징을 가진 마크업 언어들을 비교합니다 [1] [2] [3] [4].

## 1. Markdown

### 1.1 개념 및 특징

**Markdown**은 가장 간단하고 널리 사용되는 경량 마크업 언어입니다:

*   **단순성**: 배우기 쉽고 읽기 쉬운 구문
*   **가독성**: 소스 형태에서도 거의 일반 텍스트처럼 보임
*   **빠른 작성**: 최소한의 마크업으로 빠르게 작성 가능
*   **광범위한 지원**: GitHub, GitLab 등 많은 플랫폼에서 지원

### 1.2 장점

*   배우기 매우 쉬움
*   많은 도구와 플랫폼에서 지원
*   빠른 문서 작성
*   README 파일에 이상적

### 1.3 단점

*   기능 제한적 (표, 각주, 교차 참조 등)
*   복잡한 문서 구조 표현 어려움
*   많은 "flavor" (변형) 존재로 인한 호환성 문제
*   고급 기능 필요 시 HTML 혼용 필요

## 2. reStructuredText (RST)

### 2.1 개념 및 특징

**reStructuredText (RST)**는 Python 커뮤니티에서 개발한 마크업 언어입니다:

*   **명확한 사양**: 공식 명세 문서 존재
*   **Sphinx 통합**: 문서 생성 도구 Sphinx와 완벽 통합
*   **Python 기반**: Python 프로젝트 문서화에 최적
*   **강력한 기능**: Markdown보다 더 많은 기능 제공

### 2.2 주요 특징

| 항목 | 설명 |
| :--- | :--- |
| **공백 처리** | 공백과 들여쓰기가 중요 (Python처럼) |
| **섹션 구분** | 밑줄 문자로 섹션 구분 |
| **코드 블록** | `.. code-block:: language` 문법 |
| **주석** | `.. 주석 내용` 형식 |
| **파일 포함** | `.. include:: filename.rst` 지원 |
| **교차 참조** | `:ref:` 지시문으로 참조 가능 |

### 2.3 문법 예시

```rst
# 섹션 제목
==============

*이탤릭*
**굵은체**
``고정폭 폰트``

.. code-block:: python

   def hello():
       print("Hello World")

.. note::
   이것은 주석입니다.

.. include:: other_file.rst
```

### 2.4 장점

*   명확한 공식 사양 존재
*   Sphinx와의 완벽한 통합
*   복잡한 문서 구조 표현 가능
*   교차 참조 및 자동 생성 기능
*   Python 커뮤니티에서 널리 사용

### 2.5 단점

*   공백과 들여쓰기가 중요해서 학습 곡선 가파름
*   Markdown보다 복잡한 문법
*   초보자에게 어려울 수 있음
*   도구 지원이 Markdown보다 제한적

## 3. AsciiDoc

### 3.1 개념 및 특징

**AsciiDoc**은 DocBook XML을 위한 경량 마크업 언어입니다:

*   **출판 중심**: 인쇄 및 웹 출판을 위해 설계
*   **일관된 문법**: 일관된 형식 체계
*   **확장성**: 핵심 기능으로 확장 가능
*   **강력함**: Markdown과 RST보다 더 강력한 기능

### 3.2 주요 특징

| 항목 | Markdown | AsciiDoc |
| :--- | :--- | :--- |
| **굵은체** | `**bold**` | `*bold*` |
| **이탤릭** | `*italic*` | `_italic_` |
| **링크** | `[text](url)` | `url[text]` |
| **이미지** | `![alt](image.png)` | `image::image.png[alt]` |
| **코드 블록** | `` ```language `` | `[source,language]` |
| **표** | 제한적 | 강력함 |
| **각주** | 지원 안 함 | 지원 |
| **교차 참조** | 지원 안 함 | 지원 |

### 3.3 문법 예시

```asciidoc
= 문서 제목
:author: 작성자

== 섹션 제목

*굵은체*
_이탤릭_
`고정폭 폰트`

https://example.com[링크 텍스트]

image::logo.png[로고, width=200]

[source,python]
----
def hello():
    print("Hello World")
----

[NOTE]
====
이것은 주석입니다.
====

[cols="1,1"]
|===
|헤더 1 |헤더 2
|셀 1 |셀 2
|===
```

### 3.4 장점

*   Markdown보다 더 강력한 기능
*   일관된 문법 체계
*   표, 각주, 교차 참조 등 고급 기능
*   출판 품질의 문서 생성 가능
*   DocBook 변환 지원
*   Markdown 호환성 있음

### 3.5 단점

*   Markdown보다 학습 곡선이 가파름
*   도구 지원이 Markdown보다 적음
*   초보자에게 복잡할 수 있음

## 4. LaTeX

### 4.1 개념 및 특징

**LaTeX**는 강력한 조판 언어입니다:

*   **수학 공식**: 복잡한 수학 방정식에 최적
*   **학술 출판**: 학술 논문 및 책 출판에 사용
*   **전문적 품질**: 출판 품질의 문서 생성
*   **정밀한 제어**: 세밀한 레이아웃 제어 가능

### 4.2 문법 예시

```latex
\documentclass{article}
\usepackage[utf-8]{inputenc}

\title{문서 제목}
\author{작성자}
\date{\today}

\begin{document}

\maketitle

\section{섹션 제목}

\textbf{굵은체}
\textit{이탤릭}
\texttt{고정폭 폰트}

\[
E = mc^2
\]

\begin{itemize}
  \item 항목 1
  \item 항목 2
\end{itemize}

\end{document}
```

### 4.3 장점

*   복잡한 수학 공식 표현 최고
*   학술 출판에 표준
*   전문적 품질의 문서
*   매우 강력한 기능

### 4.4 단점

*   매우 가파른 학습 곡선
*   복잡한 문법
*   간단한 문서에는 과도함
*   소스 형태에서 읽기 어려움

## 5. Org-mode

### 5.1 개념 및 특징

**Org-mode**는 Emacs 편집기의 마크업 언어입니다:

*   **Emacs 기반**: Emacs 편집기와 완벽 통합
*   **다목적**: 문서, 할 일 목록, 프로젝트 관리
*   **강력한 기능**: 코드 실행, 테이블, 할 일 추적
*   **유연성**: 다양한 출력 형식 지원

### 5.2 문법 예시

```org
* 섹션 제목
** 하위 섹션

*굵은체*
/이탤릭/
=고정폭 폰트=

[[https://example.com][링크 텍스트]]

#+BEGIN_SRC python
def hello():
    print("Hello World")
#+END_SRC

| 헤더 1 | 헤더 2 |
|--------|--------|
| 셀 1   | 셀 2   |

- [ ] 완료되지 않은 작업
- [x] 완료된 작업
```

### 5.3 장점

*   다목적 도구 (문서, 할 일, 프로젝트 관리)
*   강력한 기능
*   Emacs 사용자에게 최적
*   코드 실행 가능

### 5.4 단점

*   Emacs 사용자 중심
*   Emacs 없이는 사용 어려움
*   학습 곡선 가파름
*   일반적인 도구 지원 부족

## 6. 비교표

| 항목 | Markdown | RST | AsciiDoc | LaTeX | Org-mode |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **학습 난이도** | 매우 쉬움 | 중간 | 중간 | 어려움 | 어려움 |
| **기능성** | 기본 | 강함 | 매우 강함 | 매우 강함 | 강함 |
| **표 지원** | 제한적 | 좋음 | 우수 | 우수 | 우수 |
| **수학 공식** | 제한적 | 제한적 | 제한적 | 우수 | 좋음 |
| **교차 참조** | 없음 | 우수 | 우수 | 우수 | 우수 |
| **도구 지원** | 매우 많음 | 많음 | 중간 | 많음 | 적음 |
| **플랫폼 지원** | 매우 광범위 | 광범위 | 중간 | 광범위 | 제한적 |
| **커뮤니티** | 매우 큼 | 중간 | 중간 | 큼 | 작음 |
| **최적 용도** | README, 블로그 | 기술 문서 | 출판 | 학술 논문 | 할 일, 노트 |

## 7. 사용 시나리오별 선택 가이드

### 7.1 Markdown을 선택해야 할 때

*   **README 파일**: GitHub, GitLab 등에서 자동 렌더링
*   **블로그 글**: 간단하고 빠른 작성
*   **간단한 문서**: 기본 포맷만 필요
*   **초보자**: 배우기 쉬운 언어 필요
*   **웹 콘텐츠**: 많은 웹 플랫폼에서 지원

### 7.2 reStructuredText를 선택해야 할 때

*   **Python 프로젝트**: Sphinx와의 완벽 통합
*   **기술 문서**: 복잡한 구조의 문서
*   **자동 생성**: API 문서 자동 생성
*   **교차 참조**: 문서 간 참조 필요
*   **공식 사양**: 명확한 표준 필요

### 7.3 AsciiDoc을 선택해야 할 때

*   **출판 품질 문서**: 인쇄 및 웹 출판
*   **복잡한 구조**: 고급 포맷 필요
*   **확장성**: 커스텀 기능 필요
*   **일관된 문법**: 체계적인 마크업 선호
*   **DocBook 변환**: 엔터프라이즈 출판 워크플로우

### 7.4 LaTeX를 선택해야 할 때

*   **학술 논문**: 학술 출판 표준
*   **수학 공식**: 복잡한 수학 표현
*   **책 출판**: 전문적 품질의 책
*   **정밀한 제어**: 세밀한 레이아웃 필요
*   **과학 논문**: 과학 커뮤니티 표준

### 7.5 Org-mode를 선택해야 할 때

*   **Emacs 사용자**: Emacs 환경에서 작업
*   **다목적 도구**: 문서, 할 일, 프로젝트 관리
*   **코드 실행**: 문서에 코드 포함 및 실행
*   **개인 노트**: 개인 노트 및 일정 관리
*   **복잡한 프로젝트**: 프로젝트 전체 관리

## 8. 마이그레이션 경로

```
Markdown (시작)
    ↓
복잡한 기능 필요?
    ├─ 기술 문서 → reStructuredText
    ├─ 출판 품질 → AsciiDoc
    ├─ 수학 공식 → LaTeX
    └─ 할 일 관리 → Org-mode
```

## 9. 결론

| 사용자 유형 | 추천 언어 | 이유 |
| :--- | :--- | :--- |
| **초보자** | Markdown | 배우기 쉬움 |
| **Python 개발자** | reStructuredText | Sphinx 통합 |
| **기술 작가** | AsciiDoc | 강력한 기능 |
| **학술 연구자** | LaTeX | 수학 공식 |
| **Emacs 사용자** | Org-mode | 완벽 통합 |
| **웹 개발자** | Markdown | 광범위한 지원 |

**최종 추천**:
- **일반적인 경우**: **Markdown** (간단하고 빠름)
- **기술 문서**: **reStructuredText** 또는 **AsciiDoc** (강력한 기능)
- **학술 출판**: **LaTeX** (표준)
- **개인 생산성**: **Org-mode** (다목적)

## References

1.  [Elements Of a Great Markup Language - matklad](https://matklad.github.io/2022/10/28/elements-of-a-great-markup-language.html)
2.  [Compare AsciiDoc to Markdown - Asciidoctor](https://docs.asciidoctor.org/asciidoc/latest/asciidoc-vs-markdown/)
3.  [Markup lowdown: 4 markup languages every team should know - Opensource.com](https://opensource.com/life/15/8/markup-lowdown)
4.  [ReStructured Text for those who know Markdown - Open MPI](https://docs.open-mpi.org/en/v5.0.x/developers/rst-for-markdown-expats.html)
