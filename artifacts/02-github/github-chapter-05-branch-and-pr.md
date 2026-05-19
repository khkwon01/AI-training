# Chapter 05: branch와 Pull Request

## 이 장에서 배우는 것

- branch가 무엇인지, 왜 쓰는지
- branch를 만들고 전환하는 방법
- branch에서 작업하고 main에 합치는 방법
- Pull Request(PR)를 만들고 검토하는 방법
- merge 후 branch를 정리하는 방법

---

## 먼저 쉬운 설명

코드를 수정할 때 원본을 바로 건드리면 위험하다.

작업 중 실수로 원본이 망가질 수 있고, 여러 사람이 동시에 작업하면 서로 충돌이 생긴다.

**branch**는 원본 코드(`main`)에서 독립된 작업 공간을 만드는 방법이다.

```
main ─────────────────────────────▶ (원본, 항상 안정적)
         │
         └─ feature/add-search ───▶ (새 기능 개발)
                  │
                  └─────── PR ───▶ main에 합치기
```

새 기능을 만들 때마다 branch를 만들고, 완성되면 PR로 검토받고 main에 합친다.

---

## 1. branch 만들기

```bash
# 현재 branch 확인
git branch

# 새 branch 만들기
git branch feature/add-search

# branch 전환
git checkout feature/add-search

# 만들면서 바로 전환 (위 두 명령 한 번에)
git checkout -b feature/add-search
```

branch 이름 규칙:
- `feature/기능이름` — 새 기능
- `fix/버그이름` — 버그 수정
- `docs/내용` — 문서 수정

---

## 2. branch에서 작업하기

branch를 전환한 상태에서 파일을 수정하면 해당 branch에만 기록된다.

```bash
# 현재 branch 확인 (앞에 * 표시가 현재 branch)
git branch
# * feature/add-search
#   main

# 파일 수정 후 commit
git add search.py
git commit -m "검색 기능 추가"

# branch를 GitHub에 push
git push origin feature/add-search
```

---

## 3. Pull Request 만들기

branch를 GitHub에 push하면 PR을 만들 수 있다.

### GitHub 웹에서 PR 만들기

1. GitHub 저장소 페이지 접속
2. 상단에 노란 배너 **"Compare & pull request"** 버튼 클릭  
   (없으면 **Pull requests** 탭 → **New pull request**)
3. 설정 확인:
   - **base**: `main` (합칠 대상)
   - **compare**: `feature/add-search` (내 작업 branch)
4. 제목과 설명 입력
5. **Create pull request** 클릭

### 좋은 PR 제목과 설명

```
제목: 메모 검색 기능 추가

## 변경 내용
- search_memo(keyword) 함수 구현
- 키워드가 포함된 메모 목록 출력

## 테스트 방법
1. 메모 3개 추가
2. search_memo("Python") 실행
3. 해당 키워드 포함 메모만 출력되는지 확인
```

---

## 4. PR 검토하기

### Files changed 탭

PR 페이지의 **Files changed** 탭에서 변경된 코드를 줄 단위로 확인할 수 있다.

- 초록색 (`+`): 추가된 줄
- 빨간색 (`-`): 삭제된 줄

### 코멘트 남기기

줄 번호 왼쪽의 `+` 버튼을 클릭하면 해당 줄에 코멘트를 남길 수 있다.

```
이 함수에서 빈 keyword가 들어오면 어떻게 처리하나요?
```

### 승인(Approve) 또는 변경 요청(Request changes)

검토가 끝나면:
- **Approve**: 코드가 문제없다고 승인
- **Request changes**: 수정이 필요하다고 요청
- **Comment**: 의견만 남기기

---

## 5. merge하기

PR이 승인되면 main에 합친다.

1. PR 페이지 아래 **Merge pull request** 클릭
2. **Confirm merge** 클릭
3. branch 삭제: **Delete branch** 클릭

### 로컬에서도 최신 상태 반영

```bash
# main으로 전환
git checkout main

# GitHub의 최신 내용 가져오기
git pull origin main
```

---

## 6. VS Code에서 branch 작업하기

Source Control 패널 하단 상태 표시줄에서 현재 branch 이름을 볼 수 있다.

```
main  ← 클릭하면 branch 목록/전환 가능
```

- 클릭 → **Create new branch** 선택 → 이름 입력
- 또는 Command Palette (`Cmd/Ctrl+Shift+P`) → `Git: Create Branch`

---

## 7. 따라 하기 실습

### 실습 1. branch 만들고 기능 추가하기

앞에서 만든 `memo-service` 저장소에서:

```bash
# 새 branch 생성
git checkout -b feature/search-memo

# search_memo.py 파일 만들기
```

```python
# search_memo.py
def search_memo(memos, keyword):
    results = [m for m in memos if keyword in m]
    if not results:
        print(f"'{keyword}' 가 포함된 메모가 없습니다.")
    else:
        for i, m in enumerate(results, 1):
            print(f"{i}. {m}")
    return results
```

```bash
git add search_memo.py
git commit -m "메모 검색 기능 추가"
git push origin feature/search-memo
```

### 실습 2. GitHub에서 PR 만들기

1. GitHub 저장소에서 **Compare & pull request** 클릭
2. 제목: `메모 검색 기능 추가`
3. 변경 내용 설명 입력
4. **Create pull request** 클릭

### 실습 3. Files changed에서 코드 확인하기

PR 페이지에서 **Files changed** 탭을 열고 변경 내용을 확인한다. 줄에 코멘트를 하나 남겨본다.

### 실습 4. merge하고 로컬 반영하기

```bash
# GitHub에서 Merge pull request 클릭 후
git checkout main
git pull origin main
```

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| main에 직접 commit | 원본 코드가 불안정해짐 | 항상 branch 만들고 작업 |
| push 전에 PR 시도 | GitHub에 branch가 없어서 PR 불가 | `git push origin branch이름` 먼저 |
| merge 후 로컬 main 미업데이트 | 로컬이 GitHub보다 뒤처짐 | merge 후 `git pull origin main` |
| branch 이름 오타 | 엉뚱한 branch로 push | `git branch` 로 현재 branch 확인 |

---

## 확인 체크리스트

- [ ] `git checkout -b 이름` 으로 branch를 만들고 전환할 수 있는가
- [ ] branch에서 commit하고 GitHub에 push할 수 있는가
- [ ] GitHub에서 PR을 만들 수 있는가
- [ ] Files changed에서 변경 내용을 확인하고 코멘트를 남길 수 있는가
- [ ] merge 후 로컬 main을 최신 상태로 업데이트할 수 있는가

---

## 한 번 더 생각해 보기

1. main에 직접 push하지 않고 branch와 PR을 쓰는 이유는 무엇인가?
2. PR 제목과 설명을 잘 쓰면 어떤 점이 좋을까?
3. 여러 사람이 동시에 다른 branch에서 작업하면 어떤 일이 생길까?

---

## 다음 장

다음 장에서는 merge conflict(충돌)가 무엇인지, 어떻게 해결하는지 배운다.

---

## 참고 자료

- GitHub Docs: About branches — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-branches
- GitHub Docs: Creating a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request
