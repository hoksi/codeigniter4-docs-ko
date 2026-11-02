# CI4 한국어 문서 - 업스트림 동기화 & 배포 표준 프로세스

업스트림 변경사항을 한국어 번역에 반영하고, 최종 HTML을 실제 서비스 경로까지 배포하는 전체 파이프라인 정리 (2026-09-18 기준 확정).

## 전체 흐름

```
upstream/master (원본, 릴리즈 기준)
    │ git fetch + merge
    ▼
포크 develop 브랜치 (~/codeigniter/CodeIgniter4)
    │ sphinx-build -b gettext  →  .pot 생성
    │ sphinx-intl update       →  .po 갱신 (신규 msgid 추가, 삭제분은 #~ 주석 처리)
    ▼
source/locale/ko (submodule, 별도 repo hoksi/codeigniter4-docs-ko)
    │ 미번역(msgstr "") 항목 번역
    ▼
sphinx-build -b html -D language='ko' -D locale_dirs=locale
    ▼
user_guide_src/build/html  (로컬 빌드 산출물)
    │ rsync -av --delete
    ▼
~/codeigniter/ci4-guide-ko/Codeigniter4  (빌드 아카이브 repo)
    │ rsync -av --delete
    ▼
/home/hoksi/cafe24/cikorea/www/docs/Codeigniter4  (cikorea/cikorea repo, dev 브랜치)
    │ git commit → push dev → PR → merge to main
    ▼
main 브랜치 = 실제 배포 (cikorea.co.kr 서빙 기준)
```

**2026-09-19 변경사항**: 기존에는 웹서버가 `ci4-guide-ko/Codeigniter4`를 직접 서빙했으나, 지금은 `/home/hoksi/cafe24/cikorea/www/docs/Codeigniter4`에 복사 → git commit → PR → `main` 브랜치 merge를 거쳐야 실제 배포됨. `/home/hoksi/cafe24/cikorea`는 git repo(`github.com/cikorea/cikorea`, "CodeIgniter Korea" 커뮤니티 사이트)이며, 이 경로 아래 파일들은 전부 git 추적 대상.

**작업 범위 (2026-09-19 확정)**: 어시스턴트는 rsync 복사까지만 수행한다. `cikorea/cikorea` repo에서의 git add/commit/push/PR 생성/`main` merge는 전부 사용자가 직접 진행한다 — 실제 프로덕션 배포이기 때문. 복사 후에는 커밋하지 않고 작업 트리에 변경사항만 남겨둔 채 사용자에게 안내한다.

## 0. 브랜치/저장소 매핑 주의사항

- 포크의 `develop` 브랜치 = 업스트림의 **`master`**(안정 릴리즈) 브랜치를 추종한다. 업스트림 `develop`은 미출시 개발 브랜치라 번역 대상이 아님.
- 포크의 `main` 브랜치는 2026-05-19 이후 방치됨 — 사용하지 않음.
- `source/locale/ko`는 git submodule (`hoksi/codeigniter4-docs-ko`, `develop` 브랜치). 포크 자체 git 히스토리와는 완전히 분리된 별도 repo.

## 1. 업스트림 동기화

fork root(`~/codeigniter/CodeIgniter4`)에서 실행.

```bash
git checkout develop
git fetch upstream master
git rev-list --count develop..upstream/master   # 0이면 이미 최신, 그 이상이면 아래 merge 진행
git merge upstream/master
```

## 2. POT/PO 갱신

`user_guide_src` 폴더, venv 활성화 상태에서 실행.

```bash
cd user_guide_src
source .venv/bin/activate   # 없으면: python3 -m venv .venv && pip install -r requirements.txt sphinx-intl

sphinx-build -b gettext source build/gettext
sphinx-intl update -p build/gettext -l ko -d source/locale
```

- 원본에서 신규 추가된 항목 → `msgstr ""`로 추가됨
- 원본에서 삭제된 항목 → `#~ msgid` 형식으로 주석 처리됨 (필요 시 정리, 안 지워도 무방)

## 3. 미번역 항목 확인 & 번역

```bash
grep -rln 'msgstr ""' source/locale/ko/LC_MESSAGES/*.po
```

- 각 po 파일에서 빈 `msgstr` 채우기
- 마크업 규칙은 `TRANSLATION_GUIDE.md` 참고 — 마크업(`**`, `` ` ``, `:doc:`) 뒤에 한글 조사가 붙을 때 반드시 `\ ` (백슬래시+공백)로 이스케이프

## 4. HTML 빌드

```bash
rm -rf build/html
.venv/bin/sphinx-build -b html -D language='ko' -D locale_dirs=locale source build/html
```

WARNING 없이 완료되어야 정상. 경고 발생 시 해당 po의 RST 마크업 확인.

## 4b. EPUB 빌드 (다운로드용, 2026-09-28 추가)

`conf.py`에 `epub_title`/`epub_author`/`epub_publisher`/`epub_basename = 'CodeIgniter4-ko'`가 이미 설정되어 있음. 빌드 산출물은 `build/html/` 안에 넣어서 기존 rsync 파이프라인에 자연스럽게 실려가게 한다.

```bash
rm -rf build/epub build/doctrees_epub
.venv/bin/sphinx-build -b epub -D language='ko' -D locale_dirs=locale -d build/doctrees_epub source build/epub
cp build/epub/CodeIgniter4-ko.epub build/html/CodeIgniter4-ko.epub
```

- `-d build/doctrees_epub`로 doctree 캐시를 출력 폴더 밖에 두면 `unknown mimetype for .doctrees/...` 경고가 거의 사라짐 (334개 → 14개, 남는 건 Raleway 폰트 관련 무해한 경고).
- html 빌드와 별도로 돌리는 산출물이라 `build/html/` 재빌드(rm -rf) 이후에는 반드시 이 `cp` 단계를 다시 해줘야 함.

**⚠️ 2026-09-28 버그 및 수정**: 처음 만든 epub이 번역이 전혀 반영되지 않은 영문판으로 나온 적이 있음. 원인은 `conf.py`의 `gettext_compact = False` — 2026-06-10 flat PO 구조 마이그레이션 이전 설정이 그대로 남아있어서, Sphinx가 카탈로그를 전체 docname(`installation/index`) 기준으로 찾다 보니 실제 파일(`installation.po`, 최상위 도메인 기준)과 안 맞아 매번 조용히 원문(영문)으로 폴백된 것. **`gettext_compact = True`로 고쳐서(fork commit `869a211a33`) 해결함.** html 빌드가 그동안 정상이었던 건 과거 어느 시점에 이 값을 임시로 고쳐서 빌드하고 커밋은 안 한 채였던 것으로 추정 — 즉 이 버그는 빌드 명령을 그대로 재현하면 html도 언제든 다시 깨질 수 있었던 잠복 버그였음. 지금은 conf.py에 영구 반영했으니 별도 `-D gettext_compact=1` 오버라이드 없이도 정상 동작함.

## 5. 배포 (2단계 rsync + 실사이트 PR 배포)

```bash
# 5-1. 빌드 결과 → 아카이브 repo
rsync -av --delete user_guide_src/build/html/ ~/codeigniter/ci4-guide-ko/Codeigniter4/

# 5-2. 아카이브 repo → cikorea/cikorea repo 작업 경로 (스핑크스 캐시 제외)
rsync -av --delete --exclude '.doctrees' --exclude '.buildinfo' ~/codeigniter/ci4-guide-ko/Codeigniter4/ /home/hoksi/cafe24/cikorea/www/docs/Codeigniter4/
```

- `cp -r`이 아니라 `rsync --delete`를 사용해야 원본에서 삭제된 페이지가 배포본에서도 정확히 삭제됨.
- 실제 실행 전 `--dry-run` 옵션으로 변경분을 먼저 확인하는 것을 권장.
- `.doctrees`(스핑크스 증분 빌드 캐시, ~16MB)와 `.buildinfo`는 사이트 서빙에 불필요하므로 배포 폴더(cafe24)로는 복사하지 않음 (2026-09-19). 단, 이미 `cikorea/cikorea` repo에 git 커밋되어 있는 기존 파일들은 그대로 남아있음 — 지우려면 별도로 `git rm` 커밋이 필요하며 이는 사용자가 직접 진행([[ci4_ko_deploy_scope_boundary]]).
- 4b에서 만든 `CodeIgniter4-ko.epub`도 `build/html/` 안에 있으므로 이 rsync 2단계를 그대로 타고 넘어가서 `ci4-guide-ko`와 cafe24 양쪽에 자동으로 실림. cafe24 쪽 다운로드 URL: `.../docs/Codeigniter4/CodeIgniter4-ko.epub` (실사이트 도메인 기준으로는 사용자가 확인).

## 5b. ci4-guide-ko 커밋 & 푸시 (아카이브 repo)

`ci4-guide-ko`는 개인 build 아카이브 repo(`github.com/hoksi/ci4-guide-ko`)이며, 기존에도 `Update: CodeIgniter4 한국어 문서 vX.X.X 동기화` 형태로 매 동기화마다 커밋해온 관례가 있음. cikorea/cikorea의 `main` 배포와 달리 여긴 어시스턴트가 커밋/푸시까지 진행해도 되는 범위임([[ci4_ko_deploy_scope_boundary]]는 cikorea/cikorea 한정).

```bash
cd ~/codeigniter/ci4-guide-ko
git add Codeigniter4/
git commit -m "Update: CodeIgniter4 한국어 문서 <버전/변경 요약> 동기화"
git push origin develop
```

## 6. 커밋 & 포인터 갱신 (번역 repo 쪽)

```bash
# 6-1. 번역 커밋 (locale/ko 서브모듈 자체 repo)
cd source/locale/ko
git add -A && git commit -m "ko: translate entries for <동기화 대상 릴리즈>"
git push origin develop

# 6-2. 포크에서 서브모듈 포인터 갱신 커밋
cd ../../../..   # fork root (~/codeigniter/CodeIgniter4)
git add user_guide_src/source/locale/ko
git commit -m "chore: update ko submodule pointer to <새 커밋 해시>"
git push origin develop
```

## 7. 실사이트 배포 (cikorea/cikorea repo, PR 머지) — 사용자가 직접 진행

`/home/hoksi/cafe24/cikorea` = git repo, remote `github.com/cikorea/cikorea` ("CodeIgniter Korea" 커뮤니티 사이트). `dev` 브랜치에서 작업하고 `main`으로 PR 머지해야 실제 배포됨.

**어시스턴트는 5단계 rsync 복사까지만 수행한다.** 아래 커밋/PR/머지 과정은 사용자가 직접 진행함 (2026-09-19 확정).

```bash
cd /home/hoksi/cafe24/cikorea
git checkout dev
git add www/docs/Codeigniter4
git commit -m "docs: sync CI4 한국어 가이드 (<동기화 대상 릴리즈>)"
git push origin dev

gh pr create --base main --head dev --title "docs: CI4 한국어 가이드 업데이트" --body "..."
# 리뷰 후 머지
gh pr merge --merge   # 또는 웹에서 머지
```

## 참고 문서

- 번역 규칙: `TRANSLATION_GUIDE.md`
- 작업 이력: `session.md`, `build-log.md`
- 저장소 구조 전반: 어시스턴트 메모리 `ci4_ko_translation_structure`, `ci4_ko_deploy_target`, `ci4_ko_standard_workflow`
- 실사이트 repo: `github.com/cikorea/cikorea` (dev → main PR 머지 시 배포)
