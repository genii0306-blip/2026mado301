# 초등학교 학급회의 앱 — 프로젝트 안내 (CLAUDE.md)

> 이 파일은 Claude Code가 세션을 시작할 때 자동으로 읽는 프로젝트 컨텍스트입니다.
> 다른 컴퓨터/새 세션에서 작업을 이어갈 때 이 문서를 먼저 참고하세요.
> **작업 성격이 크게 바뀌거나 새 기능/버그를 다루면, 마지막 "변경 이력"과 "개선 로드맵"을 갱신해 주세요.**

## 한눈에 보기

- **무엇**: 초등학교 학급회의를 진행하고, 학생 참여를 게임처럼 동기부여하는 웹앱.
  회의 진행(발언·투표), 게시판, 실천 과제, 학급/개인 점수, 캐릭터 성장(레벨·직업·아이템 꾸미기), 숙제, 건의함을 한 화면에서 다룹니다.
- **대상 사용자**: 담임 교사 1명 + 학생들(태블릿/휴대폰). 실제 학급에서 운영 중이므로 **동작하는 기능을 깨뜨리지 않는 것**이 최우선입니다.
- **형태**: 빌드 도구 없는 **단일 HTML 파일**(`학급회의.html`). 열면 바로 실행됩니다.
- **데이터**: **Firebase Realtime Database**(리전 asia-southeast1)에 실시간 동기화. 로그인/서버 코드 없음. 클라이언트가 곧 전부입니다.
- **배포**: GitHub(`genii0306-blip/2026mado301`) → push하면 **Vercel이 자동 배포**.

## 실행 & 배포

로컬 확인은 파일을 브라우저로 열기만 하면 됩니다(별도 빌드/서버 불필요). 배포는:

```bash
cd "C:\Users\User\Claude\Projects\초등학교 학급회의 어플"
git add .
git commit -m "변경 내용 설명"
git push
```

→ Vercel이 새 버전을 자동 배포합니다.

**주의사항**
- 여러 컴퓨터에서 작업할 때는 시작 전 `git pull`, 끝나면 `git push`로 충돌을 예방하세요.
- `vercel.json`은 모든 경로를 정적 파일로 서빙하도록만 설정돼 있습니다(라우팅 규칙 최소).
- Firebase 설정 키(`firebaseConfig`)는 파일에 노출돼 있습니다. Realtime DB 웹앱 특성상 정상이지만, **보안 규칙은 Firebase 콘솔에서 관리**됩니다. 규칙을 넘어서는 접근 제어는 클라이언트에서 기대하지 마세요.

## 세계관: 마도 아카데미아 모험학교 (Mado Academia Adventure School)

이 앱의 RPG 세계관. 실제 마도초등학교 = 판타지 세계의 **마도 아카데미아 모험학교(1935년 설립)**.

### 공식 색상 팔레트
| 이름 | HEX | 용도 |
|------|-----|------|
| Academy Green | `#1E7D3B` | 학교 기본색, 배지 테두리 |
| Academy Gold | `#FFD23C` | 길드 배지, 금장 장식 |
| Leather Brown | `#8B5E34` | 벨트, 파우치, 신발 |
| Ivory | `#F5F0E8` | 연습복 기본색 |
| Silver Gray | `#B7BDC6` | 금속 버클, 장식 |

직업 선택(Lv.20) 이후에는 직업 고유색을 위 팔레트 위에 더합니다.

### 재질 스타일
가죽 · 린넨/천 · 황동 · 금장 · 수정 · 나무.  
광원 방향, 채도, 그림자, 모서리 라운딩 모두 일관되게 제작.

### 길드 (House) 5종
| 길드 | 상징 | 색 | 가치 |
|------|------|----|------|
| SAPIENTIA | 📖 | 보라 | 지혜 |
| FORTIS | ⚔️ | 빨강 | 용기 |
| NATURA | 🌱 | 초록 | 탐구 |
| CREARE | 🎨 | 하늘 | 창조 |
| LUMEN | 🤝 | 노랑 | 협력 |

### 장비 슬롯 구조 (11슬롯 + 자동배지)
```
head | hair | face | neck | chest | waist
back | cape | leftHand | rightHand | pet | foot
badge (자동)
```

### 장비 이미지 규격
- **형식**: 투명배경 PNG, 캔버스 크기(400×540) 맞춤
- **경로**: `equipment/{slot}/{item_id}.png`
- **렌더링**: 전체 캔버스에 drawImage → 위치가 이미지에 내장됨
- **이모지 폴백**: PNG 없으면 `CARD_ITEM_POS`의 앵커 위치에 이모지 표시

### 레벨 세트 구조
| 레벨 | 세트 | 주요 아이템 |
|------|------|-------------|
| Lv.1 | Starter | 길드 배지(chest) |
| Lv.5 | Apprentice | 허리벨트·파우치(waist), 모험수첩(leftHand), 견습망토(cape) |
| Lv.10 | Explorer | 탐험가가방(back), 장갑(rightHand), 나침반(neck), 물병(waist) |
| Lv.20 | Job Set | 직업별 무기·망토·정령(15종) |

---

## 파일 구조

- `학급회의.html` — **메인 앱**(약 3500줄). HTML + CSS + JS가 한 파일에 모두 들어 있습니다. 대부분의 작업은 여기서 이뤄집니다.
- `index.html` — 별도의 캐릭터/아이템 미리보기용 페이지(참고용, 메인 앱과 분리).
- `char_01.png ~ char_10.png` — 학생 기본 캐릭터 이미지(명단 순서와 1:1 매핑).
- `card/` — 캐릭터 카드 배경 이미지(`char_XX_bg_YY.png`, `char_XX_nobg.png`).
- `equipment/` — **장비 이미지 폴더**. 슬롯별 하위 폴더(`badge/`, `waist/`, `cape/` 등). PNG는 400×540 투명배경 풀캔버스 오버레이.
- `AvatarSample_Y.vrm` — VRM 캐릭터 파일(현재 미사용, 참조용으로 보관).

## 코드 구조 (`학급회의.html` 내부)

크게 세 부분입니다.

1. `<style>` — CSS 변수(`:root`) 기반 테마. 파랑(`--primary`) 톤, 카드형 UI, 라운드/그라디언트가 일관된 디자인 언어입니다. 새 UI는 기존 클래스(`.card`, `.btn`, `.btn-primary`, `.chip`, `.step-item` 등)를 재사용하세요.
2. `<body>` — 화면(탭) 마크업. `<main>` 안의 `.tab` 요소들이 탭별 화면이고, `.tab.active`만 보입니다.
3. `<script>` — Firebase 초기화, 상수(카탈로그/직업/아이템), 상태, 렌더 함수들.

### 탭 구성 (nav)
`go('탭키')`로 전환합니다.

| 탭키 | 이름 | 설명 |
|------|------|------|
| `flow` | 🏃 회의 진행 | 4단계 회의(안건→토의→발언·투표→정리), 발언 대기열, 타이머 |
| `board` | 💬 게시판 | 글 작성/좋아요/필터 |
| `action` | 🎯 실천 | 학급 실천 과제 체크 + 진행률 |
| `score` | 🏆 학급 점수 | 총점, 월별 꺾은선 그래프, (교사) 점수 사용 |
| `personal` | ⭐ 개인 점수 | 학생별 점수, (교사) 임의 점수 부여 |
| `history` | 📂 지난 회의 | 과거 회의 기록 |
| `mypage` | 🧑 마이페이지 | 로그인 학생 전용(기본 숨김) |
| `homework` | 📚 숙제 | (교사) 숙제 추가/현황, (학생) 내 숙제 |
| `suggest` | 📣 건의 | 건의사항 |
| `admin` | ⚙️ 관리자 | 교사 전용(기본 숨김): 학생 관리, 회장/부회장 권한, 직업 배정 |

### 상태 & 권한
- `me` — 현재 사용자 이름(문자열). 교사는 `'선생님'`.
- `classInfo.president` / `classInfo.vicePresident` — 회장/부회장.
- 헬퍼: `isPresident()`, `isGranter()`(회장·부회장·선생님), `openM(id)`/`closeM(id)`(모달), `todayStr()`, `nowYM()`, `genId()`.

### 데이터 모델 (Firebase refs, `학급회의.html` 상단에서 정의)
각 `db.ref(...)`가 하나의 데이터 묶음입니다. 주요 항목:

- `students` — 학생 명단. 기본값 `DEFAULT_STUDENTS`(10명).
- `classInfo` — 회장/부회장 등 학급 정보.
- `xpData` — 학생별 누적 XP(`{name:{total, ...}}`). 레벨의 근거.
- `jobData` — 학생별 직업(`{name:{jobId}}`).
- `customEquipData`(`customEquip`) — (구버전, 거의 미사용). 신규 장비는 `characterData`에 저장.
- `characterData` — 학생별 착용 아이템. 슬롯 이름이 키: `{hair, face, neck, chest, waist, back, cape, leftHand, rightHand, pet, foot, badge}`. `selectCCItem(slot, itemId)`으로 저장.
- `board`(게시판), `practiceBoard`(실천 후기), `proposals2`(안건/투표), `evaluation`(발언 평가),
  `meetingFlow`(회의 단계 상태), `classScore`(학급 점수), `stepActions`, `actionData`(실천),
  `speakRequests`/`speakCount`(발언 대기열·횟수), `homework`(숙제), `suggestions`(건의),
  `personalScore`/`personalScoreReq`(개인 점수·요청), `shopData`, `timerState`(타이머), `jobData`, `adminConfig`.

### 게임 시스템(핵심 규칙)
- **XP → 레벨**: `getLevel(xp)`가 임계값 배열로 Lv.1~50을 계산. 설계 기준은 "하루 3XP × 194일 = 581XP → 12월에 Lv.50".
  구간별 XP 증가: Lv.2~5 +4, Lv.6~10 +7, Lv.11~20 +10, Lv.21~30 +15, Lv.31~50 +14.
  등급명: 신입(Lv.1)/견습(5)/초급(10)/정식(20)/중급(30)/상급(50) 모험가.
- **직업**: `JOB_CLASSES`(15종, 지식·탐구·예술·체육·인성 5계열). Lv.3 달성 시 선택 모달, Lv.4부터 직업 전용 장비 해금.
- **아이템**: `ITEM_CATALOG`. 각 아이템은 `slot`(`hair`/`face`/`neck`/`chest`/`waist`/`back`/`cape`/`leftHand`/`rightHand`/`pet`/`foot`/`badge`)과 `unlock:{lv, job?}` 조건을 가집니다. `isItemUnlocked(itemId,name)`이 레벨+직업으로 해금 여부 판정.  `_auto:true`인 배지는 장비 UI에서 숨김. `getUnlockedItems(name)`은 `_auto` 아이템 제외 반환.
- **아이템 이미지 렌더링**: `equipment/{slot}/{id}.png` (400×540 투명 풀캔버스) 시도 → 없으면 이모지 폴백(`CARD_ITEM_POS` 앵커 사용). `SPRITE_MAPS`는 제거됨.
- **캔버스 레이어 순서**: BEHIND슬롯(`back`, `cape`) 먼저 → 캐릭터 nobg PNG → 나머지 아이템 위에 그림.
- **배지**: 레벨에 따라 자동 해금(bronze→silver→gold→platinum). `getBestBadge(name)`.

## 코딩 컨벤션 & 주의점

- **단일 파일 유지**: 이 프로젝트는 의도적으로 파일 하나로 운영됩니다. 별도 CSS/JS 분리는 하지 마세요(빌드 파이프라인 없음).
- **기존 스타일 재사용**: 새 버튼/카드는 기존 CSS 클래스를 쓰고, 색은 CSS 변수(`var(--primary)` 등)를 사용하세요.
- **간결한 코드 스타일**: 기존 코드가 한 줄에 압축된 형태(세미콜론 구분)를 많이 씁니다. 무리하게 리팩터링하지 말고 주변 스타일에 맞추세요.
- **실시간 앱**: 값 변경은 대개 `ref.set()/update()`로 저장하면 리스너가 다시 렌더합니다. 로컬 상태만 바꾸고 저장을 빠뜨리지 않도록 주의.
- **깨지면 안 됨**: 실제 수업에서 학생들이 사용합니다. 변경 후에는 최소한 **브라우저에서 열어 콘솔 에러가 없는지** 확인하고, 가능하면 관련 탭을 눌러 동작을 확인하세요.
- **검증 팁**: 큰 변경 전후로 `<script>` 구문을 Node로 확인할 수 있습니다(아래 "검증" 참고).

## 검증 방법

배포 전 간단 점검:

1. 브라우저에서 `학급회의.html`을 열고 개발자도구 콘솔에 에러가 없는지 확인.
2. 바꾼 기능과 관련된 탭을 눌러 실제로 동작하는지 확인.
3. (선택) 스크립트 구문만 빠르게 보고 싶으면, `<script>` 내용만 추출해 `node --check`로 검사.

## 알려진 이슈 / 개선 로드맵

작업을 이어갈 때 참고할 후보입니다. **하나 처리하면 이 목록을 갱신**하세요.

### Phase 1 — 아트 가이드 & 브랜드 (진행중)
- [x] 마도 아카데미아 IP 컨셉·색상·길드·슬롯 구조 확정
- [x] `ITEM_CATALOG` 재설계 (새 슬롯 체계, `equipment/` 폴더 방식)
- [x] VRM 3D 코드 제거, 풀캔버스 오버레이 방식으로 전환
- [ ] `equipment/` 폴더 하위 디렉터리 생성 (placeholder)
- [ ] 공식 학교 로고 디자인 (나침반+책+방패)

### Phase 2 — 기본 캐릭터 재작업
- [ ] `char_01~10.png` SD 3D 치비 스타일 리메이크 (머리:몸 = 2.5:1, 1024×1024, 투명배경)
- [ ] `card/char_XX_nobg.png` 동일 스타일로 업데이트

### Phase 3 — 장비 라이브러리 제작
- [ ] `guild_badge.png` (chest) — 최우선
- [ ] `belt_apprentice.png`, `pouch_small.png` (waist)
- [ ] `notebook_adventure.png` (leftHand)
- [ ] `cape_apprentice.png` (cape)
- [ ] `bag_explorer.png` (back)
- [ ] 나머지 Explorer·Job 세트 장비들

### Phase 4 — 학생 개별 생성
- [ ] 10명 학생 개별 SD 3D 캐릭터 카드 생성 (장비 오버레이 테스트 포함)

### 유지보수
- [ ] (검토) `flow` 회의 진행 4단계 UX 다듬기.
- [ ] (검토) 모바일(좁은 화면)에서 nav 가로 스크롤·그리드 밀림 여부 점검.
- [ ] (아이디어) 접근성: 버튼 `aria-label`, 색 대비, 포커스 스타일 보강.
- [ ] (아이디어) 오프라인/네트워크 끊김 시 사용자 안내.

## 변경 이력

- 2026-07-05 — 마도 아카데미아 IP 도입. VRM 3D 제거. `ITEM_CATALOG` 전면 재설계(새 슬롯 11종: hair/face/neck/chest/waist/back/cape/leftHand/rightHand/pet/foot). `SPRITE_MAPS` 제거, `equipment/{slot}/{id}.png` 풀캔버스 오버레이 방식 도입. `_drawItem()`, `CARD_ITEM_POS` 업데이트. CLAUDE.md 대폭 갱신(세계관·장비 규격·로드맵).
- 2026-07-04 — CLAUDE.md 신설(프로젝트 컨텍스트 문서화). `getLevel()` 이모지 오타 수정.
- (이전) 실천 후기 게시판 추가.
