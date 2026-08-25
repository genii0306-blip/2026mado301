# 🎮 마도 아카데미아 앱 — 이어서 작업하기

> 이 파일을 새 세션 시작 시 Claude에게 보여주면 바로 이어서 작업 가능합니다.
> 더 자세한 내용은 **CLAUDE.md**를 참고하세요.

---

## 프로젝트 한눈에 보기

- **앱**: 초등학교 학급회의 + RPG 성장 시스템 웹앱
- **파일**: `학급회의.html` (단일 파일, ~3600줄, 빌드 도구 없음)
- **DB**: Firebase Realtime Database (asia-southeast1)
- **배포**: GitHub `genii0306-blip/2026mado301` → Vercel 자동 배포
- **세계관**: 마도 아카데미아 모험학교 (판타지 RPG, 6개 길드, 50레벨 성장)

---

## 지금까지 완성된 것

### ✅ 코드 (학급회의.html)
- `GUILDS` 상수 6종 (SAPIENTIA/FORTIS/NATURA/CREARE/LUMEN/REGIA)
- `ITEM_CATALOG` 전면 재설계 — `folder` 속성 추가, 아이템 추가
- `getCharTier(name)` 함수 — 플레이어 레벨 → 시각 티어(1~9) 변환
- `_drawItem()` — starter/apprentice 세트는 스킵, `equipment/{folder}/` 경로 사용
- `renderCardPreview()` — `characters/lv{tier}/character.png` 우선 사용

### ✅ 폴더 구조
```
equipment/
 ├── badge/       ← badge_lv1.png, brooch_sprout.png
 ├── shoes/       ← shoes_lv1.png
 ├── gloves/      ← gloves_lv3.png
 ├── boots/       ← boots_lv8.png
 ├── belt/        ← belt_apprentice.png
 ├── notebook/    ← notebook_adventure.png
 ├── pouch/       ← pouch_apprentice.png
 └── cape/        ← cape_apprentice.png

characters/
 ├── lv01/character.png   ← 연습복 + 길드배지
 ├── lv02/character.png   ← + 기본 신발
 ├── lv03/character.png   ← + 연습 장갑
 ├── lv04/character.png   ← + 새싹 브로치
 ├── lv05/character.png   ← + 견습 벨트
 ├── lv06/character.png   ← + 모험 수첩
 ├── lv07/character.png   ← + 모험 파우치
 ├── lv08/character.png   ← + 견습 장화(신발 업그레이드)
 └── lv09/character.png   ← + 견습 망토

assets/
 ├── icon/school_emblem.svg    ✅ 완성
 ├── icon/guild_badges.svg     ✅ 완성 (6종)
 └── ASSET_PACK_VOL1.html      ✅ SVG 스펙 시트 완성
```

### ✅ 문서
- `CLAUDE.md` — 세계관, Art Bible v1.0, 장비 시스템, 전체 로드맵 기록 완료

---

## 핵심 설계 결정 (변경 금지)

### 이미지 이중 구조
| 종류 | 경로 | 용도 | 규격 |
|------|------|------|------|
| 티어 캐릭터 | `characters/lv{01-09}/character.png` | 카드 메인 이미지 | 400×540 투명 PNG |
| 개별 아이템 | `equipment/{folder}/{id}.png` | UI 카드, 인벤토리 | 400×540 투명 PNG |

### 렌더링 규칙
- `characters/lv{tier}/character.png` = 해당 레벨까지 **모든 장비 누적 착용한 완성 이미지**
- starter/apprentice 세트 아이템은 `_drawItem()`에서 **오버레이 스킵** (티어 이미지에 이미 포함)
- explorer/job 세트 아이템만 티어 이미지 위에 오버레이

### 레벨 → 티어 매핑
```
게임 Lv.1 → lv01 | 게임 Lv.2 → lv02 | ... | 게임 Lv.9+ → lv09
```
(Lv.9 이후는 lv09 유지, Lv.20 직업 세트부터 오버레이 방식으로 추가)

---

## 지금 당장 해야 할 것 (Next Steps)

### STEP 1 — GPT로 이미지 생성
GPT(DALL-E)에서 아래 이미지를 생성해서 해당 폴더에 넣기:

**characters/ 티어 캐릭터 (최우선)**
- SD 3D 치비 스타일, 머리:몸 = 2.5:1, 400×540 투명배경 PNG
- lv01: 연습복(아이보리 #F6F3E6) + 학교 배지(초록 #1E7D3B + 금 #FFD23C)
- lv02: lv01 + 기본 신발(아이보리/베이지)
- lv03: lv02 + 연습 장갑(가죽 #8B5A2B)
- lv04: lv03 + 새싹 브로치(목에)
- lv05: lv04 + 견습 벨트(허리, 가죽+황동 버클)
- lv06: lv05 + 모험 수첩(왼손, 나침반 엠보싱)
- lv07: lv06 + 모험 파우치(허리 오른쪽)
- lv08: lv07 + 견습 장화(신발 업그레이드, 더 두꺼운 가죽)
- lv09: lv08 + 견습 망토(초록 #1E7D3B, 금 테두리)

**equipment/ 개별 아이템**
- 400×540 투명배경 PNG, 아이템이 캔버스 전체 영역에 배치
- `equipment/badge/badge_lv1.png` 등 (위 폴더 구조 참조)

### STEP 2 — 이미지 넣고 확인
1. 생성된 PNG를 해당 폴더에 저장
2. `학급회의.html` 브라우저로 열어서 마이페이지 → 캐릭터 카드 확인
3. 콘솔 에러 없는지 체크

### STEP 3 — Git Push
```powershell
cd "C:\Users\User\Claude\Projects\초등학교 학급회의 어플"
git add .
git commit -m "캐릭터 티어 이미지 추가"
git push
```

---

## 색상 팔레트 (Art Bible v1.0)
| 이름 | HEX | 용도 |
|------|-----|------|
| Academy Green | `#1E7D3B` | 학교 기본색, 망토, 배지 |
| Academy Gold | `#FFD23C` | 금장 장식, 테두리 |
| Ivory | `#F6F3E6` | 연습복 기본색 |
| Leather Brown | `#8B5A2B` | 벨트, 파우치, 신발 |
| Silver Gray | `#C7C7C7` | 금속 버클, 장식 |

---

## 주의사항

- **단일 파일 유지**: 별도 CSS/JS 분리 하지 말 것 (빌드 파이프라인 없음)
- **실제 수업 운영 중**: 기존 기능 깨뜨리지 않는 것이 최우선
- **Git 충돌 예방**: 작업 시작 전 반드시 `git pull`
- **Git lock 에러 시**: PowerShell에서 `del .git\index.lock` 후 재시도
