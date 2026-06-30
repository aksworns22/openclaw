# CLAUDE.md

이 파일은 이 저장소에서 작업할 때 Claude Code(claude.ai/code)에게 제공되는 안내 문서입니다.

## 프로젝트 개요

`learning-notes`는 매일매일의 학습 내용을 정리하는 개인 위키/지식 베이스입니다. [Hugo](https://gohugo.io/) + [hugo-book](https://github.com/alex-shpak/hugo-book) 테마로 빌드됩니다.

- `raw/` — 위키 작성을 위해 수집한 원본 자료(노트, 아티클, 참고 자료 등)
- `content/docs/` — `raw/`의 내용을 바탕으로 정리/작성한 위키 콘텐츠(markdown)
- `content/docs/map.md` — **지식 지도**. front matter의 `concepts`가 개념 그래프의 단일 진실의 원천이며, `layouts/shortcodes/concept-map.html`이 이를 mermaid로 자동 렌더링합니다.
- `themes/hugo-book/` — git submodule. 직접 수정하지 않습니다.
- `layouts/_partials/docs/inject/` — 테마 오버라이드(mermaid 스크립트·custom.css 주입)
- `layouts/shortcodes/` — 커스텀 shortcode(`concept-map` = 지식 지도 렌더러)
- `static/custom.css` — 테마 위에 얹는 커스텀 스타일

## 작업 방식

Claude는 `raw/`에 있는 자료를 읽고, 그 내용을 정리하여 `content/docs/`에 작성합니다.

## 규칙

- `raw/`는 원본 자료 보관용이므로 수정, 삭제, 이동 등 어떠한 변경도 하지 않습니다. 읽기만 합니다.

## wiki 작성 포맷

`content/docs/` 아래의 파일은 markdown(`.md`)으로 작성합니다. markdown으로 표현하기 어려운 인터랙션/시각화 요소는 raw HTML을 그대로 섞어 써도 됩니다(`goldmark.renderer.unsafe = true` 설정됨). 다음 규칙을 따릅니다.

- 파일 상단에 front matter를 둡니다. `tags`는 그날 다룬 모든 주제의 태그를 합쳐 중복 없이 기록하고, `sources`는 그날 참고한 모든 원본 자료 경로를 리스트로 기록합니다:

  ```yaml
  ---
  title: "YYYY-MM-DD"
  date: YYYY-MM-DD
  tags: ["태그1", "태그2"]
  sources: ["raw/ 안의 원본 자료 경로1", "raw/ 안의 원본 자료 경로2"]
  ---
  ```

  이 front matter는 화면에 별도 블록으로 렌더링되지 않습니다(분류·검색·AI 인덱싱용 메타데이터). 손으로 메타 블록 HTML을 작성하지 않습니다.
- 목차는 hugo-book이 자동으로 우측 ToC와 사이드바를 제공하므로 손으로 만들 필요가 없습니다. 헤딩에 `id`를 직접 붙이지 않습니다(Hugo가 자동 생성).
- 보충 설명, 참고 자료 목록, 긴 코드/로그처럼 본문 흐름을 방해하는 내용은 `<details><summary>...</summary>...</details>`로 접어둡니다.
- 계산 그래프, 아키텍처, 파이프라인, 워크플로, 시퀀스(턴 단위 상호작용) 등 구조를 설명하는 내용은 텍스트나 ASCII 다이어그램 대신 [Mermaid](https://mermaid.js.org/)로 실제 렌더링되는 도식을 그립니다. `<div class="mermaid">...</div>` 블록 안에 다이어그램 코드를 작성하면, 테마에 주입된 mermaid 스크립트(`layouts/_partials/docs/inject/head.html`, `footer.html`)가 자동으로 렌더링합니다. 별도 `<script>` 태그를 파일에 추가할 필요가 없습니다.
- 같은 날짜에 작성하는 모든 학습 내용은 하나의 파일로 묶습니다. 파일명: `content/docs/YYYY-MM-DD.md` (예: `content/docs/2026-06-15.md`)
- 같은 날짜에 여러 주제를 다룰 경우, 각 주제는 파일 안에서 `##` 레벨 섹션으로 구분합니다(하위 제목은 그에 맞춰 한 단계씩 내려서 작성: `###`, `####` ...).

## 지식 지도 (`content/docs/map.md`)

날짜 노트는 "그날의 로그"라서 지식이 날짜별로 흩어진다. 이를 연결하는 **개념 축**이 지식 지도다. 개념 그래프 전체는 `content/docs/map.md`의 front matter `concepts` 리스트 **하나에만** 존재한다(단일 진실의 원천). 본문에 mermaid를 손으로 그리지 않는다 — `{{< concept-map >}}` shortcode가 front matter를 읽어 자동 렌더링한다.

각 개념 항목 스키마(필드는 이게 전부 — 단순하게 유지한다):

```yaml
- id: multi-head-attention      # 영문 kebab 식별자. needs에서 이 값으로 참조
  label: "멀티 헤드 어텐션"        # 지도 표시명(생략 시 id)
  status: solid                 # stub | learning | solid (학습 상태 = 색)
  needs: [attention]            # 선행 개념 id 목록(화살표: needs → 이 개념). 없으면 생략
  days: [2026-06-30]            # 이 개념을 다룬 날짜 노트(본문 점프용)
```

**날짜 노트를 새로 쓸 때마다** 그날 다룬 개념을 지도에 반영한다:
- 새 개념이면 항목을 추가하고 `needs`로 선행 개념에 연결한다(고립 노드를 만들지 말 것).
- 이미 있는 개념이면 `days`에 그날 날짜를 추가하고, 더 깊이 다뤘으면 `status`를 올린다(stub→learning→solid).

**AI 토큰 효율 규약**: 개념 간 관계나 "어디서 무엇을 다뤘는지"가 필요한 작업은 **먼저 `map.md`의 front matter만 읽어** 전체 구조·학습 상태·각 개념이 어느 날짜 노트에 있는지(`days`)를 파악한 뒤, 정말 필요한 날짜 노트 본문만 펼쳐 읽는다. 날짜 노트를 통째로 grep하지 않는다. front matter가 가벼운 인덱스 역할을 하도록 항상 최신으로 유지한다.

## 로컬 미리보기

`hugo server` 로 로컬에서 렌더링을 확인할 수 있습니다.

## GitHub Pages 배포

이 저장소는 GitHub Actions(`.github/workflows/hugo.yml`)로 빌드되어 GitHub Pages에 배포됩니다(Pages 소스: Actions). `main`에 push되면 자동으로 빌드/배포되므로, 별도로 인덱스 페이지를 손으로 갱신할 필요가 없습니다(사이드바와 `content/docs/_index.md`가 그 역할을 대체).
