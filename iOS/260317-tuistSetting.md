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
* **배운 점 (Git의 안전성):** * `git pull`은 코드를 무작정 덮어씌워 삭제하는 것이 아니라, 원격과 로컬의 두 역사를 하나로 **결합(Merge)**하는 과정임. 
  * 로컬 데이터의 무결성을 유지하려는 Git의 설계 철학을 이해함.
     * "내 컴퓨터(로컬)에 있는 코드는 무조건 안전해야 한다."
     * "서버에서 새로운 파일이 내려오더라도, 내 기존 코드를 밀어버리는 게 아니라 빈 공간에 예쁘게 덧붙여(Merge)주어야 한다."
     * "만약 서버 내용과 내 코드가 같은 줄을 건드려서 자동으로 합칠 수 없다면? 맘대로 덮어쓰지 말고, 개발자에게 에러(Conflict)를 띄워 직접 눈으로 보고 선택하게 만들어야 한다."
  * 프로젝트 초기 세팅 단계에서 로컬 구조가 확실한 정답일 경우, 강제 푸쉬(`git push -f`)를 통해 히스토리를 깔끔하게 동기화할 수 있음을 배움.
