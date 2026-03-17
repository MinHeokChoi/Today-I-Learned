# [iOS/Tuist] 다중 프로젝트 환경(Workspace) 구축 및 Git/Xcode 트러블슈팅

## 🗓 학습 날짜
- 2026년 3월 18일

## 💡 학습 내용 및 문제 해결 과정

### 1. Tuist 기반 Workspace (다중 프로젝트) 구조 설계
* **개념:** 하나의 Git 저장소 안에서 여러 개의 독립된 앱(예: 과제용 앱, 실습용 앱)을 개발할 때, Tuist의 `Workspace` 개념을 활용하여 통합 관리하는 아키텍처를 구축함.
* **구조 설계 원칙 (단일 책임 원칙):**
  * Git 최상단(Root)에 `Workspace.swift`를 배치하여 하위 폴더의 모든 프로젝트를 하나로 묶음.
  * 하위의 각 앱 폴더(예: `UMC_MegaBox`, `Practice`)는 자신만의 독립적인 설계도인 `Project.swift`를 가짐.

### 2. Tuist 초기화 및 Project.swift 세팅
* 빈 폴더에서 새롭게 프로젝트를 구성할 때는 `tuist init`을 통해 기본 구조(`Sources`, `Resources`, `Project.swift` 등)를 자동 생성함.
* 기존 프로젝트에 Tuist를 붙일 때는 `tuist edit`을 통해 `Project.swift`의 `sources`, `resources` 경로를 실제 파일 시스템 구조와 정확히 매핑해야 함. (예: `sources: ["Sources/**"]`)

### 3. Xcode CLI 경로 재설정 (xcode-select)
* **문제:** `tuist edit` 실행 시 `Couldn't find Xcode's Info.plist...` 에러 발생.
* **원인:** 웹에서 직접 다운로드한 Xcode를 응용 프로그램 폴더로 옮기는 과정에서, macOS 시스템의 Command Line Tools 경로가 꼬여 CLI 도구(Tuist)가 컴파일러를 찾지 못함.
* **해결:** 터미널에서 `sudo xcode-select -s /Applications/Xcode.app` 명령어를 통해 개발 도구의 표준 경로를 다시 연결하고, Xcode 앱 최초 실행(Verifying)을 통해 시스템 권한을 갱신함.

### 4. Git 병합(Merge) 원리와 Push/Pull 트러블슈팅
* **문제:** 서버(origin)에 로컬 코드를 Push하려 할 때 `The local repository is out of date` 경고와 함께 거부됨.
* **원인:** GitHub에서 생성된 초기 파일(README 등)의 원격 히스토리와 로컬 노트북의 히스토리가 불일치하여 발생.
* **배운 점 (Git의 안전성):** * 배운 점 (Git의 든든한 안전장치 역할): * git pull은 서버에 있는 파일을 무작정 다운로드해서 내 컴퓨터에 '덮어쓰기(Overwrite)' 해버리는 위험한 버튼이 아님을 깨달음. 
  * 깃(Git)은 내가 노트북에서 열심히 작성한 소중한 코드가 서버 데이터에 의해 날아가지 않도록 철저히 보호해 줌. (이것이 바로 깃이 '데이터의 무결성'을 지키는 방식)
  * 코드를 덮어쓰는 대신, 서버의 역사와 내 노트북의 역사를 안전한 빈 공간에 예쁘게 **'결합(Merge)'**하려고 시도함. 만약 코드가 꼬일 위험이 있으면 함부로 합치지 않고 에러를 띄워 개발자에게 먼저 물어봄.
  * 단, 지금처럼 '내 노트북에 있는 초기 세팅 파일들이 100% 정답'인 특수한 상황에서는, 깃의 경고를 끄고 강제 푸쉬(git push -f)를 날려 서버를 내 노트북 상태로 깔끔하게 맞추는 유연성도 필요하다는 것을 배움.
