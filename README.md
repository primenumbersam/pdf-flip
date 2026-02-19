<aside>
📌

PDF 파일을 이미지 시퀀스로 변환하여 웹상에서 플립북 효과를 구현하고, 정적 사이트 **3D Virtual Bookshelf**를 메인 인터페이스로 하는 Zero-Cost 자동화 프로젝트.

</aside>

## 🎯 개요

PDF 파일을 이미지 시퀀스로 변환하여 웹상에서 플립북 효과를 구현하고, 정적 사이트 **3D Virtual Bookshelf**를 main UI로 하는 프로젝트입니다.

학생들은 **"파일 업로드 = 서비스 자동 업데이트"** 라는 자동화 경험을 단 **0원의 비용**으로 학습합니다.

```
[PDF 업로드] → [GitHub Actions 자동 실행] → [이미지 변환 & books.json 갱신] → [GitHub Pages 즉시 배포]
```

---

## 🧠 학습 목표

이 프로젝트를 완료하면 다음 개념을 실전으로 이해합니다.

| **개념** | **설명** |
| --- | --- |
| **Infrastructure as Code (IaC)** | 서버를 직접 세팅하는 것이 아니라, YAML 파일 하나로 서버 환경을 정의하는 현대적 개발 문화를 경험합니다. |
| **[Skip CI] 전략** | 자동 커밋 메시지에 `[skip ci]`를 넣지 않으면, 로봇이 커밋한 것에 반응해 로봇이 다시 작동하는 **무한 루프**에 빠질 수 있습니다. 이 레포의 YAML에는 예방 로직이 포함되어 있습니다. |
| **GitHub Pages 배포** | 레포지토리 `Settings > Pages`에서 배포 소스를 `Deploy from a branch`로 설정하고 Actions가 푸시하는 브랜치(`main`)를 지정하면 설계가 완료됩니다. |
| **정적 사이트 아키텍처** | 서버 없이 JSON 파일 하나를 "데이터베이스"로 활용하는 패턴을 이해합니다. |

---

## 🛠 Tech Stack

| **역할** | **기술** | **비고** |
| --- | --- | --- |
| **Storage & Hosting** | GitHub Repository | 1GB 미만 적정 |
| **Automation** | GitHub Actions | 무료 티어 |
| **PDF Engine** | `pdf2image` (Python), `Poppler` | Linux System Lib |
| **Frontend** | GitHub Pages | `index.html`, `viewer.html` |
| **Main UI** | 3D Bookshelf (CSS3D / JS) | [virtual-bookshelf](https://github.com/petargyurov/virtual-bookshelf) 스타일 |
| **Viewer** | HTML5, `page-flip` | [Flipbook Effect](https://www.npmjs.com/package/page-flip) |
| **CI/CD** | GitHub-native 또는 Local Server |  |

---

## 📁 File Tree Structure

```
pdf-flip/
├── .github/
│   └── workflows/
│       └── update-library.yml  # [Automation] GitHub Actions 파이프라인 설계도
├── pdf-source/                 # [Input]  사용자가 PDF를 업로드하는 곳
├── pdf-image/                  # [Output] 로봇(Actions)이 생성한 이미지 저장소
│   └── {book-name}/
│       ├── page-01.jpg
│       ├── page-02.jpg
│       └── ...
├── assets/                     # 공통 에셋
│   ├── style.css               # 3D Bookshelf 및 플립북 스타일
│   └── script.js               # books.json 로드 및 렌더링 로직
├── index.html                  # [Main]   3D Virtual Bookshelf (Home)
├── viewer.html                 # [Viewer] Page-flip 기반 플립북 화면
├── books.json                  # [Database] 로봇이 자동 갱신하는 메타데이터
├── pdf_processor.py            # [Logic]  PDF 변환 및 JSON 생성 스크립트
├── requirements.txt            # [Env]    Python 패키지 명세 (pdf2image, pillow)
└── README.md
```

---

## ⚙️ Engine 핵심 로직

### Pre-processing (`pdf_processor.py`)

1. `pdf-source/` 폴더의 PDF 파일을 감지합니다.
2. 각 PDF 파일명을 딴 폴더를 `pdf-image/` 하위에 생성합니다. (예: `pdf-image/my-book/`)
3. PDF 페이지를 `page-01.jpg`, `page-02.jpg` 형태의 이미지로 변환합니다. (300 DPI)

### Cataloging

변환 완료 후, 메인 화면에서 사용할 `books.json` 파일을 자동 생성/갱신합니다.

- 포함 정보: 책 제목, 썸네일 경로, 전체 페이지 수, 이미지 경로 배열

### Rendering

| **화면** | **파일** | **동작** |
| --- | --- | --- |
| **Main** | `index.html` | `books.json`을 읽어 3D 책장에 책을 진열. [virtual-bookshelf](https://github.com/petargyurov/virtual-bookshelf) 참고. |
| **Viewer** | `viewer.html?book=filename` | 책 클릭 시 이동하여 [page-flip](https://www.npmjs.com/package/page-flip) 기반 플립북 렌더링. |

---

## 🔄 CI/CD UX 시나리오

사용자는 로컬에서 복잡한 스크립트를 실행할 필요가 없습니다.

👉 **"GitHub 웹사이트나 데스크톱 앱에서 `pdf-source/` 폴더에 PDF를 업로드(Push)한다."**

나머지는 **GitHub Actions**라는 클라우드 로봇이 수행합니다.

```
[Your PDF] ──push──▶ [pdf-source/ 폴더]
                              │ Trigger
                     [GitHub Actions]
                     [ubuntu-latest VM]
                              │
              ┌───────────────┼───────────────┐
           Install         Process          Commit
        Poppler+Python  pdf_processor.py  [skip ci]
                                             │
                                    [GitHub Pages]
                                   (자동 웹 배포 완료)
```

| **단계** | **설명** |
| --- | --- |
| **[Trigger]** | `pdf-source/` 폴더에 변화가 생기면 로봇이 가동됩니다. |
| **[Environment]** | 가상 서버에 Python과 PDF 처리 엔진(Poppler)을 자동 설치합니다. |
| **[Process]** | `pdf_processor.py`를 실행하여 이미지를 추출하고 `books.json`을 갱신합니다. |
| **[Auto-Commit]** | 생성된 이미지와 JSON을 내 레포지토리에 자동으로 다시 커밋합니다. |
| **[Deploy]** | 업데이트된 내용을 바탕으로 GitHub Pages가 즉시 웹에 게시합니다. |

### 핵심 파일: `.github/workflows/update-library.yml`

이 파일이 전체 파이프라인의 설계도입니다.

```yaml
name: Update PDF Library

on:
  push:
    paths:
      - 'pdf-source/**'  # PDF 폴더에 변화가 있을 때만 실행

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Install System Dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y poppler-utils  # PDF 변환 핵심 엔진

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - name: Install Python Dependencies
        run: |
          pip install -r requirements.txt

      - name: Run Processor
        run: python pdf_processor.py

      - name: Commit and Push Changes
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
          git add pdf-image/ books.json
          git commit -m "chore: auto-generate images from pdf [skip ci]" || echo "No changes to commit"
          git push
```

<aside>
💡

**[skip ci] 전략**: 커밋 메시지에 `[skip ci]`가 없으면 Actions가 자신의 커밋에 반응해 무한 루프가 발생합니다. 위 YAML에는 해당 예방 로직이 포함되어 있습니다.

</aside>

---

## 🚀 Usage: Fork하여 사용하기 (권장)

GitHub에서 직접 사용하는 가장 간단한 방법입니다. 로컬 환경 설정이 필요 없습니다.

### 1단계: 레포지토리 Fork

1. 이 페이지 우측 상단의 **Fork** 버튼을 클릭합니다.
2. 자신의 GitHub 계정에 레포지토리가 복사됩니다.

### 2단계: GitHub Pages 활성화

1. Fork된 레포지토리의 **Settings** 탭으로 이동합니다.
2. 좌측 메뉴에서 **Pages**를 선택합니다.
3. **Source**를 `Deploy from a branch`로 설정합니다.
4. Branch를 `main` / `(root)`으로 지정하고 **Save**합니다.

<aside>
⚠️

Pages가 활성화되지 않으면 Actions가 배포를 완료해도 사이트가 열리지 않습니다.

</aside>

### 3단계: Actions 권한 확인

1. **Settings > Actions > General**로 이동합니다.
2. **Workflow permissions**를 `Read and write permissions`로 설정합니다.
3. **Save**를 클릭합니다.

<aside>
⚠️

이 설정이 없으면 Actions가 변환된 이미지를 레포에 자동 커밋할 수 없습니다.

</aside>

### 4단계: PDF 업로드

1. `pdf-source/` 폴더로 이동합니다.
2. **Add file > Upload files**를 클릭하여 PDF 파일을 업로드합니다.
3. **Commit changes**를 클릭합니다.
4. **Actions** 탭에서 파이프라인이 자동 실행되는 것을 확인합니다.
5. 완료 후 `https://YOUR_USERNAME.github.io/pdf-flip`에서 결과를 확인합니다.

---

## 💻 Usage: 로컬 개발 환경 설정

로컬에서 직접 개발하거나 커스터마이징할 때 사용합니다.

### 사전 요구사항

- Python 3.10 이상
- `poppler-utils` (시스템 패키지)

```bash
# macOS
brew install poppler

# Ubuntu / Debian
sudo apt-get install -y poppler-utils

# Windows
# https://github.com/oschwartz10612/poppler-windows 에서 바이너리 다운로드 후 PATH 설정
```

### 설치 및 실행

**1. 레포지토리 클론**

```bash
git clone https://github.com/YOUR_USERNAME/pdf-flip.git
cd pdf-flip
```

**2. 가상 환경 생성 및 활성화**

```bash
python3 -m venv .venv
source .venv/bin/activate      # macOS / Linux
# .venv\Scripts\activate       # Windows
```

**3. Python 패키지 설치**

```bash
pip install -r requirements.txt
```

**4. PDF 파일 추가**

`pdf-source/` 디렉토리에 `.pdf` 파일을 넣습니다.

```bash
cp your-document.pdf pdf-source/
```

**5. PDF 처리 스크립트 실행**

```bash
python3 pdf_processor.py
```

`pdf-image/` 하위에 이미지가 생성되고 `books.json`이 갱신됩니다.

**6. 로컬 서버 실행**

```bash
python3 -m http.server 3030
```

**7. 브라우저에서 확인**

```
http://localhost:3030
```

---

## 🌐 GitHub Pages 배포 설정

로컬 작업 결과를 GitHub Pages에 배포하려면 변경사항을 푸시합니다.

```bash
git add .
git commit -m "feat: add new books"
git push origin main
```

Actions가 자동 실행되어 변환 및 배포까지 완료합니다. 진행 상황은 레포지토리의 **Actions** 탭에서 실시간으로 확인할 수 있습니다.

---

## 🗂 books.json 스키마

`pdf_processor.py`가 자동 생성하는 `books.json`의 구조입니다.

```json
[
  {
    "id": "my-book",
    "title": "my-book",
    "thumbnail": "pdf-image/my-book/page-01.jpg",
    "pageCount": 42,
    "pages": [
      "pdf-image/my-book/page-01.jpg",
      "pdf-image/my-book/page-02.jpg",
      "..."
    ]
  }
]
```

| **필드** | **타입** | **설명** |
| --- | --- | --- |
| `id` | `string` | PDF 파일명 기반 고유 식별자 |
| `title` | `string` | 책장에 표시될 제목 |
| `thumbnail` | `string` | 커버 이미지 경로 (1페이지) |
| `pageCount` | `number` | 전체 페이지 수 |
| `pages` | `string[]` | 순서대로 정렬된 페이지 이미지 경로 배열 |

---

## ❓ FAQ / 트러블슈팅

- **Actions가 실행되지 않습니다.**
    - `Settings > Actions > General > Workflow permissions`가 `Read and write`로 설정되어 있는지 확인합니다.
    - `pdf-source/` 폴더에 파일 변화가 실제로 발생했는지 확인합니다. (동일 파일 재업로드는 트리거되지 않을 수 있습니다.)
- **Actions는 성공했는데 사이트가 보이지 않습니다.**
    - `Settings > Pages`에서 배포 소스가 올바르게 설정되어 있는지 확인합니다.
    - GitHub Pages가 활성화된 직후에는 수 분의 초기 배포 시간이 필요합니다.
- **이미지 변환 품질을 조정하고 싶습니다.**
    
    `pdf_processor.py` 내의 DPI 값을 수정합니다. 기본값은 `300`이며, 높을수록 품질이 좋아지지만 파일 크기와 처리 시간이 증가합니다.
    

```python
# pdf_processor.py
images = convert_from_path(pdf_path, dpi=300)  # 이 값을 조정
```

- **레포지토리 용량이 1GB를 초과할 것 같습니다.**
    
    GitHub 무료 레포는 1GB 소프트 제한이 있습니다. PDF 품질(DPI)을 낮추거나, 오래된 `pdf-image/` 내 폴더를 주기적으로 정리하는 것을 권장합니다.
    

---

## 🤝 Contributing

기여를 환영합니다. 아래 절차를 따릅니다.

1. 이 레포지토리를 **Fork**합니다.
2. 새 브랜치를 생성합니다. (`git checkout -b feature/your-feature`)
3. 변경사항을 커밋합니다. (`git commit -m 'feat: add some feature'`)
4. 브랜치에 Push합니다. (`git push origin feature/your-feature`)
5. **Pull Request**를 생성합니다.

### 커밋 메시지 컨벤션

[Conventional Commits](https://www.conventionalcommits.org/) 규칙을 따릅니다.

| **접두사** | **용도** |
| --- | --- |
| `feat:` | 새로운 기능 추가 |
| `fix:` | 버그 수정 |
| `docs:` | 문서 수정 |
| `chore:` | 빌드/설정 관련 변경 (자동화 커밋 포함) |
| `style:` | 코드 스타일, 포맷 변경 |

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).