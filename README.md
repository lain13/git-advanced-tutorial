# Git 고급 명령어 반복 연습 튜토리얼

## 프로젝트 소개

Git의 기본 명령어는 알지만 merge conflict, rebase, cherry-pick, stash 등의 고급 명령어에 익숙하지 않은 개발자를 위한 반복 연습용 튜토리얼입니다.

### 학습 목표
- Merge conflict 해결 능력 체화
- Rebase (기본, interactive) 완벽 이해
- Cherry-pick (단일, 범위) 자유자재로 활용
- Stash 관리 마스터
- Revert & Reset 차이점 이해 및 활용

### 예상 소요시간
약 1시간 (단계별 반복 연습 포함)

---

## Phase 1: 환경 설정

### 1.1 GitHub 리포지터리 생성
1. GitHub에서 새 리포지터리 생성합니다. (`git-advanced-tutorial`)
2. README.md 파일을 포함하여 생성합니다.

### 1.2 로컬 환경 구성
```bash
# Git 기본 설정 (필수)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 리포지터리 클론
git clone https://github.com/YOUR_USERNAME/git-advanced-tutorial.git
cd git-advanced-tutorial

# 초기 파일 생성
echo "console.log('Hello World');" > app.js
echo "body { margin: 0; }" > styles.css
echo "<html><head><title>Tutorial</title></head><body><h1>Git Tutorial</h1></body></html>" > index.html

# 초기 커밋
git add .
git commit -m "Initial project setup"
git push origin main
```

### 1.3 상태 확인
```bash
git log --oneline
git status
```
**예상 결과**: main 브랜치에 "Initial project setup" 커밋이 추가되어 있어야 함

---

## Phase 2: 기본 워크플로우

### 2.1 Feature 브랜치 생성 및 작업
```bash
# feature 브랜치 생성
git checkout -b feature/login
echo "function login() { console.log('Login function'); }" >> app.js
git add app.js
git commit -m "Add login function"

# 추가 작업
echo "function logout() { console.log('Logout function'); }" >> app.js
git add app.js
git commit -m "Add logout function"

# 브랜치 확인
git log --oneline
```

### 2.2 기본 Merge 연습
```bash
# main으로 돌아가서 merge
git checkout main
git merge feature/login
git push origin main

# 브랜치 정리
git branch -d feature/login
```

### 2.3 상태 확인
```bash
git log --oneline --graph
```
**예상 결과**: main 브랜치에 3개의 커밋 (initial + login + logout)

---

## Phase 3: Merge Conflict 해결

### 3.1 기본 Conflict 해결
```bash
# 첫 번째 브랜치에서 작업
git checkout -b feature/user-management
echo "function getUser() { return 'user data'; }" >> app.js
git add app.js
git commit -m "Add getUser function"

# main으로 돌아가서 같은 부분 수정
git checkout main
echo "function getUser() { return 'current user'; }" >> app.js
git add app.js
git commit -m "Add getUser function with different implementation"

# conflict 발생시키기
git merge feature/user-management
```

**수동 Conflict 해결**:
1. `app.js` 파일을 에디터로 열기(notepad app.js)
2. `<<<<<<<`, `=======`, `>>>>>>>` 마커 확인
3. 원하는 버전 선택하거나 수동으로 병합
4. 마커 제거 후 저장

```bash
# conflict 해결 후
git add app.js
git commit -m "Resolve merge conflict manually"
git branch -d feature/user-management
```

### 3.2 --ours 전략 연습 (내 변경사항 유지)
```bash
# 새로운 conflict 상황 생성
git checkout -b feature/styles-update
echo ".button { background: blue; color: white; }" >> styles.css
git add styles.css
git commit -m "Add blue button styles"

git checkout main
echo ".button { background: red; color: black; }" >> styles.css
git add styles.css
git commit -m "Add red button styles"

# --ours 전략으로 내 변경사항(main) 유지
git merge -X ours feature/styles-update
git push origin main
git branch -d feature/styles-update
```

### 3.3 --theirs 전략 연습 (상대방 변경사항 적용)
```bash
# 또 다른 conflict 상황 생성
git checkout -b feature/navigation
echo "<nav>New Navigation</nav>" > navigation.html
git add navigation.html
git commit -m "Add new navigation"

git checkout main
echo "<nav>Main Navigation</nav>" > navigation.html
git add navigation.html
git commit -m "Add main navigation"

# --theirs 전략으로 feature 브랜치 변경사항 적용
git merge -X theirs feature/navigation
git push origin main
git branch -d feature/navigation
```

### 3.4 Fetch 후 Merge 연습
```bash
# 원격 변경사항 시뮬레이션을 위한 추가 작업
git checkout -b temp-remote-work
echo "function remoteFeature() { console.log('Remote work'); }" >> app.js
git add app.js
git commit -m "Remote team work simulation"
git push origin temp-remote-work

# main에서 로컬 작업
git checkout main
echo "function localFeature() { console.log('Local work'); }" >> app.js
git add app.js
git commit -m "Local work while others are working"

# fetch 후 merge 연습
git fetch origin
git merge origin/temp-remote-work
```

**Conflict 해결 및 수동 커밋**:
1. `app.js` 파일을 에디터로 열기(notepad app.js)
2. conflict 마커(`<<<<<<<`, `=======`, `>>>>>>>`) 제거
3. 두 함수 모두 유지하도록 수정
4. 저장 후 단계별로 진행

```bash
# conflict 해결 후
git add app.js

# 커밋하기 전에 상태 확인 (staged 상태 확인)
git status
git diff --cached

# 수동으로 커밋 (merge 메시지 커스터마이징 가능)
git commit -m "Merge remote work: integrate both local and remote features"

# 정리
git push origin main
git push origin --delete temp-remote-work
git branch -d temp-remote-work
```

---

## Phase 4: Rebase 연습

### 4.1 기본 Rebase 실습
```bash
# feature 브랜치 생성 및 작업
git checkout -b feature/dashboard
echo "function renderDashboard() { console.log('Dashboard'); }" >> app.js
git add app.js
git commit -m "Add dashboard render function"

echo "function updateDashboard() { console.log('Update dashboard'); }" >> app.js
git add app.js
git commit -m "Add dashboard update function"

# main에서 별도 작업 (시간차 시뮬레이션)
git checkout main
echo "/* Main branch update */" >> app.js
git add app.js
git commit -m "Main branch maintenance"

# rebase 실행
git checkout feature/dashboard
git rebase main
```

### 4.2 Interactive Rebase 실습
```bash
# 여러 커밋 생성
echo "function helper1() { console.log('helper1'); }" >> app.js
git add app.js
git commit -m "Add helper1 - typo fix needed"

echo "function helper2() { console.log('helper2'); }" >> app.js
git add app.js
git commit -m "Add helper2"

echo "function helper3() { console.log('helper3'); }" >> app.js
git add app.js
git commit -m "Add helper3 - will be squashed"

# Interactive rebase 실행 (최근 3개 커밋)
git rebase -i HEAD~3
```

**Interactive rebase 연습**:
- `pick` → `squash`: 커밋 합치기
- `pick` → `reword`: 커밋 메시지 수정
- `pick` → `edit`: 커밋 수정
- 순서 변경하기

### 4.3 Rebase 완료 및 병합
```bash
git checkout main
git merge feature/dashboard
git branch -d feature/dashboard
git push origin main
```

---

## Phase 5: Cherry-pick 연습

### 5.1 Cherry-pick 준비
```bash
# experimental 브랜치 생성
git checkout -b experimental
echo "function experimentalFeature1() { console.log('exp1'); }" >> app.js
git add app.js
git commit -m "Experimental feature 1"

echo "function experimentalFeature2() { console.log('exp2'); }" >> app.js
git add app.js
git commit -m "Experimental feature 2"

echo "function experimentalFeature3() { console.log('exp3'); }" >> app.js
git add app.js
git commit -m "Experimental feature 3"

# 커밋 해시 확인
git log --oneline -3
```

### 5.2 단일 Cherry-pick
```bash
git checkout main
# 두 번째 experimental feature만 가져오기 (실제 해시로 교체)
git cherry-pick <COMMIT_HASH_OF_FEATURE2>
```

### 5.3 범위 Cherry-pick
```bash
# 여러 커밋을 범위로 cherry-pick
git cherry-pick <FIRST_HASH>^..<LAST_HASH>
```

### 5.4 Cherry-pick 옵션 활용
```bash
# --no-commit 옵션으로 staging만
git cherry-pick --no-commit <COMMIT_HASH>
git status
git commit -m "Cherry-picked and modified"

# -x 옵션으로 원본 커밋 정보 포함
git cherry-pick -x <COMMIT_HASH>
```

---

## Phase 6: Stash 연습

### 6.1 기본 Stash 사용
```bash
# 작업 중인 변경사항 생성
echo "function workInProgress() { console.log('WIP'); }" >> app.js
echo ".wip { background: yellow; }" >> styles.css

# 변경사항 확인
git status

# stash 저장
git stash save "Work in progress on new features"

# clean working directory 확인
git status
```

### 6.2 Stash 관리
```bash
# 다른 작업 시뮬레이션
echo "function quickFix() { console.log('Quick fix'); }" >> app.js
git add app.js
git commit -m "Quick fix while working on features"

# 또 다른 stash 생성
echo "function anotherWIP() { console.log('Another WIP'); }" >> app.js
git stash save "Another work in progress"

# stash 목록 확인
git stash list
```

### 6.3 Stash 복원 및 활용
```bash
# 특정 stash 적용 (유지)
git stash apply stash@{1}

# stash 적용 후 제거
git stash pop stash@{0}

# 특정 stash 제거
git stash drop stash@{0}

# 모든 stash 제거
git stash clear
```

---

## Phase 7: Revert & Reset 연습

### 7.1 Revert 연습
```bash
# 문제가 있는 커밋 생성
echo "function buggyFunction() { throw new Error('Bug!'); }" >> app.js
git add app.js
git commit -m "Add buggy function - this will be reverted"

echo "function goodFunction() { console.log('Good'); }" >> app.js
git add app.js
git commit -m "Add good function"

# 커밋 히스토리 확인
git log --oneline -5

# 문제 커밋만 revert
git revert <BUGGY_COMMIT_HASH>
```

### 7.2 Reset 연습
```bash
# 테스트용 커밋들 생성
echo "// Test 1" >> app.js
git add app.js
git commit -m "Test commit 1"

echo "// Test 2" >> app.js
git add app.js
git commit -m "Test commit 2"

# 현재 상태 확인
git log --oneline -3

# Soft reset (커밋만 취소, 변경사항 유지)
git reset --soft HEAD~1
git status

# Mixed reset (default, 커밋과 staging 취소)
git reset HEAD~1
git status

# Hard reset (모든 변경사항 삭제 - 주의!)
git reset --hard HEAD~1
git status
```

### 7.3 범위 Revert
```bash
# 여러 커밋을 한 번에 revert
git revert --no-commit HEAD~3..HEAD
git commit -m "Revert multiple commits"
```

---

---

## 최종 정리

### 생성된 파일들 정리
```bash
# 튜토리얼 과정에서 생성된 추가 파일들 삭제
git rm -f app.js index.html navigation.html styles.css

# 변경사항 커밋
git add .
git commit -m "Clean up tutorial files"
git push origin main

# 최종 상태 확인
git status
git log --oneline -5
git branch --all
echo "Tutorial completed! Repository is clean and ready for re-practice."
```

---

## 트러블슈팅

### 자주 발생하는 문제
1. **Merge conflict 해결 후 계속 충돌 상태**: `git status`로 모든 파일이 staged 되었는지 확인
2. **Rebase 중 멈춤**: `git rebase --continue` 또는 `git rebase --abort`
3. **Cherry-pick 실패**: 충돌 해결 후 `git cherry-pick --continue`
4. **Reset으로 실수한 경우**: `git reflog`로 복구 가능

### 추가 연습 제안
1. **Worktree**: `git worktree add` 를 사용한 멀티 워킹 디렉토리 관리
2. **Bisect**: `git bisect`를 사용한 버그 찾기
3. **Reflog**: `git reflog`를 사용한 히스토리 복구

### 실무 팁
- **안전한 실험**: 중요한 프로젝트에서는 항상 백업 브랜치 생성
- **커밋 메시지**: 명확하고 일관된 커밋 메시지 작성
- **브랜치 전략**: Git Flow나 GitHub Flow 같은 브랜치 전략 도입
- **훅 활용**: pre-commit, pre-push 훅으로 코드 품질 자동 검사

이 튜토리얼을 완주하면 Git의 고급 명령어들을 자신있게 사용할 수 있을 것입니다. 각 명령어를 최소 3회씩 반복하여 근육 기억으로 체화시키는 것이 중요합니다.