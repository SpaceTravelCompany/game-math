# Game Math

게임 개발자를 위한 수학 레퍼런스 사이트. 빌더는 [`topic-pages`](https://github.com/SpaceTravelCompany/topic-pages) 패키지를 사용한다.

## 사용법

```bash
cd game-math
npm install         # topic-pages 의존성 설치
npm run build       # dist/ 정적 HTML 생성
```

`dist/index.html` + `dist/assets/` 번들로 동작하는 **단일 페이지 앱**이다.

`assets/`의 이미지·파비콘 등 추가 에셋도 빌드 시 자동으로 `dist/assets/`에 복사된다.

서빙은 사용자 환경의 도구로 (VS Code Live Server, `npx serve`, `python -m http.server` 등).
`content/*.md` 또는 `site.json` 수정 후 `npm run build` 다시 실행 → `dist/` 갱신.

## 구조

- `site.json` — 섹션·주제·참조 링크·테마·baseUrl 정의
- `content/` — 주제별 마크다운
- `assets/` — 참조 이미지·파비콘(content 마크다운에서 참조; 빌드 시 `dist/assets/`로 복사)
- `package.json` — `topic-pages` 의존성 + 빌드 스크립트
- `topic-pages` (의존성) — 빌더 + 라이브러리 + 에셋
- `.github/workflows/` — main 푸시 시 GitHub Pages 자동 배포(`site.json`의 baseUrl과 연동)
- `docs/` — AI 계획 문서

## 라이선스

- 문서/콘텐츠: [CC BY-NC-SA 4.0](LICENSE)
- 빌더(topic-pages): [MIT](https://github.com/SpaceTravelCompany/topic-pages/blob/main/LICENSE)
