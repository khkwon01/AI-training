# Chapter 07: issue 트래킹

## 이 장에서 배우는 것

- issue가 무엇인지, 언제 쓰는지
- issue를 만들고 label과 assignee를 설정하는 방법
- issue를 branch와 PR에 연결하는 방법
- issue를 PR merge 시 자동으로 닫는 방법
- issue를 활용한 작업 흐름 (issue → branch → PR → close)

---

## 먼저 쉬운 설명

코드를 수정하다 보면 이런 상황이 생긴다.

- "검색 기능을 추가해야 한다"
- "로그인 버튼이 가끔 안 눌린다"
- "README가 너무 오래됐다"

이런 것들을 메모장에 적어두면 잃어버리기 쉽고, 누가 뭘 하고 있는지 알 수 없다.

**issue**는 GitHub에서 이런 작업 항목을 관리하는 공간이다. 각 issue는 번호가 붙고, 누가 담당하는지, 어떤 상태인지 추적할 수 있다.

---

## 1. issue 만들기

1. GitHub 저장소 페이지 → **Issues** 탭
2. **New issue** 버튼 클릭
3. 제목과 내용 입력
4. 오른쪽 패널에서 설정:
   - **Assignees**: 담당자 지정 (나 자신 선택 가능)
   - **Labels**: 종류 표시 (bug, enhancement, documentation 등)
5. **Submit new issue** 클릭

issue가 만들어지면 번호가 붙는다. 예: `#1`, `#2`

---

## 2. issue 제목과 내용 잘 쓰기

### 버그 신고 issue

```
제목: 검색어가 빈 문자열일 때 오류 발생

## 문제
search_memo("") 를 실행하면 IndexError가 발생합니다.

## 재현 방법
1. 메모 3개 추가
2. search_memo("") 실행
3. IndexError 확인

## 기대 동작
빈 문자열이면 전체 메모 목록을 반환해야 합니다.

## 환경
Python 3.11, macOS
```

### 기능 추가 issue

```
제목: 메모 수정 기능 추가

## 목적
저장된 메모의 내용을 수정할 수 있어야 합니다.

## 상세
- 번호로 수정할 메모를 선택
- 새 내용을 입력
- 기존 내용을 교체

## 관련 issue
없음
```

---

## 3. issue와 branch 연결하기

issue를 처리하는 작업은 전용 branch를 만드는 것이 좋다.

branch 이름에 issue 번호를 포함하는 것이 관행이다.

```bash
# issue #3을 처리하는 branch
git checkout -b fix/3-empty-search-bug

# 또는
git checkout -b feature/5-edit-memo
```

---

## 4. PR과 issue 연결하고 자동 닫기

PR 설명에 아래 키워드를 쓰면 PR이 merge될 때 issue가 자동으로 닫힌다.

| 키워드 | 예시 |
|--------|------|
| `Closes` | `Closes #3` |
| `Fixes` | `Fixes #3` |
| `Resolves` | `Resolves #3` |

PR 설명 예:

```
## 변경 내용
빈 문자열 입력 시 전체 메모를 반환하도록 수정

## 테스트
- search_memo("") → 전체 메모 반환 확인
- search_memo("Python") → 키워드 포함 메모만 반환 확인

Closes #3
```

PR이 merge되면 issue #3이 자동으로 "Closed" 상태가 된다.

---

## 5. issue → branch → PR → close 전체 흐름

```
1. issue 생성 (#3: 빈 문자열 검색 버그)
        ↓
2. branch 생성
   git checkout -b fix/3-empty-search-bug
        ↓
3. 코드 수정 및 commit
   git commit -m "fix: 빈 문자열 검색 오류 수정 (#3)"
        ↓
4. push
   git push origin fix/3-empty-search-bug
        ↓
5. PR 생성 (설명에 "Closes #3" 포함)
        ↓
6. PR merge → issue #3 자동 close
```

---

## 6. Labels 활용하기

GitHub 기본 label:

| Label | 의미 |
|-------|------|
| `bug` | 버그 수정 |
| `enhancement` | 기능 추가/개선 |
| `documentation` | 문서 작업 |
| `good first issue` | 입문자에게 적합한 작업 |
| `help wanted` | 도움 요청 |

label을 일관되게 쓰면 issue 목록을 필터링해서 보기 쉽다.

---

## 7. 따라 하기 실습

### 실습 1. issue 만들기

`memo-service` 저장소에서 아래 두 가지 issue를 만든다.

**Issue 1:**
```
제목: 메모 수정 기능 추가
Label: enhancement
내용: 번호로 기존 메모를 수정하는 edit_memo(number, new_text) 함수 필요
```

**Issue 2:**
```
제목: 빈 메모 추가 시 경고 메시지 표시 안 됨
Label: bug
내용: add_memo("") 실행 시 아무 메시지 없이 넘어감
```

### 실습 2. issue branch 만들고 PR 연결하기

1. issue #1 처리용 branch 만들기

```bash
git checkout -b feature/1-edit-memo
```

2. `edit_memo` 함수 추가

```python
def edit_memo(number, new_text):
    if 1 <= number <= len(memos):
        old = memos[number - 1]
        memos[number - 1] = new_text.strip()
        print(f"✓ 수정됨: '{old}' → '{new_text}'")
    else:
        print("잘못된 번호입니다.")
```

3. commit하고 PR 만들기

```bash
git add memo.py
git commit -m "feature: 메모 수정 기능 추가 (#1)"
git push origin feature/1-edit-memo
```

PR 설명에 `Closes #1` 포함.

4. merge 후 issue #1이 자동으로 닫히는지 확인

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| issue 없이 바로 코딩 | 왜 이 코드가 있는지 나중에 알기 어려움 | 작업 전 issue 먼저 생성 |
| PR에 issue 번호 미포함 | merge 후 issue 수동으로 닫아야 함 | PR 설명에 `Closes #번호` 추가 |
| issue 제목이 너무 모호함 | "fix something" 같은 제목 | 구체적으로 무엇을 어떻게 할지 명시 |

---

## 확인 체크리스트

- [ ] issue를 만들고 label과 assignee를 설정할 수 있는가
- [ ] issue 번호를 포함한 branch를 만들 수 있는가
- [ ] PR 설명에 `Closes #번호`를 써서 자동으로 issue를 닫을 수 있는가
- [ ] issue → branch → PR → close 전체 흐름을 직접 실습했는가

---

## 한 번 더 생각해 보기

1. issue를 먼저 만들고 코딩하는 것이 왜 좋을까?
2. 혼자 작업할 때도 issue를 쓰면 어떤 장점이 있을까?
3. PR에 여러 issue를 연결하는 것이 좋을까, 하나씩 따로 하는 것이 좋을까?

---

## 참고 자료

- GitHub Docs: About issues — https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues
- GitHub Docs: Linking a pull request to an issue — https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue
