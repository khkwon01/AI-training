# Chapter 06: merge conflict 이해와 해결

## 이 장에서 배우는 것

- merge conflict가 왜 발생하는지 실제 상황으로 이해하기
- conflict 표시(`<<<<<<<`, `=======`, `>>>>>>>`)를 읽고 해석하는 방법
- 터미널에서 conflict를 직접 해결하는 방법
- VS Code의 "Accept Current Change / Accept Incoming Change" 버튼 사용법
- 3-way merge가 무엇인지 개념 이해
- conflict를 의도적으로 만들어 해결하는 실습

---

## 왜 merge conflict를 알아야 할까

처음 Git을 배우면 conflict가 발생했을 때 당황해서 작업을 포기하거나, 상대방 코드를 무작정 덮어쓰는 실수를 하는 경우가 많다.

하지만 **conflict는 버그가 아니다.** Git이 "두 가지 버전이 동시에 존재하는데, 어느 것을 선택할지 사람이 결정해줘야 해"라고 알려주는 신호다.

conflict를 올바르게 해결하면:
- 두 사람의 작업이 모두 보존된다
- 코드를 잃어버리지 않는다
- 팀원과의 변경 내용을 명시적으로 확인하고 통합한다

실무에서 conflict는 매일 생길 수 있다. 무서워하지 말고 정해진 순서대로 처리하면 된다.

---

## 1. merge conflict가 발생하는 상황

### 1-1. 기본 원리

두 사람이 **같은 파일의 같은 줄**을 서로 다르게 수정한 후 merge를 시도하면 conflict가 발생한다.

예를 들어, `greeting.py`에 이런 코드가 있다고 하자.

```python
def greet():
    print("안녕하세요")
```

- **사람 A** (main branch에서 작업): `print("안녕하세요")` → `print("Hello")`로 수정
- **사람 B** (feature branch에서 작업): `print("안녕하세요")` → `print("안녕!")`로 수정

A가 먼저 main에 push했다. B가 push하거나 merge를 시도하면 Git은 판단할 수 없다.

```
공통 조상(같은 줄):  print("안녕하세요")
main (HEAD):         print("Hello")
feature branch:      print("안녕!")
                             ↓
Git이 묻는다: "어느 걸 써야 해?"
```

### 1-2. commit 그래프로 보기

```
main:     A ── B ── C
                \
feature:         D ── E
```

- A: 공통 시작점
- B, C: main branch에서 한 commit들 (`greeting.py` 수정 포함)
- D, E: feature branch에서 한 commit들 (같은 파일 같은 줄 수정 포함)

E까지 작업한 feature를 main에 merge하려 할 때:
- C와 E가 같은 줄을 다르게 수정했다면 → conflict 발생

### 1-3. 혼자 작업할 때도 conflict가 생긴다

conflict는 꼭 두 사람이 협업할 때만 생기는 게 아니다.

혼자 작업할 때도:
- GitHub 웹에서 파일을 직접 수정한 뒤, 로컬에서도 같은 파일을 수정하고 push하면 발생한다
- 작업용 branch와 main branch 둘 다 같은 파일을 수정하면 발생한다

---

## 2. conflict 표시 읽기

conflict가 발생한 파일을 열면 Git이 직접 파일 안에 표시를 넣는다.

```python
<<<<<<< HEAD
print("Hello")
=======
print("안녕!")
>>>>>>> feature/update-greeting
```

이 표시를 하나씩 분석해보자.

### 2-1. 각 부분의 의미

```
<<<<<<< HEAD
```
"현재 내가 있는 branch(HEAD)의 내용이 여기서 시작된다"는 표시다. `HEAD`는 보통 main branch를 가리킨다.

```
=======
```
두 버전을 나누는 구분선이다. 위가 현재 branch, 아래가 합치려는 branch의 내용이다.

```
>>>>>>> feature/update-greeting
```
"합치려는 branch(`feature/update-greeting`)의 내용이 여기서 끝난다"는 표시다.

### 2-2. 전체 구조 요약

| 부분 | 의미 |
|------|------|
| `<<<<<<< HEAD` ~ `=======` | 현재 branch (내가 commit한 내용) |
| `=======` ~ `>>>>>>> 브랜치명` | 합치려는 branch (상대방이 commit한 내용) |

### 2-3. 여러 곳에 conflict가 생기는 경우

파일 하나에 conflict가 여러 곳에 생길 수 있다. 각 conflict는 독립적으로 처리해야 한다.

```python
<<<<<<< HEAD
def greet():
    print("Hello")
=======
def greet():
    print("안녕!")
>>>>>>> feature/update-greeting

# 중간에 conflict 없는 코드가 있을 수 있음
x = 10

<<<<<<< HEAD
print("프로그램 시작")
=======
print("시작합니다")
>>>>>>> feature/update-greeting
```

이 파일에는 conflict가 2개 있다. 둘 다 해결해야 commit할 수 있다.

### 2-4. conflict를 확인하는 명령어

```bash
git status
```

실행 결과에 아래처럼 표시된다:

```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   greeting.py
```

`both modified`는 "두 branch 모두 이 파일을 수정했다"는 뜻이다. conflict가 남아있는 파일 목록이다.

---

## 3. 3-way merge 개념

Git이 conflict를 감지하는 방식이 **3-way merge**다.

```
1. 공통 조상 (두 branch가 갈라지기 전 마지막 공통 commit)
2. 현재 branch (HEAD)의 최신 상태
3. 합치려는 branch의 최신 상태
```

Git은 이 세 가지를 비교한다.

- 한 쪽만 변경됐으면: 변경된 쪽을 자동으로 채택 (conflict 없음)
- 양쪽 다 변경됐으면: conflict 표시를 파일에 삽입 → 사람이 결정해야 함

```
공통 조상:   print("안녕하세요")
HEAD:        print("Hello")           ← 한쪽이 바꿨음
feature:     print("안녕하세요")      ← 이쪽은 그대로
결과:        자동으로 print("Hello") 채택 → conflict 없음

공통 조상:   print("안녕하세요")
HEAD:        print("Hello")           ← 양쪽 모두 바꿨음
feature:     print("안녕!")           ← 양쪽 모두 바꿨음
결과:        conflict → 사람이 결정해야 함
```

이 개념을 알면 "왜 어떤 건 자동으로 합쳐지고 어떤 건 conflict가 생기는지" 이해할 수 있다.

---

## 4. 터미널에서 conflict 해결하기

### 4-1. 해결 방법 3가지

**방법 1: 현재 branch(HEAD) 내용을 선택**

상대방 내용을 버리고 내 코드를 유지한다.

```python
# 이렇게 수정
print("Hello")
```

**방법 2: 합치려는 branch 내용을 선택**

내 내용을 버리고 상대방 코드를 유지한다.

```python
# 이렇게 수정
print("안녕!")
```

**방법 3: 두 내용을 모두 반영해서 새로 작성**

어느 쪽도 그대로 쓰지 않고, 두 의도를 합쳐 새로 작성한다.

```python
# 이렇게 수정
print("Hello, 안녕!")
```

### 4-2. 반드시 해야 할 것

어떤 방법을 선택했든, 반드시 `<<<<<<`, `=======`, `>>>>>>>` 마커를 모두 지워야 한다. 이 마커가 파일에 남아있으면 코드가 정상 동작하지 않는다.

파일을 저장한 뒤 마커가 남아있는지 확인하는 방법:

```bash
grep -n "<<<<<<" 파일이름.py
```

아무것도 출력되지 않으면 마커가 모두 제거된 것이다.

### 4-3. 해결 후 commit

```bash
# conflict가 해결된 파일을 stage
git add greeting.py

# 모든 conflict가 해결됐는지 확인
git status
# "nothing to commit" 또는 "All conflicts fixed" 확인

# merge commit 생성
git commit -m "merge conflict 해결: greeting 방식 통일"
```

> `git commit`을 실행했을 때 편집기(vim이나 nano)가 열리면서 기본 merge commit 메시지가 표시되는 경우: 내용을 그대로 두고 저장하고 닫으면 된다. vim이라면 `:wq`를 입력하고 Enter를 누른다.

---

## 5. VS Code에서 시각적으로 해결하기

VS Code는 conflict를 색으로 표시하고 클릭 한 번으로 해결할 수 있게 해준다. 텍스트를 직접 편집하는 것보다 훨씬 편하다.

### 5-1. conflict 파일 열기

Source Control 패널을 열면 conflict 파일이 `C` 아이콘과 함께 빨간색으로 표시된다. 해당 파일을 클릭해서 에디터에서 연다.

### 5-2. 버튼 위치와 의미

파일을 열면 conflict 영역 바로 위에 4개의 링크(클릭 가능한 텍스트)가 나타난다.

```
                  [Accept Current Change] [Accept Incoming Change] [Accept Both Changes] [Compare Changes]
<<<<<<< HEAD
print("Hello")
=======
print("안녕!")
>>>>>>> feature/update-greeting
```

| 버튼 | 선택되는 내용 | 언제 사용하나 |
|------|-------------|-------------|
| **Accept Current Change** | `<<<<<<< HEAD` 아래 내용 (내 코드) | 내 변경이 맞고 상대방 것은 필요 없을 때 |
| **Accept Incoming Change** | `=======` 아래 내용 (합치려는 branch) | 상대방 변경이 맞고 내 것은 필요 없을 때 |
| **Accept Both Changes** | 두 내용 모두 보존 (위 + 아래) | 두 변경 모두 필요할 때 |
| **Compare Changes** | 나란히 비교 창 열림 | 두 버전의 차이를 자세히 보고 싶을 때 |

버튼을 클릭하면 자동으로 `<<<`, `===`, `>>>` 마커가 제거되고 선택한 내용만 남는다.

### 5-3. Accept Both Changes 사용 후 수동 조정

**Accept Both Changes**를 클릭하면 두 내용이 위아래로 합쳐진다.

```python
# Accept Both Changes 후 결과
print("Hello")
print("안녕!")
```

이 상태가 의도한 것이 아니라면 에디터에서 직접 수정한다. 예를 들어 하나만 남기거나, 새로운 내용으로 바꿀 수 있다.

### 5-4. VS Code에서 해결 후 commit

파일을 저장한 후 Source Control 패널에서:
1. 해결된 파일 옆 `+` 버튼 클릭 (stage)
2. 커밋 메시지 입력
3. **✓ Commit** 클릭

---

## 6. conflict 예방하는 협업 습관

conflict를 완전히 없앨 수는 없지만, 빈도와 규모를 줄일 수 있다.

| 습관 | 효과 | 실천 방법 |
|------|------|----------|
| 작업 시작 전 `git pull` | 최신 상태에서 시작해서 diverge 최소화 | 하루 시작할 때, 작업 branch 만들기 전에 실행 |
| 파일을 작게 분리 | 같은 파일을 두 사람이 동시에 수정할 가능성 감소 | 파일 하나에 기능 하나 원칙 |
| 자주 commit하고 push | 변경 범위가 작아져서 conflict 규모가 작아짐 | 기능 단위로 자주 commit |
| 큰 파일 수정 전 팀원과 소통 | 같은 파일 작업 영역 조율 | Slack, GitHub issue 댓글로 사전 공유 |
| branch를 오래 살려두지 않기 | 오래된 branch일수록 conflict 가능성 높아짐 | feature branch는 작업 완료 즉시 merge |

---

## 실습

### 실습 1. 따라 하기: conflict 직접 만들어 보기

목표: 의도적으로 conflict를 발생시키고 해결하는 전 과정을 경험한다.

**준비**: `python-study` 저장소가 로컬에 clone되어 있어야 한다.

**1단계: feature branch 만들기**

```bash
cd python-study
git checkout -b feature/test-conflict
```

**2단계: feature branch에서 파일 수정**

`hello.py`를 열어 첫 번째 print 줄을 아래처럼 바꾼다.

```python
print("Hello from feature branch!")
```

저장 후 commit:

```bash
git add hello.py
git commit -m "feature: hello 메시지 변경"
```

**3단계: main branch로 돌아가서 같은 줄 수정**

```bash
git checkout main
```

`hello.py`의 같은 줄을 이번에는 다른 내용으로 바꾼다.

```python
print("Hello from main branch!")
```

저장 후 commit:

```bash
git add hello.py
git commit -m "main: hello 메시지 변경"
```

**4단계: merge 시도 → conflict 발생**

```bash
git merge feature/test-conflict
```

아래 메시지가 나타난다:

```
Auto-merging hello.py
CONFLICT (content): Merge conflict in hello.py
Automatic merge failed; fix conflicts and then commit the result.
```

"Automatic merge failed"가 보이면 conflict가 발생한 것이다. 당황하지 말고 다음 단계로 넘어간다.

**5단계: conflict 내용 확인**

```bash
cat hello.py
```

출력 결과:

```python
<<<<<<< HEAD
print("Hello from main branch!")
=======
print("Hello from feature branch!")
>>>>>>> feature/test-conflict
```

**6단계: conflict 해결**

`hello.py`를 에디터에서 열어 아래처럼 수정한다 (두 내용을 합친 버전):

```python
print("Hello from both branches!")
```

그리고 `<<<`, `===`, `>>>` 마커가 없는지 확인한다.

**7단계: 해결 후 commit**

```bash
git add hello.py
git commit -m "merge conflict 해결: greeting 통일"
```

---

### 실습 2. 따라 하기: VS Code로 conflict 해결하기

실습 1과 같은 상황을 만든 뒤 VS Code를 사용해서 해결한다.

**1단계**: 실습 1의 1~4단계를 반복해서 conflict 상태를 만든다 (branch 이름은 `feature/test-conflict-2`로 바꿈).

**2단계**: VS Code에서 `hello.py`를 연다. Source Control 패널을 보면 파일이 빨간 `C`로 표시된다.

**3단계**: 에디터 상단에 나타나는 버튼 중 **Accept Current Change**를 클릭한다.

결과: `print("Hello from main branch!")` 만 남고 마커가 사라진다.

**4단계**: 파일 저장 후 Source Control 패널에서 `+` 버튼으로 stage → 커밋 메시지 입력 → **✓ Commit**.

---

### 실습 3. 직접 해보기: 3가지 방법으로 모두 해결해 보기

같은 conflict 상황을 3번 만들어, 각각 다른 방법으로 해결해본다.

1. 첫 번째: 터미널에서 파일을 직접 편집해서 해결
2. 두 번째: VS Code의 **Accept Incoming Change** 버튼으로 해결
3. 세 번째: VS Code의 **Accept Both Changes** 후 수동으로 내용 정리

세 가지 방법 중 어떤 것이 가장 편한지 느껴본다.

---

## 자주 하는 실수

| 상황 | 어떤 일이 생기나 | 해결 방법 |
|------|----------------|----------|
| conflict 마커를 그대로 commit | `<<<<<<` 문자열이 코드에 남아 프로그램 오류 발생 | 모든 conflict 표시 제거 후 commit |
| conflict 해결 후 `git add` 빠짐 | `git status`에서 여전히 "both modified" 표시 | `git add 파일명` 후 commit |
| 상대방 코드를 무조건 덮어씀 | 상대방의 작업 내용이 손실됨 | 두 변경 내용의 의도를 이해하고 의식적으로 선택 |
| conflict 상태에서 새로운 작업 시작 | 이전 conflict가 해결 안 된 채로 복잡해짐 | `git status`로 conflict 파일 먼저 확인하고 해결 |
| 해결 후 `git commit` 안 함 | merge가 완료되지 않은 상태 유지 | `git add` → `git commit` 순서대로 실행 |

---

## 확인 체크리스트

- [ ] conflict가 발생하는 상황(같은 파일 같은 줄을 양쪽이 수정)을 설명할 수 있는가
- [ ] `<<<<<<< HEAD`, `=======`, `>>>>>>>` 표시가 각각 무엇을 나타내는지 말할 수 있는가
- [ ] 터미널에서 파일을 직접 수정해서 conflict를 해결하고 commit할 수 있는가
- [ ] VS Code의 Accept Current Change / Accept Incoming Change 버튼을 사용할 수 있는가
- [ ] 3-way merge가 무엇인지 설명할 수 있는가
- [ ] conflict 해결 후 `git add`와 `git commit`을 올바른 순서로 실행했는가

---

## 한 번 더 생각해 보기

1. conflict가 발생했을 때 상대방 코드를 무조건 내 코드로 덮으면 어떤 문제가 생길까?
2. 3-way merge에서 "공통 조상"이 중요한 이유는 무엇일까?
3. 혼자 작업할 때도 conflict가 발생할 수 있는 경우는 어떤 경우일까?
4. conflict를 예방하는 가장 효과적인 습관은 무엇이라고 생각하는가?

---

## 다음 장

다음 장에서는 issue 트래킹을 배운다. "이런 기능을 만들자", "이런 버그가 있다"를 issue로 관리하고, branch와 PR에 연결하는 방법을 익힌다.

---

## 참고 자료

- GitHub Docs: Resolving a merge conflict — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line
- VS Code merge conflict 가이드 — https://code.visualstudio.com/docs/sourcecontrol/overview#_merge-conflicts
- Git 3-way merge 설명 — https://www.atlassian.com/git/tutorials/using-branches/git-merge
