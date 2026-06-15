# 🎴 바카라 멀티 (Multiplayer Baccarat)

Supabase(Postgres + Realtime) 백엔드를 쓰는 **멀티플레이 바카라** 웹앱입니다.
이름·생년월일로 입장하고, 솔로 플레이 / 실시간 공유 테이블 / 관전 / 랭킹을 한 화면에서 즐깁니다.

> 단일 `index.html` (의존성은 CDN의 supabase-js 하나). GitHub Pages 등 정적 호스팅으로 바로 배포됩니다.

## 기능

| 기능 | 설명 |
|------|------|
| **로그인** | 이름 + 생년월일로 식별/생성 (가벼운 식별, 보안 인증 아님) |
| **솔로** | 혼자 베팅·딜·단계별 카드 공개. 잔액은 DB에 저장되고 랭킹에 반영 |
| **공유 탭** | 전원이 같은 핸드에 베팅. **40초 주기(베팅 30초 + 공개 10초)**, 칩 스테이징 → 배팅 확정 → 딜 공개, 남은 시간 카운트다운. 좌측 하단 실시간 베팅 현황(누가·어디·얼마) |
| **관전 탭** | 접속자 목록 → 선택하면 그 사람의 실시간 플레이/잔액 관전 |
| **접속자 드로어** | 우측 반투명 `›` 핸들 → 슬라이드로 현재 접속 중인 이름 표시 |
| **랭킹** | 잔액 기준 실시간 리더보드 (순손익·판수 표시) |

공유 라운드의 카드는 라운드별 시드로 **결정론적 생성**되어 모든 클라이언트가 동일한 결과를 봅니다.
서버 시각(`server_now`)에 맞춰 라운드/타이머를 동기화합니다.

## 게임 규칙

표준 바카라(Punto Banco): 카드값 A=1, 2~9 그대로, 10·J·Q·K=0, 합계 mod 10.
내추럴 8·9, 정식 3번째 카드 룰. 배당 — Player 1:1, Banker 0.95:1(5% 커미션), Tie 8:1. 타이 시 Player/Banker 반환.

## 백엔드 (Supabase)

- 테이블: `players`, `shared_rounds`, `shared_bets`, `shared_settlements`
- RPC: `login_player`, `server_now`, `get_round_seed`, `place_shared_bet`(베팅창·잔액 검증), `settle_shared`(라운드당 멱등 정산)
- Realtime: `players`, `shared_bets` 구독으로 랭킹/관전/공유 베팅 실시간 갱신
- RLS 활성 (캐주얼: 공개 읽기 + anon 접근). 공유 잔액 변경은 SECURITY DEFINER RPC로 처리

### 설정

`index.html` 상단의 `SUPABASE_URL` / `SUPABASE_KEY`(publishable key)를 본인 프로젝트 값으로 바꾸면 다른 백엔드에 연결됩니다.

## 신뢰 모델 / 제한

- **캐주얼(친구용)**: 솔로 정산은 클라이언트 신뢰 기반이라 마음먹으면 조작 가능 — 친구끼리 즐기는 용도.
- 공유 라운드 정산은 라운드당 1회 멱등 처리되며, 누락 시 다음 로그인에서 자동 보정됩니다.
- 공유 라운드 시드는 라운드 시작 시 생성·저장되므로, 작정하고 DB를 들여다보면 결과를 미리 알 수도 있습니다(친구용 한도 내 허용).

## 기술

- 프런트: 순수 HTML/CSS/JS 단일 파일 + `@supabase/supabase-js@2`(CDN)
- 백엔드: Supabase (PostgreSQL 17, Realtime)
