# codeigniter4-docs-ko

CodeIgniter 4 사용자 가이드 한국어 번역 저장소입니다.

이 저장소는 [CodeIgniter4](https://github.com/hoksi/CodeIgniter4) 저장소의 `user_guide_src/source/locale/ko` 경로에 서브모듈로 포함됩니다.

---

## 한국어 HTML 빌드 방법

### 사전 준비

- Python 3.10 이상
- Git (서브모듈 지원)

---

### 1. 저장소 클론 및 서브모듈 초기화

```bash
git clone https://github.com/hoksi/CodeIgniter4.git
cd CodeIgniter4

# ko 서브모듈 초기화 및 체크아웃
git submodule update --init user_guide_src/source/locale/ko
```

이미 클론된 상태라면 서브모듈만 최신으로 업데이트합니다.

```bash
cd CodeIgniter4
git submodule update --remote user_guide_src/source/locale/ko
```

이후 빌드 명령은 모두 `CodeIgniter4/user_guide_src` 폴더에서 실행합니다.

```bash
cd user_guide_src
```

---

### 2. Python 가상 환경 설정

`user_guide_src` 폴더에서 실행합니다.

```bash
# 가상 환경 생성
python3 -m venv .venv

# 가상 환경 활성화 (Linux / macOS)
source .venv/bin/activate

# 가상 환경 활성화 (Windows)
.venv\Scripts\activate

# 의존성 설치
pip install -r requirements.txt
```

---

### 3. 한국어 HTML 빌드

```bash
# user_guide_src 폴더에서 실행
.venv/bin/sphinx-build -b html -D language='ko' source build/html
```

빌드가 완료되면 `build/html/` 폴더에 HTML 파일이 생성됩니다.

빌드 성공 시 출력 예시:

```
build succeeded.
The HTML pages are in build/html.
```

---

### 4. 클린 빌드 (기존 빌드 결과물 제거 후 재빌드)

```bash
# 기존 빌드 결과물 삭제
rm -rf build/html

# 한국어 HTML 재빌드
.venv/bin/sphinx-build -b html -D language='ko' source build/html
```

---

### 5. 빌드 경고 확인

정상적인 번역 상태라면 경고(WARNING) 없이 빌드가 완료됩니다.
경고가 발생하면 해당 PO 파일의 RST 마크업을 확인하세요.
자세한 내용은 [TRANSLATION_GUIDE.md](TRANSLATION_GUIDE.md)를 참고하세요.

---

## 디렉터리 구조

```
user_guide_src/
├── source/
│   ├── locale/
│   │   └── ko/                  ← 이 저장소 (서브모듈)
│   │       └── LC_MESSAGES/
│   │           ├── database/
│   │           │   └── query_builder.po
│   │           ├── incoming/
│   │           │   └── routing.po
│   │           └── ...
│   └── ...
├── build/
│   └── html/                    ← 빌드 결과물 (한국어 HTML)
├── requirements.txt
└── .venv/                       ← Python 가상 환경
```

---

## PO 파일 업데이트 방법

원본 RST 문서가 변경되면 POT 파일을 재생성한 뒤 PO 파일에 새 항목을 반영해야 합니다.

### 1. sphinx-intl 설치

`sphinx-intl`은 기본 `requirements.txt`에 포함되어 있지 않으므로 별도로 설치합니다.

```bash
pip install sphinx-intl
```

---

### 2. POT 파일 생성

`user_guide_src` 폴더에서 gettext 빌더로 POT 파일을 생성합니다.

```bash
.venv/bin/sphinx-build -b gettext source build/gettext
```

실행 결과 `build/gettext/` 폴더에 `.pot` 파일이 생성됩니다.

---

### 3. PO 파일 업데이트

`sphinx-intl update` 명령으로 기존 PO 파일에 새 항목을 추가합니다.

```bash
.venv/bin/sphinx-intl update -p build/gettext -l ko -d source/locale
```

- 기존 번역 항목은 유지됩니다.
- 원본에서 추가된 항목은 `msgstr ""`로 추가됩니다.
- 원본에서 삭제된 항목은 `#~ msgid` 형식으로 주석 처리됩니다.

---

### 4. 신규/변경 항목 번역

업데이트된 PO 파일에서 `msgstr ""`인 항목을 찾아 번역합니다.

```bash
# 미번역 항목 확인 (msgstr이 비어 있는 항목)
grep -rn 'msgstr ""' source/locale/ko/LC_MESSAGES/ | grep -v "^Binary"
```

번역 완료 후 빌드로 검증합니다.

```bash
.venv/bin/sphinx-build -b html -D language='ko' source build/html
```

---

### 전체 워크플로우 요약

```
RST 문서 변경
    ↓
sphinx-build -b gettext   → build/gettext/*.pot 생성
    ↓
sphinx-intl update        → LC_MESSAGES/**/*.po 업데이트 (신규 항목 추가)
    ↓
PO 파일 편집              → 신규 msgstr 번역
    ↓
sphinx-build -b html      → build/html/ 에 한국어 HTML 생성
    ↓
git commit & push         → ko 서브모듈에 커밋
```

---

## 번역 파일 수정 후 빌드

PO 파일(`LC_MESSAGES/**/*.po`)을 수정한 뒤 빌드를 실행하면 변경 사항이 반영됩니다.

```bash
# PO 파일 수정 후 재빌드 (증분 빌드 - 변경된 파일만 처리)
.venv/bin/sphinx-build -b html -D language='ko' source build/html

# 전체 재빌드
rm -rf build/html && .venv/bin/sphinx-build -b html -D language='ko' source build/html
```

---

## 번역 가이드

번역 규칙 및 RST 마크업 처리 방법은 [TRANSLATION_GUIDE.md](TRANSLATION_GUIDE.md)를 참고하세요.
