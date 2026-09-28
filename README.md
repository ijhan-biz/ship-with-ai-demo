# Ship with AI

**초기 데모 기준 저장소입니다. 의도된 보안 문제가 남아 있으며 현재 배포·에이전트 전체 흐름의 완료를 보장하지 않습니다. 운영 환경에 그대로 사용하지 마세요.**

**AI Genius — 시즌 5 에피소드 3: "Ship with AI: AI로 완성하는 코드 리뷰부터 보안, 배포까지"**의
학습 사이트이자 라이브 데모 저장소입니다.

이 저장소에는 두 가지 역할이 있습니다.

1. **학습 콘텐츠 제공** — AI 지원 배포 파이프라인을 설명하는 작은 Astro 사이트입니다.
   이슈 → Copilot의 풀 리퀘스트(PR) 초안 → Copilot Code Review → Agent Merge →
   GitHub Actions(빌드 + 공급망 보안) → GitHub Pages 흐름을 다룹니다.
2. **데모 대상** — 라이브 방송에서 검토하고 보안을 개선한 뒤 배포할 코드, 의존성,
   워크플로가 들어 있습니다. 현재 저장소는 개선 전 상태입니다.

데모 전반의 보안 주제는 **OWASP Top 10:2025 A03 — 소프트웨어 공급망 실패**입니다.
오래된 의존성, commit SHA로 고정되지 않은 CI/CD Action과 과도한 토큰 권한,
커밋된 데모용 가짜 시크릿을 다룹니다. 전체 진행 순서는 [`RUNSHEET.md`](./RUNSHEET.md),
쉬운 설명은 [`src/pages/secure-supply-chain.astro`](./src/pages/secure-supply-chain.astro)를
참고하세요.

## 사전 준비 — 녹화 전에 설정하세요

저장소 파일만으로는 알기 어려운 설정들입니다. 빠진 항목이 있으면 해당 데모 단계가
실행되지 않을 수 있으므로 사전에 확인하세요. 사용 가능한 기능과 UI는 요금제, 조직 정책,
환경과 권한에 따라 다를 수 있습니다.

### 계정과 라이선스

- **코드 리뷰와 코딩 에이전트를 포함한 유효한 Copilot 라이선스**가 필요합니다.
  Copilot Pro, Pro+, Max, Business, Enterprise 등 사용 중인 요금제의 기능 제공 여부를
  확인하세요. 이 데모의 해당 기능에는 Copilot Free만으로 충분하지 않습니다.
- 프로필의 **설정 → Copilot → 기능(Settings → Copilot → Features)**에서
  다음 두 항목이 **사용(Enabled)** 상태인지 확인하세요.
  - **Copilot 코드 리뷰(Copilot code review)** — Copilot으로 코드를 검토하고 PR 요약을 생성합니다.
  - **Copilot 클라우드 에이전트(Copilot cloud agent)** — 활성화된 저장소에서 작업을
    Copilot 클라우드 에이전트에 위임합니다. 코딩 에이전트와 Agent Merge 시나리오에 사용합니다.
  - 조직이나 기업에서 라이선스를 제공한다면 방패 아이콘이 표시될 수 있습니다.
    조직이 설정을 강제한다는 뜻으로, 기능이 고장 난 것이 아니라 직접 끌 수 없다는 의미입니다.

### 저장소 설정 — 소유자 또는 관리자 권한 필요

- **설정 → 규칙 → 규칙 집합 → 새 규칙 집합(Settings → Rules → Rulesets → New ruleset)** —
  기본 브랜치에 **Copilot 코드 리뷰 자동 요청(Automatically request Copilot code review)**
  규칙을 추가하세요. 자동 리뷰 규칙이 없으면 PR마다 직접 리뷰를 요청해야 할 수 있습니다.
- **설정 → 일반 → 풀 리퀘스트 → 자동 병합 허용(Settings → General → Pull Requests → Allow auto-merge)** —
  자동 병합 시나리오에 필요합니다. 검사를 통과해도 이 설정과 필요한 권한이 없으면 자동으로
  병합되지 않을 수 있습니다. API로도 설정할 수 있습니다.
  `gh api -X PATCH /repos/<owner>/<repo> -f allow_auto_merge=true`
- **설정 → Pages → 소스(Settings → Pages → Source): GitHub Actions** —
  `deploy.yml`에서 사이트를 게시하려면 필요합니다. 이 설정이 빠지면 `build`가 성공해도
  `deploy` 작업에서 404 유형의 오류가 발생할 수 있습니다. API로도 설정할 수 있습니다.
  `gh api -X POST /repos/<owner>/<repo>/pages -f build_type=workflow`
- **설정 → 코드 보안(Settings → Code security)** — Dependabot 알림(Dependabot alerts),
  Dependabot 보안 업데이트(Dependabot security updates), 의존성 검토(Dependency review),
  코드 검사(Code scanning), 시크릿 검사(Secret scanning)와 푸시 보호(Push protection)를
  활성화하세요. `RUNSHEET.md`의 §5.1과 §5.3에 해당하는 데모 문제를 확인할 때 사용합니다.
  탐지 결과는 설정과 지원되는 패턴에 따라 달라지며, 데모용 가짜 시크릿이 반드시 탐지되는 것은 아닙니다.

### 로컬 `gh`/git 인증 시 주의 사항

`git push`가 "`workflow` 범위 없이 OAuth 앱이 `.github/workflows/...` 워크플로를
만들거나 업데이트할 수 없다"는 오류(*"refusing to allow an OAuth App to create or update
workflow ... without `workflow` scope"*)로 거부되면 현재 `gh`/git 자격 증명에
`workflow` OAuth 범위가 없는 것입니다. `gh auth refresh -h github.com -s workflow`를
실행하거나, 이미 해당 범위가 있는 자격 증명이나 토큰으로 푸시하세요.
예를 들어 `workflow` 범위가 있는 계정의 `gh auth token` 결과를
`git -c http.extraHeader=...`로 전달하는 방식이 있습니다. 토큰을 공유하거나 로그에 남기지 마세요.

## 로컬에서 실행하기

아래 주소는 이 한국어 데모 저장소입니다. 원본 자료는
[anothergeorgecoldham/ship-with-ai](https://github.com/anothergeorgecoldham/ship-with-ai)를
참고하세요. 두 저장소는 별도의 독립 사본입니다.

```bash
git clone https://github.com/ijhan-biz/ship-with-ai-demo.git
cd ship-with-ai-demo
npm install
npm run dev
```

터미널에 표시된 로컬 URL을 여세요. `npm run build`는 `dist/`에 정적 사이트를 생성합니다.
`npm run preview`는 빌드 결과를 로컬에서 제공합니다. 원격 저장소나 워크플로 실행,
의도적으로 남긴 데모 문제를 변경하지 않고 사이트를 확인할 수 있습니다.

## 구조

```
src/
  pages/            콘텐츠 페이지(홈, 파이프라인, 기능별 안내, 공급망 보안, 직접 해보기)
  components/
    FeedbackWidget.astro   유일한 대화형 기능 — 질문과 피드백, 클라이언트에서만 실행
  lib/                위젯 로직(제출 처리 함수 + 데모용 가짜 설정)
astro.config.mjs      정적 출력, GitHub Pages 프로젝트 사이트용 `site`/`base` 설정
.github/
  workflows/deploy.yml     의존성 설치 → 보안 게이트 → 사이트 빌드 → Pages 배포
  dependabot.yml           npm + GitHub Actions 버전 업데이트
  ISSUE_TEMPLATE/feature-request.md   라이브 데모를 시작하는 이슈 템플릿
```

## 배포하기

`main`에 푸시하면 `.github/workflows/deploy.yml`의 **Build and deploy** 워크플로가 실행됩니다.
보안 게이트를 통과하면 사이트를 빌드하고 GitHub Pages에 배포하는 구성입니다.
저장소의 **Pages → 소스(Source): GitHub Actions** 설정이 필요합니다.
현재 상태에서는 의도적으로 남긴 오래된 의존성 때문에 `npm audit` 보안 게이트가 실패할 수 있습니다.
워크플로에는 라이브 데모용 CI/CD 문제 두 가지도 그대로 남아 있습니다. `RUNSHEET.md`를 참고하세요.
로컬 빌드 성공이 원격 배포 성공을 뜻하지는 않습니다.

이 한국어판은 문구만 바꿉니다. 기존 `site`/`base` 설정과 `marked@0.3.19`를 그대로
보존하므로, 잘못된 공개 Pages 경로와 피드백 렌더링의 기존 오류가 해결된 것은 아닙니다.
배포 가능한 수정본과 구분해서 사용하세요.

## 다른 언어나 지역에서 데모 진행하기

`RUNSHEET.md`에는 단계별 진행 대본, 등록할 이슈 본문, 도구별로 확인할 문제를 담았습니다.
설명된 에이전트 동작과 병합은 목표 시나리오이며 관찰된 완료 결과가 아닙니다.
이 저장소의 문제를 미리 수정하지 마세요. 녹화를 시작할 때도 의도적으로 남긴 데모 문제가
존재해야 합니다.
