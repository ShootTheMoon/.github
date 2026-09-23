# ShootTheMoon GitHub 공개 계획

## Context

`ShootTheMoon` 조직(free 플랜, 현재 리포 0개, `Seungpyo1007` 계정으로 `read:org`+`repo` 인증됨)에
OVERDARE 게임 프로젝트와 그 맵 파이프라인을 **공개 포트폴리오**로 올린다.

문제는 규모와 성격이다. 로컬 작업물은 약 50 GB이고(창덕궁 20 GB, 베를린 8.8 GB, 시부야 5.1 GB),
그중 대부분은 국가유산청 스캔 원본·PLATEAU 도시데이터·Sketchfab 모델 등 **3자 소스이거나 그 중간 산물**이다.
GitHub 리포 권장 상한은 5 GB이고 free 조직의 Git LFS 무료 한도는 **스토리지 1 GB / 대역폭 월 1 GB**이므로,
"폴더를 그대로 push"는 성립하지 않는다.

목표 결과물: 누가 봐도 무엇을 만들었는지 3분 안에 파악되는 공개 리포 세트 —
코드·파이프라인·문서·렌더는 일반 git, **게임레디 익스포트만 선별해서 LFS**, 대용량 팩은 Releases.
현재 어느 폴더에도 git이 없다(`overdare-mcp`만 개인 계정 리포로 존재).

## 결정된 리포 구성 (6개, 전부 public)

| 리포 | 내용 | git 크기 | LFS |
|---|---|---|---|
| `.github` | 조직 프로필 README = 랜딩 페이지. 맵 6종 표 + 스크린샷 + 각 리포 링크 | < 5 MB | 0 |
| `onlyoneshot` | OVERDARE 게임 본체: `Lua/*.lua`, `docs/`(README+마일스톤 4건), UI 정의, 스크린샷 | < 50 MB | 0 |
| `overdare-map-pipeline` | Blender→OVERDARE 변환 툴킷(MIT). 순수 코드 | < 10 MB | 0 |
| `heritage-maps` | 산곡·경복궁·창덕궁·근정전·소쇄원 (KOGL 계열) | < 50 MB | ~150 MB |
| `coldwar-berlin-assetpack` | 베를린 38 에셋 팩 (Sketchfab CC-BY + AI 생성) | < 60 MB | ~290 MB |
| `shibuya-assetpack` | 시부야 (PLATEAU CC-BY + OSM ODbL) | < 30 MB | ~61 MB |

LFS 합계 ≈ **500 MB / 1 GB** — 무료 한도 안에 들어간다. 이 여유를 지키려고 무기 11종(총 550 MB)과
FBX 세트는 LFS에 넣지 않는다.

라이선스가 리포 분리의 실제 기준이다. KOGL Type 1(국가유산청) / CC-BY(Sketchfab·PLATEAU) /
ODbL(OSM 도로) / 자작을 한 리포에 섞으면 재배포 조건을 문서로 못 지킨다. 그래서 소스 출처별로 갈랐다.

## 리포별 상세

### 1. `ShootTheMoon/.github`
`profile/README.md` 하나가 실질 콘텐츠. 조직 페이지 상단에 자동 노출된다.
- 맵 6종 표: 이름 / 상태(라이브·창고·소스only) / 소스 데이터 / 대표 이미지
- 상태는 조사에서 확인된 사실 그대로: 산곡=라이브(3,396 인스턴스), 화성·경복궁=ServerStorage 파킹,
  창덕궁=마커만, 베를린=제거됨(에셋팩만 생존), 시부야·크렘린=Blender 단계
- 이미지는 `blender/*/04_Previews`, `GY_Gyeongbokgung/GY_Phase4_*.png`, `onlyoneshot/Screenshots`에서
  8~12장만 골라 1600px 리사이즈 후 커밋

### 2. `ShootTheMoon/onlyoneshot`
포함: `Lua/*.lua`(약 25종 — HUD, CombatServer, LobbyUI, MobileControls, BoundaryServer 등),
`docs/README.md`, `docs/milestones/001~004`, `metadata.json`, `WorldSettings` 관련 텍스트, 큐레이션한 스크린샷.

제외(.gitignore):
- `*.ovdrjm` / `*.ovdrjm.bak` — 개당 29 MB, 백업본만 9개(≈270 MB). 저장할 때마다 전체가 바뀌는
  UTF-16 JSON이라 LFS에 넣으면 커밋 한 번에 29 MB씩 적립된다. 대신 스냅샷 1개를 Releases에 올린다.
- `*.uasset`, `*.umap`, `Baseplate.ovdrm`, `.overdare-backups/`, `UGCLocalAssetTable.json`(2.5 MB, 로컬 경로 포함)
- `_backup_*`, `_live_*.lua`, `Play.log`

대신 `docs/world-tree.md`를 생성해 커밋한다 — `.ovdrjm`에서 인스턴스 트리를 요약한 텍스트
(JSN 3,375 / HWS_COL 992 등). 바이너리 없이 월드 구성을 보여주는 용도.

### 3. `ShootTheMoon/overdare-map-pipeline`
포트폴리오에서 가장 재사용 가치가 높은 리포. 이미 존재하는 코드를 모은다:
- `MeshTest/overdare_convert.py` — Blender→OVERDARE FBX 변환기 (30k tri / 1 텍스처 규칙, 함정 6종)
- `blender/JSN_Sangok/_scripts/`, `blender/SSW_Soswaewon/_scripts/ssw_live.py`(77 KB — `_import_fbx_file`,
  OP 알파 배선, `height_at` 스캐터, 정점컬러 PBR. JSN이 이미 재사용한 실적 있음)
- `blender/CDG_Changdeokgung/_scripts/` — 머티리얼 분할 + 재질별 데시메이션 정책
- `JSN_Sangok/06_OVERDARE/IMPORT_GUIDE.md`, `IMPORT_ORDER.md`, `PLACEMENT_SPEC.md`
- `MeshTest/IMPORT_ORDER_V18.md`
- `onlyoneshot/.overdare/procedural/*` 중 배치·충돌 레시피 (hwaseong-v23-placement 등)

README에 축 변환 규칙(Blender 1 m = 100 cm, (x,y,z)→(X,Z,Y), +Y북 = −Z)과 검증된 함정 목록을 정리.
라이선스 **MIT**. 여기엔 3자 에셋이 한 톨도 안 들어가므로 제약이 없다.

### 4~6. 에셋 팩 3종
공통 구조:
```
README.md            에셋 표(이름/트라이앵글/출처/라이선스) + 프리뷰 그리드
ATTRIBUTION.md       3자 소스 전부 나열 (Kremlin/ATTRIBUTION.md 형식 재사용)
LICENSE / LICENSE-ASSETS
previews/            리사이즈 PNG (일반 git)
exports/*.glb        게임레디 GLB만 LFS
docs/                제작 기록
```
- `heritage-maps`: JSN 산곡(`06_OVERDARE/10_MASTERS` 중심), 경복궁 `FBX_Final`(1.3 MB — 통째로 가능),
  창덕궁·근정전·소쇄원은 **문서와 프리뷰만**(원본 20 GB·1.8 GB는 업로드 대상 아님). KOGL Type 1 출처표시.
- `coldwar-berlin-assetpack`: `02_Exports_GLB`(290 MB)를 LFS로. `01_Exports_FBX`(489 MB)는 Release zip.
  Sketchfab UID·CC-BY 저작자를 에셋별로 명시(베를린 장벽·Guard Tower·Road Gate Barrier·Emirates Stadium 등),
  Hunyuan3D 생성분은 AI 생성으로 별도 표기.
- `shibuya-assetpack`: GLB 61 MB LFS. PLATEAU(CC-BY 4.0)·OSM(**ODbL** — 도로 파생물엔 share-alike 조건이
  붙으므로 README에 명시) 출처와 EPSG:6677 / 앵커 좌표 레시피 문서화.

## 대용량은 Releases로

LFS와 달리 **Release 첨부파일은 LFS 쿼터를 쓰지 않고**(파일당 2 GB) 대역폭 과금도 없다. 따라서:
- `coldwar-berlin-assetpack v1.0` → `ColdWarBerlin_FBX.zip` (489 MB)
- `heritage-maps v1.0` → `JSN_Sangok_OVERDARE.zip` (189 MB)
- `onlyoneshot v1.0` → `onlyoneshot_world_snapshot.ovdrjm.zip` (29 MB)

## 실행 순서

1. **사전 정리(로컬)**: 각 리포용 스테이징 폴더를 `C:\tmp\stm\<repo>` 에 구성.
   원본 폴더는 건드리지 않고 복사만 한다.
2. `.gitignore` + `.gitattributes`(LFS 패턴: `*.glb`, `*.fbx` — 리포별로 실제 넣는 것만) 작성.
3. 커밋 전 점검: `git count-objects -vH`로 리포 크기, `git lfs ls-files --size` 합계 확인.
   비밀정보 스캔은 이미 1회 통과(토큰/키 패턴 0건)했으나, 스테이징 확정 후 한 번 더 돌린다.
   로컬 절대경로(`C:\Users\29\...`)가 문서에 남아 있으므로 상대경로로 치환한다.
4. `gh repo create ShootTheMoon/<name> --public` → 초기 커밋 → push.
5. Release 업로드(`gh release create`).
6. `.github/profile/README.md` 링크 연결.

우선순위: **`overdare-map-pipeline` → `onlyoneshot` → `.github` → 에셋 팩 3종**.
앞의 셋은 LFS가 필요 없어 바로 끝나고, 포트폴리오 임팩트도 가장 크다.

## 검증

- `gh repo list ShootTheMoon` 에 6개, 전부 public
- 각 리포 `git count-objects -vH` size-pack < 100 MB (에셋 팩 제외)
- `gh api /repos/ShootTheMoon/<r> --jq .size` 로 서버 측 크기 확인
- 빈 폴더에서 `git clone` → LFS 파일이 포인터가 아닌 실제 바이너리로 받아지는지 확인
- 조직 LFS 사용량이 1 GB 미만인지 확인 (초과 시 $5/월 데이터팩 필요 — 초과하면 해당 GLB를
  Release로 내리는 것이 기본 대응)
- 조직 페이지에 프로필 README가 렌더되는지, 이미지 깨짐 없는지 육안 확인
- 각 에셋 팩 README의 출처 표기가 실제 포함된 파일과 1:1로 맞는지 대조
