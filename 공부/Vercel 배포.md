---
type: study
area: 도구·인프라
status: active
created: 2026-08-19
updated: 2026-08-19
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# Vercel 배포

Git 연동 배포는 **레포 루트를 프로젝트 루트로 가정**한다 — 사이트가 서브폴더에 있으면 Root Directory를 반드시 지정해야 하고, 빠뜨리면 사이트 전체가 404가 된다.

## 핵심 정리

- **Root Directory** (Project Settings → General): Git 연동으로 만들어지는 배포는 이 값 기준으로 빌드한다. 비어 있으면 레포 루트. 서브폴더 사이트(`landing/`)면 `landing`으로 설정. CLI(`vercel`)로 서브폴더 안에서 배포하던 프로젝트를 나중에 Git에 연결하면 **CLI 때는 잘 되던 것이 다음 push부터 통째로 404**가 된다 — 사람이 어느 폴더에서 배포하느냐가 아니라 프로젝트 설정이 기준이 되기 때문.
- 404인데 배포 상태는 "Ready"인 경우: 빌드 로그에 `Build Completed in 186ms` `Skipping cache upload because no files were prepared` 처럼 **거의 아무것도 안 만든 흔적**이 있으면 루트/출력 디렉터리 문제를 먼저 의심.
- **배포 URL(`*-<team>.vercel.app`)은 Deployment Protection 때문에 브라우저 외 요청에 302/401**을 돌려준다 — 함수가 죽었는지 확인할 땐 프로덕션 alias로 쳐야 한다. alias는 `vercel alias ls`, 어느 배포가 걸렸는지 `vercel ls`, 빌드 로그는 `vercel inspect <url> --logs`.
- 런타임 로그(`vercel logs <alias> --since 5d --json`)에는 **엣지에서 끝난 404(NOT_FOUND)는 남지 않는다** — 함수까지 도달한 요청만 보인다. "요청이 왔는지" 확인은 상대(여기선 Apple) 쪽 전송 이력이 더 믿을 만하다.
- 프로젝트 설정 변경은 대시보드 없이도 REST로 가능: `PATCH https://api.vercel.com/v9/projects/<id>?teamId=<org>` body `{"rootDirectory":"landing"}` (토큰은 `(로컬 경로)`). 이후 `vercel redeploy <deployment-url>`로 같은 커밋을 새 설정으로 재빌드.

## 기록

### 2026-08-19 — Git 연동 후 Root Directory 미설정으로 랜딩·웹훅 4일간 404
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] — "구매 슬랙 알림이 안 온다"를 추적하다가 `zappy-landing.vercel.app`이 `/`까지 404인 것을 발견
- 배운 것:
  - 08-14 프로젝트를 GitHub `dbsghdz1/Zappy`에 연결했는데 Root Directory가 비어 있었고, 08-15 push 배포가 레포 루트(Swift 프로젝트)를 빌드해 index.html도 `api/`도 없는 배포가 프로덕션 alias를 차지했다. 그 전까지는 `landing/` 폴더에서 CLI로 배포해 왔기 때문에 문제가 안 보였다.
  - 배포 URL로 웹훅을 찔렀을 땐 401이 나와 "살아 있다"고 착각할 뻔했다 — Deployment Protection의 401이었다. alias로 쳐서 404를 확인한 것이 결정적.
  - `rootDirectory=landing` PATCH + `vercel redeploy` 14초 만에 200/405 복구.
- 근거: 빌드 로그 `Cloning github.com/dbsghdz1/Zappy (Branch: main, Commit: 047ef1d)` … `Build Completed in 186ms`; 프로젝트 API 응답 `rootDirectory: None → 'landing'`; [[프로젝트/개인/Zappy/Zappy 마케팅 플랜]] 랜딩 절 장애 기록. 관련: [[공부/App Store Server Notifications|App Store Server Notifications]]

## 참고 자료
- [Vercel — Configuring a Build: Root Directory](https://vercel.com/docs/deployments/configure-a-build#root-directory) — 서브폴더 프로젝트 설정 위치 (2026-08-19 확인)
- [Vercel REST API — Update an existing project](https://vercel.com/docs/rest-api/reference/endpoints/projects/update-an-existing-project) — `rootDirectory` 필드 (2026-08-19 확인)
