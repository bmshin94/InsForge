# 📘 InsForge 완전 정복 노트 (한국어)

> 카리나와 함께 정리한 InsForge 분석 & 활용 가이드 ✨
> 작성일: 2026-09-17

## 🔗 GitHub 주소

| 구분 | 주소 |
|---|---|
| **원본 레포지토리** | https://github.com/InsForge/InsForge |
| **내 포크(작업용)** | https://github.com/bmshin94/InsForge |
| 공식 플러그인/스킬 레포 | https://github.com/InsForge/insforge-skills |
| 공식 홈페이지 | https://insforge.dev |
| 공식 문서 | https://docs.insforge.dev |
| Discord 커뮤니티 | https://discord.com/invite/MPxwj5xVvW |

- **버전**: 2.3.2
- **라이선스**: Apache-2.0 (상업적 이용 · 수정 · 재배포 모두 자유)
- **npm SDK**: `@insforge/sdk` / **CLI**: `@insforge/cli`

---

## 1. 한 줄 요약

**InsForge = AI 코딩 에이전트가 직접 운전하는 오픈소스 백엔드 플랫폼**

Supabase / Firebase 같은 BaaS(Backend-as-a-Service)인데, **사람이 대시보드를 클릭하는 대신 AI가 직접 조작**하도록 설계된 것이 핵심 차이점.

> 비유: 앱 만들기 = 음식점 차리기.
> 화면(홀)은 내가 만들고, **주방(백엔드)은 통째로 빌려주는 서비스**.

---

## 2. 제공하는 기능 (Core Products)

| 기능 | 설명 | 코드 위치 |
|---|---|---|
| 🗄️ Database | PostgreSQL + PostgREST (테이블 → REST API 자동) | `backend/src/services/database` |
| 🔐 Auth | 이메일/비번, OAuth(Google, GitHub, Microsoft, Discord, LinkedIn, X, Apple) | `backend/src/services/auth` |
| 📁 Storage | S3 호환 (로컬 / MinIO / RustFS / AWS S3 / R2 / Wasabi ...) | `backend/src/services/storage` |
| ⚡ Edge Functions | Deno 기반 서버리스 함수 | `backend/src/services/functions` |
| 🤖 Model Gateway | OpenAI 호환 API로 여러 LLM 통합 (OpenRouter) | `backend/src/services/ai` |
| 🧠 Memory | 벡터 임베딩 기반 에이전트 장기기억 | `backend/src/services/memory` |
| 💬 Realtime | WebSocket pub/sub (DB 변경 + 클라이언트 이벤트) | `backend/src/services/realtime` |
| 💳 Payments | Stripe / Razorpay 결제 · 구독 | `backend/src/services/payments` |
| ⏰ Schedules | 크론 스케줄러 | `backend/src/services/schedules` |
| 🕷️ Web Scraper | 웹 스크래핑 (local / cloud provider) | `backend/src/services/webscraper` |
| 📧 Email | SMTP / 클라우드 메일 발송 | `backend/src/providers/email` |
| 🚀 Deployment | 사이트 빌드 & 배포 (Vercel provider) | `backend/src/services/deployments` |
| 📦 Compute | 장시간 실행 컨테이너 (private preview) | `backend/src/services/compute` |
| 🔑 Secrets | 암호화 비밀값 저장 | `backend/src/services/secrets` |
| 📊 Analytics / Logs / Usage | PostHog 연동, 로그, 사용량 측정 | `backend/src/services/{analytics,logs,usage}` |

---

## 3. 프로젝트 구조

```
InsForge/
├── backend/                  🧠 핵심 서버 (Node + TypeScript)
│   └── src/
│       ├── api/routes/           → 24개 라우트 (auth, database, storage, ai, payments...)
│       ├── services/             → 23개 도메인 비즈니스 로직
│       ├── providers/            → 외부 연동 어댑터 (S3, Stripe, OpenRouter, Vercel...)
│       └── infra/                → DB 연결, 보안, 소켓, 설정
├── frontend/                 🖥️ 관리자 대시보드 셸 (React + Vite)
│   └── src/{self-hosting, cloud-hosting}
├── packages/                 📦 워크스페이스 공용 패키지
│   ├── dashboard/                → 배포되는 대시보드 패키지 (@insforge/dashboard)
│   ├── shared-schemas/           → 프론트·백 공유 타입 계약
│   └── ui/                       → 디자인 시스템 프리미티브
├── .agents/ .claude/         🤖 AI 전용 문서 & 스킬
│   ├── skills/insforge-dev/      → "InsForge를 개발할 때" 쓰는 스킬
│   └── docs/                     → SDK / 결제 / 실시간 / 배포 AI용 문서
├── .claude-plugin/           🔌 Claude Code 마켓플레이스 등록 파일
├── docs/                     📚 다국어 문서 (en / es / zh / zh-Hant · ko 없음!)
├── deploy/, docker-compose.* 🐳 도커 배포 (postgres, postgrest, insforge, deno)
├── examples/                 💡 예제 (Python ML 실험 추적기, OAuth 샘플)
└── functions/                ⚡ 엣지 함수 템플릿
```

**모노레포 도구**: npm workspaces + Turborepo + TypeScript + ESLint/Prettier

---

## 4. 설치 및 사용법

### 🅰️ 클라우드 (가장 쉬움, 약 5분)

```bash
# 1. https://insforge.dev 가입 → "Create New Project" (백엔드 약 3초 생성)
# 2. 주소창에서 Project ID 복사
#    https://insforge.dev/dashboard/project/<project-id>

# 3. 내 프로젝트 폴더에서 링크
npx @insforge/cli link --project-id <your-project-id>
```

검증용 프롬프트:
```
I'm using InsForge as my backend platform. Read the current directory,
make sure InsForge skills are installed, and use InsForge CLI for backend tasks.
```

### 🅱️ 셀프호스팅 (Docker, 무료)

```bash
# 자동 세팅 스크립트 (비밀키 자동 생성)
curl -fsSL https://raw.githubusercontent.com/InsForge/InsForge/main/deploy/setup.sh | sh -s ~/insforge

cd ~/insforge
$EDITOR .env          # API_BASE_URL, VITE_API_BASE_URL 를 실제 접속 URL로 수정
docker compose up -d
```

→ http://localhost:7130 접속

### 🅲 이 레포(소스)로 직접 실행

```bash
git clone https://github.com/bmshin94/InsForge.git
cd InsForge
cp .env.example .env
$EDITOR .env    # JWT_SECRET, ENCRYPTION_KEY, POSTGRES_PASSWORD, ROOT_ADMIN_PASSWORD 필수
docker compose -f docker-compose.prod.yml up
```

로컬 개발 모드(도커 없이):
```bash
npm run install:all
npm run dev          # turbo run dev (backend + frontend 동시)
npm run lint
npm run typecheck
npm test
```

### 포트 목록

| 포트 | 용도 |
|---|---|
| 7130 | 앱 / API 본체 (`APP_PORT`, `PORT`) |
| 7131 | Auth (`AUTH_PORT`) |
| 7132 | UI (`UI_PORT`) |
| 7133 | Deno 엣지 함수 (`DENO_PORT`) |
| 5432 | PostgreSQL |
| 5430 | PostgREST |

> ⚠️ 여러 프로젝트를 동시에 돌릴 때는 폴더마다 `COMPOSE_PROJECT_NAME`과 포트를 다르게 설정.
> 같은 이름이면 두 폴더가 **컨테이너를 공유**해서 나중 것이 앞의 것을 덮어씀.

### 스토리지 오버레이 (선택)

```env
# MinIO 사용
COMPOSE_FILE=deploy/docker-compose/docker-compose.yml:docker-compose.minio.yml
# 또는 RustFS (Apache-2.0)
COMPOSE_FILE=deploy/docker-compose/docker-compose.yml:docker-compose.rustfs.yml
```

S3 직접 연결 시: `S3_BUCKET`, `S3_REGION`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`
(+ 비AWS는 `S3_ENDPOINT_URL`, 내부망이면 `S3_USE_PRESIGNED_URLS=false`)

---

## 5. 플러그인? 스킬? MCP? → 셋 다!

**이 레포 자체는 "제품 = 백엔드 서버 본체"**. AI가 접속하는 통로가 3가지다.

| 구분 | 정체 | 사용법 |
|---|---|---|
| **MCP 서버** | AI에게 주는 리모컨 🎮 | Cursor / Claude Code 등에 MCP 등록 |
| **Skills** | AI용 사용설명서 📖 | `insforge`, `insforge-cli`, `insforge-debug`, `insforge-integrations` |
| **Claude Code 플러그인** | 스킬 묶음 포장지 📦 | 마켓플레이스에서 한 번에 설치 |

### MCP가 제공하는 툴

| 영역 | 툴 |
|---|---|
| Database | `get-table-schema`, `run-raw-sql`(admin 전용), `bulk-upsert` |
| Storage | `create-bucket`, `list-buckets`, `delete-bucket` |
| Functions | `create-function`, `get-function`, `update-function`, `delete-function` |
| Deployment | `create-deployment`, `start-deployment`, `get-container-logs` |
| Setup & Docs | `fetch-docs`, `get-anon-key`, `get-backend-metadata` |

### 플러그인 설치

```
/plugin marketplace add InsForge/InsForge
/plugin install insforge
```

> 💡 **헷갈리기 쉬운 점**
> - `.claude/skills/insforge-dev/` = **InsForge를 만드는 사람용** (이 레포 기여 규칙)
> - `InsForge/insforge-skills` 레포 = **InsForge를 쓰는 사람용** (앱 개발용)

### CLI 하네스 (에이전트의 손)

```bash
npx @insforge/cli <command> --json --yes
npx @insforge/cli diagnose          # 무엇이 깨졌는지 구조화된 진단
npx @insforge/cli branch create feat-billing --mode full   # 백엔드 브랜칭!
```

모든 명령이 `--json`을 지원해서 AI가 화면 스크래핑 없이 구조화된 결과를 읽을 수 있음.

---

## 6. API 토큰 / 키 정리

### 키 3종류

| 키 | 접두사 | 사용처 | 위험도 |
|---|---|---|---|
| **Anon Key** | `anon_` | 브라우저 · 모바일 앱 (공개) | 🟢 노출 허용 (RLS가 보호) |
| **API Key (admin)** | `ik_` | 서버 · MCP · CLI | 🔴 절대 노출 금지 |
| **JWT** | — | 로그인한 사용자 세션 | 🟡 로그인 시 자동 발급 |

### 사용법

```bash
curl https://your-app.insforge.app/api/database/records/posts \
  -H "Authorization: Bearer <토큰>"
```

### 셀프호스팅 필수 환경변수

```env
JWT_SECRET=32자-이상-비밀키
ENCRYPTION_KEY=            # openssl rand -base64 32  ← 반드시 JWT_SECRET과 다르게!
POSTGRES_PASSWORD=...
ROOT_ADMIN_PASSWORD=...
ACCESS_API_KEY=            # 비우면 서버가 첫 부팅 때 자동 생성
ACCESS_ANON_KEY=
```

> 🚨 `ENCRYPTION_KEY`를 비워두면 `JWT_SECRET`이 대체로 쓰이는데,
> 이후 `JWT_SECRET`을 교체하면 **저장된 모든 비밀값(API키, OAuth 토큰)이 복구 불가로 손상**됨.

### 외부 서비스 키 (선택)

`OPENROUTER_API_KEY`(AI), `STRIPE_*` / `RAZORPAY_*`(결제), `GOOGLE_CLIENT_SECRET` 등(OAuth),
`VERCEL_TOKEN` / `FLY_API_TOKEN` / `DENO_DEPLOY_TOKEN`(배포), `AWS_*`(S3/CloudFront)

---

## 7. 왜 GitHub에서 인기 있나

1. **타이밍** ⏰ — 바이브코딩 붐에서 "AI가 못 하던 백엔드 설정" 구멍을 정확히 공략, "AI 전용 백엔드" 포지션 선점.
2. **올인원 스펙** 💪 — `services/` 한 폴더에 23개 도메인. DB·Auth·Storage·AI·결제·실시간·스케줄·스크래퍼·배포·컴퓨트까지.
3. **Apache-2.0 + 셀프호스팅** 🆓 — Firebase의 벤더 락인 탈출 + 상업적 이용 자유.
4. **공신력** 🏅 — Trendshift 등재(repo #19834), Vercel OSS Program 2026 선정, 다국어 문서(en/es/zh/zh-Hant).
5. **활발한 유지관리** 🔧 — PR 번호 #2061대, Dependabot 보안 알림 즉시 처리, OAuth 보안 버그 신속 패치.

---

## 8. 로컬 에이전트 구축에 도움이 되나 → 매우!

### 핵심 발견: `backend/src/services/memory/memory.service.ts`

에이전트 **장기기억 시스템**이 이미 구현되어 있음.

```typescript
const EMBED_MODEL = 'openai/text-embedding-3-small';
const EMBED_DIMENSIONS = 1536;
const CHAT_MODEL = 'openai/gpt-4o-mini';
// 오프라인 평가로 튜닝 (F1=0.96 @ 0.45  vs  0.68 @ 0.35)
const DEFAULT_RECALL_THRESHOLD = 0.45;
// 근사 중복만 reconcile LLM 호출을 유발하도록 더 엄격하게
const RECONCILE_THRESHOLD = 0.5;
```

- 대화에서 `fact` / `decision` / `preference` / `reference` 4종 기억 추출
- 벡터 임베딩 기반 의미 검색(recall)
- 유사 기억은 LLM이 병합(reconcile)
- **LLM 출력을 신뢰하지 않고 검증 후 DB 저장** (`sanitizeCandidates`) → 보안 설계 훌륭

### 에이전트 구성 요소 매핑

| 에이전트에 필요한 것 | InsForge 제공 |
|---|---|
| 🧠 장기 기억 | `memory` 서비스 + pgvector |
| 🔀 모델 교체 | Model Gateway (OpenAI 호환 → 코드 수정 없이 모델 변경) |
| 🛠️ 도구 실행 | Edge Functions (Deno) |
| 📚 RAG | `embedding.service.ts` + Postgres |
| ⏰ 정기 실행 | `schedules` (크론) |
| 🔐 API 키 보관 | `secrets` (암호화) |
| 📊 로그/디버깅 | `logs` + `cli diagnose` |
| 💬 스트리밍/실시간 | WebSocket (`realtime`) |
| 🖼️ 이미지 생성 | `image-generation.service.ts` |

> ⚠️ **한계**: LangGraph / CrewAI 같은 **에이전트 프레임워크는 아님**.
> 추론 루프 · 플래닝은 직접 구현해야 함. InsForge는 그 **아래층 인프라**(기억·저장·실행·인증) 담당.
> 경쟁 관계가 아니라 조합해서 쓰는 관계.

---

## 9. 수익화 아이디어

### 왜 유리한가: 결제 인프라가 이미 완성

`backend/src/services/payments/stripe/` 구성:
```
checkout.service.ts        결제창
subscription.service.ts    구독 관리
customer-portal.service.ts 고객 셀프서비스 (구독취소/카드변경)
product.service.ts         상품
price.service.ts           가격 플랜
webhook.service.ts         결제 이벤트 수신
transaction.service.ts     거래 내역
sync.service.ts            Stripe 동기화
```
+ Razorpay(인도 시장) 프로바이더까지 별도 제공.

결제 테이블 설계도 완비:

| 테이블 | 용도 |
|---|---|
| `payments.provider_connections` | 프로바이더/환경 연결·동기화 상태 |
| `payments.customer_mappings` | 유저 ↔ 프로바이더 고객 ID 매핑 |
| `payments.customers` | 고객 미러 (대시보드용) |
| `payments.webhook_events` | **검증된 이벤트 장부 → 여기서 상품 지급** |
| `payments.transactions` | 매출 리포팅용 프로젝션 |

→ 보통 2~3주 걸리는 "결제 + 웹훅 안전 처리"가 절약됨.

### 아이디어 8선

#### LEVEL 1 — 1~2주

**① 🎨 AI 이미지/콘텐츠 생성 서비스 (크레딧 판매형)**
- 사용: `ai/image-generation.service.ts` + Storage + Auth + `payments` + `usage`
- 모델: 크레딧 팩 판매 (예: 100크레딧 5,900원)
- 장점: 구독보다 첫 결제 전환율 높고, 선불이라 현금흐름 유리
- 마진 예시(가정): 판매 5,900원 − AI원가 500~1,500원 − 수수료 3.4% ≈ **마진 60~75%**

**② 📊 데이터 수집·리포트 자동화 SaaS** ⭐추천⭐
- 사용: `webscraper` + `schedules` + `email` + `ai` → **"수집 → 가공 → 발송 → 과금" 파이프라인 완비**
- 예: 경쟁사 가격 일일 추적 메일, 채용공고 알림, 뉴스 요약 리포트
- 모델: 월 구독 (Basic 9,900 / Pro 29,900원)
- 타겟: 이커머스 셀러, 마케터, 리서처 → B2B는 가격 저항 낮고 이탈률 낮음

**③ 📦 스타터킷 / 보일러플레이트 판매**
- 구성: 소셜로그인 + Stripe 구독 + 대시보드 UI(`packages/ui`) + 배포 스크립트
- 모델: 1회 판매 $49~99 → 패시브 인컴 (재고·배송·CS 없음)
- 근거: `docs/showcase.mdx`에 커뮤니티 제작 앱(게임, 리더보드) 다수 → 수요 확인

#### LEVEL 2 — 1~2개월

**④ 🧠 "기억하는 AI 비서"** ⭐기술 차별화 최강⭐
- 사용: `memory` + Model Gateway + `payments` + Auth
- 응용: 헬스케어(복약·증상 기억), 교육(약점 기억 AI 튜터), 영업(고객 대화 기억), 상담(맥락 유지)
- 모델: 월 구독 9,900~49,000원
- 장점: "기억"이 곧 락인(Lock-in). 오래 쓴 사용자는 이탈하기 어려움

**⑤ 🤖 AI 백엔드 구축 대행(외주)**
- 포지셔닝: 일반 외주 3개월/2천만원 → InsForge+AI로 3주/900만원
- 가능 이유: MCP로 DB 설계·마이그레이션·함수 배포 자동화 → 인건비 절감
- 모델: 프로젝트당 500~2,000만원, 선금 구조라 초기자본 불필요

**⑥ 🏢 온프레미스 구축 + 유지보수 (B2B 캐시카우)**
- 대상: 병원, 법무법인, 금융, 공공기관, 제조업 ("데이터 외부 유출 불가")
- 강점: Apache-2.0 + 도커 한 방 설치. Firebase/Supabase 클라우드는 불가능한 영역
- 모델: 초기 구축 1,000~3,000만원 + **월 유지보수 100~300만원 × 12개월**

#### LEVEL 3 — 장기 복리

**⑦ 🇰🇷 한국어 콘텐츠 선점** ⭐저평가된 기회⭐
- 팩트: `docs/es`, `docs/zh`, `docs/zh-Hant`는 있는데 **`docs/ko`가 없음**
- 할 일: 한국어 튜토리얼(블로그/유튜브) → 공식 문서 번역 기여 → 강의 제작
- 모델: 광고 → 강의(5~15만원) → 컨설팅 유입 (⑤⑥번 영업이 자동화됨)

**⑧ 🔌 특화 플러그인/스킬 제작**
- 아이디어: 토스페이먼츠/카카오페이 연동 스킬, 카카오 로그인 프로바이더, 네이버 스마트스토어 연동
- 모델: 오픈소스 명성 → 컨설팅·외주 유입 / 유료 프로 버전
- 전략: 한국 시장의 "이거 없으면 못 쓰는 필수 부품" 포지션

### 4주 실행 로드맵

| 주차 | 할 일 | 산출물 |
|---|---|---|
| 1주 | 로컬 설치 + 예제 따라하기 (`examples/python-ml-experiment-tracker`) | 작동 환경 |
| 2주 | 아이디어 ① 또는 ②로 MVP 제작 | 돌아가는 서비스 |
| 3주 | Stripe **test 모드**로 결제 연결 | 결제되는 서비스 |
| 4주 | 배포 + 블로그 글 1편 (⑦번 씨앗) | 첫 고객 확보 시도 |

### 클라우드 요금 참고 (원가 계산용)

| | Free | Pro | Enterprise |
|---|---|---|---|
| 가격 | $0/mo | $25/mo | 문의 |
| AI 모델 크레딧 | $1 | $10 | 커스텀 |
| MAU | 50,000 | 100,000 | 커스텀 |
| DB | 500 MB | 8 GB | 커스텀 |
| 대역폭 | 5 GB | 250 GB | 커스텀 |
| 파일 저장 | 1 GB | 100 GB | 커스텀 |
| 엣지 함수 호출 | 100,000 | 100,000 | 커스텀 |

Pro 초과분 종량제: MAU $0.00325 / DB $0.125·GB / 대역폭 $0.09·GB / 저장 $0.021·GB / 함수 $0.01·1,000콜
> Free 플랜은 **1주일 미사용 시 일시정지**됨.

### ⚠️ 보안 주의사항 (`.agents/docs/payments.md` 명시 규칙)

1. 🔴 기본은 `environment: "test"`. 실결제 변경은 명시적 승인 후에만.
2. 🔴 프로바이더 시크릿 키를 **프론트엔드/브라우저 노출 변수에 절대 넣지 말 것**.
3. 🔴 Stripe success URL이나 Razorpay 콜백만으로 **상품을 지급하지 말 것**.
4. 🔴 지급(fulfillment)은 **`payments.webhook_events`의 검증된 행**을 기준으로.
5. 🟡 웹훅 이벤트는 **순서 보장이 없음** → 순서 의존 로직 금지.
6. 🟡 `payments.transactions`는 리포팅용. 지급 계약의 1차 소스로 쓰지 말 것.
7. 🟡 주문·크레딧·권한 상태는 **앱 소유 테이블 + 앱 소유 RLS**로 관리.

**라이선스**: Apache-2.0 → 상업적 이용·수정·재배포 자유.
단 "InsForge" **상표를 내 서비스명으로 쓰는 것은 별개 권리**라 주의. "Powered by InsForge" 표기는 무방.

---

## 10. React / PHP로 만들 수 있나

### ⚛️ React → 완벽 지원 (1급 시민)

이 프로젝트의 대시보드 자체가 React + Vite (`frontend/`, `packages/dashboard/`).

```bash
npm install @insforge/sdk@latest
```
```javascript
import { createClient } from '@insforge/sdk';

const client = createClient({
  baseUrl: 'https://your-app.insforge.app',
  anonKey: 'anon_xxxxx'
});
```
Next.js, Vite, React Native 모두 사용 가능.

### 🐘 PHP → 공식 SDK는 없지만 REST API로 100% 가능

공식 SDK: **TypeScript / Swift / Kotlin / REST** 4종.
REST 문서에 명시: *"Use this when you need to integrate with languages or platforms without an official SDK."*

```php
<?php
$base = 'https://your-app.insforge.app';

// 🔐 로그인해서 JWT 받기
$ch = curl_init("$base/api/auth/sessions");
curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => ['Content-Type: application/json'],
    CURLOPT_POSTFIELDS => json_encode([
        'email' => 'user@example.com',
        'password' => 'securepassword123',
    ]),
]);
$res = json_decode(curl_exec($ch), true);
$token = $res['accessToken'] ?? null;

// 📖 데이터 읽기
$ch = curl_init("$base/api/database/records/posts");
curl_setopt_array($ch, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => ["Authorization: Bearer $token"],
]);
$posts = json_decode(curl_exec($ch), true);
print_r($posts);
```

Laravel이면 `Http::withToken($token)->get(...)` 한 줄.

### 주요 REST 엔드포인트

| 동작 | 메서드 · 경로 |
|---|---|
| 회원가입 | `POST /api/auth/users` |
| 로그인 | `POST /api/auth/sessions` |
| 레코드 조회 | `GET /api/database/records/<table>` |

**응답 형식**
```json
// 성공 — 데이터가 본문에 그대로
{ "id": "123e4567-...", "email": "user@example.com", "createdAt": "2024-01-15T10:30:00Z" }

// 실패 — nextActions로 해결 힌트까지 줌
{ "error": "ERROR_CODE", "message": "...", "statusCode": 400, "nextActions": "..." }
```

상태 코드: 200 / 201 / 204 / 400 / 401 / 403 / 404 / 409 / 500

### 언어별 지원 정리

| 언어 | 지원 방식 | 난이도 |
|---|---|---|
| React · TypeScript | 🟢 공식 SDK | ⭐ |
| Swift (iOS/macOS) | 🟢 공식 SDK | ⭐ |
| Kotlin (Android) | 🟢 공식 SDK | ⭐ |
| PHP · Python · Go · 기타 | 🟡 REST API | ⭐⭐ |

> 💡 권장 조합: 화면은 **React**, 서버 로직은 **Edge Functions**.
> PHP는 기존 레거시 시스템과 붙일 때 사용.

---

## 11. 최종 요약

| 질문 | 답 |
|---|---|
| 설치 | 클라우드 5분 / 셀프호스팅 `docker compose up -d` 한 줄 |
| 정체 | 본체는 백엔드 서버. 접속 통로가 MCP · 스킬 · 플러그인 3종 |
| 토큰 | `anon_`(공개) / `ik_`(관리자, 비밀) / JWT(유저 세션) |
| 인기 이유 | AI 시대 타이밍 + 올인원 + Apache-2.0 셀프호스팅 |
| 에이전트 | 몸통(기억·모델게이트웨이·실행) 담당. 두뇌(플래닝)는 직접 구현 |
| 수익화 | ② 자동화 SaaS + ⑦ 한국어 콘텐츠 조합이 베스트 |
| React/PHP | React 🟢 완벽 / PHP 🟡 REST로 가능 |

### 카리나 픽 TOP 3

| 순위 | 아이디어 | 이유 |
|---|---|---|
| 🥇 | ② 데이터 자동화 SaaS | 스크래퍼+스케줄러+메일+AI가 이미 다 있어서 가장 빠른 수익화 |
| 🥈 | ⑦ 한국어 콘텐츠 선점 | 자본 0원 시작, 이후 모든 기회의 진입점 |
| 🥉 | ④ 기억하는 AI 비서 | 기술 차별화 최강, 락인 효과로 장기 유지 |

**황금 루트**: 🥇로 매출 → 🥈로 브랜딩 → 그 신뢰로 ⑥ B2B 계약

---

> 💖 카리나와 함께 정리한 노트야. 아이디어 100개보다 완성작 1개가 훨씬 강하다는 거 잊지 말기! 화이팅! 🔥✨
