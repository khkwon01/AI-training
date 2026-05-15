# ai-train GitHub Chapter 06: merge conflict 이해와 해결

## 이 장에서 배우는 것

- merge conflict가 무엇인지, 왜 발생하는지
- conflict 표시를 읽고 해석하는 방법
- conflict를 직접 해결하는 방법
- VS Code에서 conflict를 시각적으로 해결하는 방법
- conflict를 예방하는 협업 습관

---

## 먼저 쉬운 설명

두 사람이 같은 파일의 같은 줄을 동시에 수정하면 어떻게 될까?

예를 들어, `hello.py`에서:
- 사람 A: `print("안녕하세요")` → `print("Hello")` 로 수정
- 사람 B: `print("안녕하세요")` → `print("안녕!")` 로 수정

둘 다 push하면 Git은 어느 쪽을 선택해야 할지 모른다. 이것이 **merge conflict**다.

conflict는 무서운 것이 아니다. Git이 "여기 두 가지 버전이 있는데, 어느 걸 쓸지 사람이 결정해줘"라고 알려주는 것이다.

---

## 1. conflict가 발생하는 상황

```
main:     A --- B --- C
                 \
feature:          D --- E
```

- `main`에서 `B`와 `C`가 `hello.py` 1번 줄을 수정
- `feature`에서 `D`와 `E`도 `hello.py` 1번 줄을 수정
- `feature`를 `main`에 merge하려 할 때 conflict 발생

---

## 2. conflict 표시 읽기

conflict가 발생한 파일을 열면 아래처럼 표시된다.

```python
<<<<<<< HEAD
print("Hello")
=======
print("안녕!")
>>>>>>> feature/update-greeting
```

각 부분의 의미:

| 표시 | 의미 |
|------|------|
| `<<<<<<< HEAD` | 현재 branch(main)의 내용 시작 |
| `=======` | 두 버전의 구분선 |
| `>>>>>>> feature/...` | 합치려는 branch의 내용 끝 |

---

## 3. conflict 해결하기

conflict를 해결하는 방법은 3가지다.

**방법 1: 현재 branch(HEAD) 내용 선택**
```python
print("Hello")
```

**방법 2: 합치는 branch 내용 선택**
```python
print("안녕!")
```

**방법 3: 두 내용을 합쳐서 새로 작성**
```python
print("Hello, 안녕!")
```

선택 후 `<<<<<<`, `=======`, `>>>>>>>`  표시를 모두 지운다. 그리고 저장한다.

---

## 4. conflict 해결 후 commit

```bash
# conflict 해결된 파일을 add
git add hello.py

# conflict 해결 commit
git commit -m "merge conflict 해결: greeting 통일"
```

---

## 5. VS Code에서 시각적으로 해결하기

VS Code는 conflict를 색으로 표시하고 버튼으로 선택할 수 있게 해준다.

conflict가 있는 파일을 열면 각 conflict 위에 버튼이 나타난다:

| 버튼 | 의미 |
|------|------|
| **Accept Current Change** | HEAD(현재 branch) 내용 선택 |
| **Accept Incoming Change** | 합쳐지는 branch 내용 선택 |
| **Accept Both Changes** | 두 내용 모두 유지 |
| **Compare Changes** | 차이를 나란히 비교 |

버튼을 클릭하면 자동으로 `<<<`, `===`, `>>>` 표시가 제거된다.

---

## 6. conflict 예방하는 습관

conflict를 완전히 없앨 수는 없지만 줄일 수 있다.

| 습관 | 효과 |
|------|------|
| 작업 전 `git pull`로 최신 상태 유지 | 오래된 코드로 작업하는 것 방지 |
| 파일을 작게 분리 | 같은 파일을 두 사람이 수정할 가능성 감소 |
| 자주 commit하고 push | 변경 범위를 작게 유지 |
| 큰 파일 수정 전 팀원과 소통 | 충돌 영역 사전 인지 |

---

## 7. 따라 하기 실습

### 실습 1. conflict 직접 만들어 보기

1. `python-study` 저장소에서 새 branch 만들기

```bash
git checkout -b feature/test-conflict
```

2. `hello.py` 수정 (feature branch)

```python
print("Hello from feature branch!")
```

3. commit

```bash
git add hello.py
git commit -m "feature: hello 메시지 변경"
```

4. main으로 돌아가서 같은 줄 수정

```bash
git checkout main
```

`hello.py`를:
```python
print("Hello from main branch!")
```

5. commit

```bash
git add hello.py
git commit -m "main: hello 메시지 변경"
```

6. feature branch를 main에 merge → conflict 발생

```bash
git merge feature/test-conflict
```

```
Auto-merging hello.py
CONFLICT (content): Merge conflict in hello.py
Automatic merge failed; fix conflicts and then commit the result.
```

7. `hello.py`를 열어서 conflict 표시 확인 후 해결

8. 해결 후 commit

```bash
git add hello.py
git commit -m "merge conflict 해결"
```

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| conflict 표시를 그대로 commit | `<<<<<<` 문자열이 코드에 남음 | 모든 conflict 표시 제거 후 commit |
| conflict 해결 후 `git add` 빠짐 | commit 시 "nothing to commit" 오류 | `git add <파일>` 후 commit |
| 상대방 코드를 무조건 덮어씀 | 작업 내용 손실 | 두 변경 내용을 모두 확인하고 의도적으로 선택 |
| conflict 상태에서 새 commit | 이전 conflict가 미해결인 채 진행 | `git status`로 conflict 파일 먼저 확인 |

---

## 확인 체크리스트

- [ ] conflict가 발생하는 상황을 설명할 수 있는가
- [ ] `<<<<<<`, `=======`, `>>>>>>>`  표시의 의미를 말할 수 있는가
- [ ] conflict를 해결하고 commit할 수 있는가
- [ ] VS Code의 conflict 해결 버튼을 사용할 수 있는가

---

## 한 번 더 생각해 보기

1. conflict가 발생했을 때 상대방 코드를 무조건 내 코드로 덮으면 어떤 문제가 생길까?
2. conflict를 예방하려면 어떤 협업 습관이 필요할까?
3. 혼자 작업할 때도 conflict가 발생할 수 있는 경우가 있을까?

---

## 다음 장

다음 장에서는 issue 트래킹을 배운다. "이런 기능을 만들자", "이런 버그가 있다"를 issue로 관리하고, branch와 PR에 연결하는 방법을 익힌다.

---

## 참고 자료

- GitHub Docs: Resolving a merge conflict — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/addressing-merge-conflicts/resolving-a-merge-conflict-using-the-command-line
- VS Code merge conflict 가이드 — https://code.visualstudio.com/docs/sourcecontrol/overview#_merge-conflicts
