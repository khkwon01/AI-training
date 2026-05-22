# Chapter 07: issue 트래킹

## 이 장에서 배우는 것

- issue가 무엇이고 왜 써야 하는지
- issue 만들기 (제목/본문/라벨/담당자 각 필드 설명)
- GitHub UI의 "Create a branch" 버튼으로 issue에서 직접 branch 만들기
- commit 메시지에 `Closes #번호`를 써서 PR merge 시 자동 close
- issue 검색과 필터 사용법
- issue → branch → commit → PR → 자동 close 전체 흐름 실습

---

## 왜 issue를 써야 할까

코드를 작성하다 보면 이런 상황이 생긴다.

- "검색 기능을 추가해야 한다"
- "로그인 버튼이 가끔 클릭이 안 된다"
- "README 설명이 너무 오래됐다"

이런 것들을 메모장에 적어두면:
- 파일을 잃어버릴 수 있다
- 누가 어떤 작업을 하고 있는지 모른다
- 언제 완료됐는지 추적이 안 된다
- 왜 이 코드가 추가됐는지 나중에 이해하기 어렵다

**GitHub issue**는 이런 작업 항목을 저장소와 함께 관리하는 공간이다. 각 issue는 번호가 붙고, 누가 담당하는지, 어떤 상태인지 추적할 수 있다. 코드 변경(commit, PR)과 연결되기 때문에 "이 코드는 issue #5를 해결하기 위해 추가됐다"는 기록이 자동으로 남는다.

혼자 작업할 때도 issue를 쓰면 유용하다. 나중에 "왜 이 코드를 바꿨지?"라는 질문에 바로 답할 수 있고, 할 일 목록을 GitHub 안에서 관리할 수 있다.

---

## 1. issue 만들기

### 1-1. Issue 탭 찾기

GitHub 저장소 페이지를 열면 상단에 탭이 있다.

```
Code   Issues   Pull requests   Actions   Projects   Wiki   Security   Insights   Settings
```

**Issues** 탭을 클릭한다.

### 1-2. 새 issue 만들기

Issues 탭 오른쪽에 초록색 **New issue** 버튼이 있다. 클릭한다.

새 issue 입력 화면이 열린다.

### 1-3. 각 필드 설명

#### 제목 (Title)

```
Add title
```

issue의 핵심 내용을 한 줄로 요약한다.

좋은 제목: `메모 검색 기능 추가`, `빈 문자열 입력 시 오류 발생`
나쁜 제목: `수정 필요`, `버그`, `기능`

제목만 봐도 무슨 issue인지 알 수 있어야 한다.

#### 본문 (Description)

제목 아래 큰 텍스트 입력창이다. Markdown 형식을 지원한다.

버그 신고 issue의 본문 예:

```markdown
## 문제 설명
`search_memo("")`를 실행하면 IndexError가 발생합니다.

## 재현 방법
1. 메모 3개 추가: add_memo("A"), add_memo("B"), add_memo("C")
2. search_memo("") 실행
3. IndexError: list index out of range 확인

## 기대 동작
빈 문자열로 검색하면 전체 메모 목록을 반환해야 합니다.

## 환경
- Python 3.11
- macOS 14
```

기능 추가 issue의 본문 예:

```markdown
## 목적
저장된 메모의 내용을 수정할 수 있어야 합니다.
현재는 삭제 후 다시 추가해야 하는 불편함이 있습니다.

## 구현 방법
- `edit_memo(number, new_text)` 함수 추가
- 번호로 수정할 메모를 선택
- 새 내용을 입력받아 기존 내용 교체
- 수정 전/후 내용을 출력

## 완료 조건
- [ ] edit_memo() 함수 구현
- [ ] 잘못된 번호 입력 시 오류 메시지 표시
- [ ] 정상 동작 테스트 완료
```

#### Labels (라벨)

오른쪽 패널에 **Labels** 섹션이 있다. **Labels** 텍스트 또는 톱니바퀴 아이콘을 클릭하면 라벨 목록이 나온다.

GitHub 기본 라벨:

| 라벨 | 색상 | 의미 | 언제 사용 |
|------|------|------|----------|
| `bug` | 빨간색 | 버그 수정 | 뭔가 잘못 동작할 때 |
| `enhancement` | 파란색 | 기능 추가/개선 | 새 기능이나 기존 기능 개선 |
| `documentation` | 파란색 | 문서 작업 | README, 주석, 가이드 작성 |
| `good first issue` | 초록색 | 입문자에게 적합 | 처음 기여하는 사람에게 권장할 때 |
| `help wanted` | 초록색 | 도움 요청 | 혼자 해결하기 어려울 때 |
| `question` | 보라색 | 질문 | 어떻게 할지 의논이 필요할 때 |

라벨 이름 옆 체크박스를 클릭하면 선택된다. 여러 개 선택할 수 있다.

#### Assignees (담당자)

**Assignees** 섹션의 톱니바퀴 아이콘을 클릭하면 저장소 멤버 목록이 나온다.

본인을 선택하면 "내가 이 issue를 담당한다"는 표시가 된다. 혼자 작업할 때는 항상 본인으로 설정하는 것이 좋다.

#### 그 외 옵션

- **Projects**: 칸반 보드와 연결 (GitHub Projects 사용 시)
- **Milestone**: 버전이나 기간 목표와 연결
- **Linked pull requests**: 관련 PR 연결

처음에는 Labels와 Assignees만 사용해도 충분하다.

### 1-4. Submit new issue

모든 정보를 입력했으면 오른쪽 아래 **Submit new issue** 버튼을 클릭한다.

issue가 만들어지면 자동으로 번호가 붙는다. 첫 번째 issue는 `#1`, 두 번째는 `#2`가 된다.

---

## 2. issue에서 branch 직접 만들기

GitHub UI에서 issue 페이지에 "Create a branch" 기능이 있다.

### 2-1. "Create a branch" 버튼 위치

issue 페이지를 열면 오른쪽 사이드바에 **Development** 섹션이 있다. 여기서 **Create a branch** 링크를 클릭한다.

(이 버튼이 보이지 않으면 issue 페이지 오른쪽 사이드바를 아래로 스크롤한다)

### 2-2. branch 만들기 대화창

클릭하면 작은 팝업 창이 열린다.

```
Branch name: 3-empty-search-bug    ← 자동으로 issue 번호+제목으로 채워짐
Repository destination: 내저장소명
Change branch source: main ▼       ← 어느 branch에서 분기할지
```

- **Branch name**: 기본값이 자동 생성되지만, 직접 수정할 수 있다. `fix/3-empty-search-bug` 처럼 앞에 타입을 붙이면 좋다.
- **Change branch source**: 보통 `main`에서 시작한다.

**Create branch** 버튼을 클릭한다.

### 2-3. 로컬에 branch 가져오기

GitHub에서 branch를 만들면 원격(remote)에만 존재한다. 로컬에서 작업하려면 아래 명령을 실행한다.

```bash
# 원격 branch 목록 최신화
git fetch origin

# 원격 branch를 로컬로 가져와서 전환
git checkout 브랜치이름
```

또는:

```bash
git pull
git checkout 브랜치이름
```

### 2-4. 터미널에서 직접 branch 만들기 (기존 방식)

GitHub UI를 사용하지 않고 터미널에서 만들 수도 있다. issue 번호를 이름에 포함시키는 것이 관행이다.

```bash
# issue #3을 처리하는 branch
git checkout -b fix/3-empty-search-bug

# issue #5를 처리하는 feature branch
git checkout -b feature/5-edit-memo
```

branch 이름 앞에 타입 접두사를 붙이는 것이 좋다.

| 접두사 | 의미 | 예시 |
|--------|------|------|
| `feature/` | 새 기능 추가 | `feature/5-edit-memo` |
| `fix/` | 버그 수정 | `fix/3-empty-search-bug` |
| `docs/` | 문서 수정 | `docs/7-update-readme` |
| `refactor/` | 코드 구조 개선 | `refactor/10-split-functions` |

---

## 3. commit 메시지에 `Closes #번호` 쓰기

commit 메시지나 PR 설명에 특정 키워드와 함께 issue 번호를 쓰면, PR이 merge될 때 issue가 자동으로 닫힌다.

### 3-1. 사용할 수 있는 키워드

| 키워드 | 예시 |
|--------|------|
| `Closes` | `Closes #3` |
| `Fixes` | `Fixes #3` |
| `Resolves` | `Resolves #3` |

키워드는 대소문자를 가리지 않는다. `closes #3`, `CLOSES #3` 모두 동작한다.

### 3-2. commit 메시지에 포함하기

```bash
git commit -m "fix: 빈 문자열 검색 오류 수정 (#3)

search_memo()에 빈 문자열이 입력되면 전체 목록을 반환하도록 수정.
빈 문자열 체크 조건 추가.

Closes #3"
```

단, commit 메시지의 `Closes #번호`는 해당 commit이 **default branch(main)**에 merge될 때만 issue가 닫힌다.

### 3-3. PR 설명에 포함하기 (더 일반적)

commit 메시지보다 PR 설명에 `Closes #번호`를 쓰는 것이 더 일반적이다.

PR을 만들 때 설명란에:

```markdown
## 변경 내용
search_memo() 함수에서 빈 문자열 입력 시 IndexError가 발생하는 문제를 수정했습니다.

## 수정 방법
- 빈 문자열 체크 조건 추가: `if not keyword.strip(): return memos`
- 빈 문자열이면 전체 메모 목록 반환

## 테스트
- search_memo("") → 전체 메모 반환 확인
- search_memo("Python") → 키워드 포함 메모만 반환 확인

Closes #3
```

PR이 merge되면 issue #3이 자동으로 "Closed" 상태가 된다.

### 3-4. 여러 issue를 동시에 close

```
Closes #3, Closes #7, Closes #12
```

또는

```
Fixes #3
Fixes #7
```

한 PR로 여러 issue를 해결할 때 사용한다.

---

## 4. issue 검색과 필터

issue가 많아지면 원하는 것을 찾기 어려워진다. 검색과 필터 기능을 활용한다.

### 4-1. Issues 탭의 기본 필터

Issues 탭에 들어가면 기본적으로 `is:issue is:open` 필터가 적용되어 있다. 열린 issue만 보여준다.

상단의 **Open** / **Closed** 탭으로 전환할 수 있다.

### 4-2. 라벨로 필터

Issues 탭 상단의 **Labels** 드롭다운을 클릭한다. 원하는 라벨을 선택하면 해당 라벨이 붙은 issue만 표시된다.

예: `bug` 라벨만 보기 → Labels 클릭 → `bug` 선택

### 4-3. 담당자로 필터

**Assignee** 드롭다운 → 사람 이름 선택

내가 담당한 issue만 보려면: Assignee → 내 이름 선택

### 4-4. 검색으로 찾기

Issues 탭 검색창에 키워드를 입력하면 제목과 본문에서 검색한다.

```
is:open label:bug
is:closed assignee:내이름
is:open no:assignee
검색어
```

| 검색어 | 의미 |
|--------|------|
| `is:open` | 열린 issue |
| `is:closed` | 닫힌 issue |
| `label:bug` | bug 라벨 |
| `assignee:사용자명` | 특정 사람이 담당 |
| `no:assignee` | 담당자 없음 |
| `검색어` | 제목/본문에 해당 단어 포함 |

---

## 5. issue → branch → commit → PR → close 전체 흐름

```
1. issue 생성
   GitHub → Issues → New issue
   제목: "검색 기능 추가", 라벨: enhancement
   → issue #3 생성됨
          ↓
2. branch 생성
   GitHub issue 페이지 → Create a branch
   또는 터미널: git checkout -b feature/3-search-memo
          ↓
3. 코드 수정 및 commit
   def search_memo(keyword): ...
   git add memo.py
   git commit -m "feature: 메모 검색 기능 추가 (#3)"
          ↓
4. push
   git push origin feature/3-search-memo
          ↓
5. PR 생성
   GitHub에서 PR 만들기
   설명에: "Closes #3" 포함
          ↓
6. PR review & merge
   merge 완료
          ↓
7. issue #3 자동 close
   Issues 탭에서 확인
```

---

## 실습

### 실습 1. 따라 하기: 기능 issue 만들기

목표: `memo-service` 저장소에 기능 추가 issue를 만든다.

1. GitHub에서 `memo-service` 저장소를 연다
2. **Issues** 탭 클릭 → **New issue** 클릭
3. 아래 내용을 입력한다:

**제목**:
```
메모 검색 기능 추가
```

**본문**:
```markdown
## 목적
키워드로 메모를 검색하는 기능이 필요합니다.
현재는 전체 목록을 보고 직접 찾아야 합니다.

## 구현 방법
- `search_memo(keyword)` 함수 추가
- keyword가 포함된 메모만 반환
- 빈 문자열 입력 시 전체 목록 반환

## 완료 조건
- [ ] search_memo() 함수 구현
- [ ] 키워드가 없는 경우 처리
- [ ] 동작 테스트 완료
```

4. **Labels** → `enhancement` 선택
5. **Assignees** → 본인 선택
6. **Submit new issue** 클릭
7. issue 번호(예: `#1`)가 할당됐는지 확인한다

---

### 실습 2. 따라 하기: issue에서 branch 만들기

목표: 방금 만든 issue에서 직접 branch를 만들고 로컬에 가져온다.

1. 방금 만든 issue 페이지를 연다
2. 오른쪽 사이드바에서 **Development** → **Create a branch** 클릭
3. Branch name을 `feature/1-search-memo`로 수정한다 (issue 번호가 1인 경우)
4. **Create branch** 클릭
5. 터미널에서 아래를 실행한다:
   ```bash
   cd memo-service
   git fetch origin
   git checkout feature/1-search-memo
   ```
6. `git branch`를 실행해서 현재 branch가 `feature/1-search-memo`인지 확인한다:
   ```bash
   git branch
   # * feature/1-search-memo
   #   main
   ```

---

### 실습 3. 직접 해보기: commit으로 issue 자동 close

목표: 코드를 추가하고 PR을 통해 issue가 자동으로 닫히는 것을 확인한다.

1. `feature/1-search-memo` branch에서 `memo.py`에 아래 함수를 추가한다:

```python
def search_memo(keyword):
    if not keyword.strip():
        return memos[:]  # 빈 문자열이면 전체 반환
    result = [m for m in memos if keyword in m]
    return result
```

2. 저장 후 commit한다:
   ```bash
   git add memo.py
   git commit -m "feature: 메모 검색 기능 추가

   keyword로 메모를 검색하는 search_memo() 함수 추가.
   빈 문자열 입력 시 전체 목록 반환.

   Closes #1"
   ```

3. push한다:
   ```bash
   git push origin feature/1-search-memo
   ```

4. GitHub에서 PR을 만든다. 설명란에 `Closes #1`이 포함되어 있는지 확인한다.

5. PR을 merge한다 (본인의 PR이면 본인이 merge해도 된다).

6. Issues 탭을 열어서 issue #1이 자동으로 **Closed** 상태가 됐는지 확인한다.

---

## 자주 하는 실수

| 상황 | 어떤 일이 생기나 | 해결 방법 |
|------|----------------|----------|
| issue 없이 바로 코딩 시작 | 왜 이 코드가 추가됐는지 나중에 알기 어려움 | 작업 전 issue 먼저 생성하는 습관 만들기 |
| PR에 issue 번호 미포함 | merge 후 issue를 수동으로 직접 닫아야 함 | PR 설명에 반드시 `Closes #번호` 추가 |
| 잘못된 issue 번호 입력 | 엉뚱한 issue가 닫히거나 아무것도 안 닫힘 | PR 만들 때 issue 번호 한 번 더 확인 |
| issue 제목이 모호함 | "fix something" 같은 제목으로 나중에 찾기 어려움 | "무엇을, 어떻게" 형식으로 구체적으로 작성 |
| branch 없이 main에서 바로 작업 | main에 직접 push되어 PR 흐름이 깨짐 | issue에서 Create a branch 또는 checkout -b 사용 |

---

## 확인 체크리스트

- [ ] issue를 만들고 라벨과 담당자를 설정할 수 있는가
- [ ] issue 페이지에서 "Create a branch" 버튼으로 branch를 만들 수 있는가
- [ ] 터미널에서 issue 번호를 포함한 branch 이름으로 만들 수 있는가
- [ ] PR 설명에 `Closes #번호`를 포함해서 만들 수 있는가
- [ ] PR merge 후 issue가 자동으로 닫히는 것을 확인했는가
- [ ] Labels 필터로 특정 라벨의 issue만 볼 수 있는가

---

## 한 번 더 생각해 보기

1. issue를 먼저 만들고 코딩하는 것이 왜 좋을까? 코드와 issue가 연결되면 어떤 정보를 나중에 얻을 수 있을까?
2. 혼자 작업할 때도 issue를 쓰면 어떤 장점이 있을까?
3. PR에 여러 issue를 연결하는 것이 좋을까, 하나씩 따로 처리하는 것이 좋을까?
4. `Closes`와 `Fixes` 키워드는 동일하게 동작한다. 어떤 상황에 어떤 키워드를 쓰는 것이 더 자연스러울까?

---

## 참고 자료

- GitHub Docs: About issues — https://docs.github.com/en/issues/tracking-your-work-with-issues/about-issues
- GitHub Docs: Linking a pull request to an issue — https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue
- GitHub Docs: Creating a branch from an issue — https://docs.github.com/en/issues/tracking-your-work-with-issues/creating-a-branch-for-an-issue
