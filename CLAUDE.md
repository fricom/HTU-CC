# HTU-CC — Claude Code 부트캠프 개인 PC 총정리

> 부트캠프 수강생 본인이 보기 위한 정리 사이트. Jekyll로 GitHub Pages에 배포.

## 개요
- **한 줄 설명**: Claude Code 부트캠프 4단계(기본 세팅 → 아키텍처 → CC 확장 → 자동화 워크플로우)를 한 문서에 정리한 개인 위키
- **대상 OS**: Apple Silicon Mac(M1+) 기준, Windows(WSL) 병기
- **레포**: github.com/fricom/HTU-CC (`main` 브랜치)
- **로컬 경로**: `/Users/60hertz/projects/HTU-CC`
- **공개 형태**: GitHub Pages (Jekyll 자동 빌드)

## 기술 스택
- **Jekyll** + `jekyll-theme-cayman` 테마
- 콘텐츠는 `README.md` 단일 파일에 거의 다 들어있음 (~40KB)
- 커스텀 레이아웃은 `_layouts/` 아래
- 이미지·자산은 `assets/`

## 디렉터리
```
_config.yml      # 테마·사이트 메타 (theme: jekyll-theme-cayman)
_layouts/        # 커스텀 레이아웃 (테마 오버라이드)
assets/          # 이미지·CSS
README.md        # 본문 (사이트 홈)
```

## 부트캠프 4단계 (콘텐츠 구조)
| 단계 | 무엇을 얻는가 | 핵심 도구 |
|---|---|---|
| 1 | "CC로 앱을 만들고 배포까지" 첫 경험 | Claude Code · Git · GitHub · Vercel · Cursor |
| 2 | 아키텍처 감각 | Firebase Firestore · Auth · API · WebSocket |
| 3 | CC 자체를 확장 | Skill · Command · Hook · MCP · Plugin |
| 4 | 자동화 워크플로우 | tmux · 하네스 · /loop · wj-magic · papyrus · hermes |

## 작업 규칙
- 콘텐츠 추가는 `README.md`에 섹션 추가 (단일 페이지 흐름 유지)
- Mac/Windows(WSL) 양쪽 모두 단계별 비교 표로 병기
- 인용 박스로 "무엇을 한 것?" 메타 설명 곁들임 (스타일 일관성)
- 부트캠프 자체와 동기화 — 부트캠프 커리큘럼 바뀌면 여기도 업데이트

## 빠른 참조
- 로컬 미리보기: `bundle exec jekyll serve` (Ruby/bundler 설치 필요)
- 또는 그냥 GitHub Pages에 push해서 확인
- 빌드 산출물: `_site/` (gitignore)

## MCP 사용 메모
- 콘텐츠 작성 위주 — Serena/Context7 거의 불필요
- 외부 명령(brew, npm, gh 등) 인용은 본인 실제 환경 기준으로 검증한 것만
