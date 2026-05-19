## 이 장에서 배우는 것

- GitHub 웹 화면에서 자주 보이는 핵심 라벨의 위치와 의미를 파악한다.
- `Issues` 탭과 `New issue` 버튼이 어디에 있는지 설명할 수 있다.
- branch 선택 메뉴를 찾아 현재 브랜치를 확인할 수 있다.
- `Compare & pull request` 버튼이 언제 나타나는지 이해한다.

---

## 먼저 쉬운 설명

GitHub 웹 화면을 처음 열면 버튼과 탭이 너무 많아서 어디를 봐야 할지 막막하다. 하지만 실제로 자주 쓰는 라벨은 네다섯 개뿐이다. 이 cue note는 그 핵심 라벨만 빠르게 기억할 수 있도록 돕는 **빠른 참고 카드**다. 처음부터 전부 외울 필요 없다. 이 페이지를 옆에 펼쳐 두고, 화면을 보면서 하나씩 찾아보면 된다.

---

## 1. 저장소 메인 화면 — 탭 구조

GitHub 저장소(`github.com/사용자명/저장소명`)에 들어가면 상단에 탭이 나열된다.

```
[ Code ]  [ Issues ]  [ Pull requests ]  [ Actions ]  [ Settings ]
```

| 라벨 | 위치 | 하는 일 |
|------|------|---------|
| `Code` | 첫 번째 탭 | 파일 목록 보기 (기본 화면) |
| `Issues` | 두 번째 탭 | 할 일·버그·질문 목록 |
| `Pull requests` | 세 번째 탭 | 코드 합치기 요청 목록 |

> **기억법:** "코드 → 이슈 → PR" 순서로 왼쪽부터 읽으면 된다.

---

## 2. Issues 화면 — `New issue` 버튼 위치

`Issues` 탭을 클릭하면 이슈 목록 화면이 열린다.

```
Issues 목록 화면
┌─────────────────────────────────────────────┐
│  Q 검색창            [ New issue ]  ← 오른쪽 상단 │
│  ─────────────────────────────────────────── │
│  ● 할 일: README 작성        #1  open         │
│  ● 버그: 로그인 오류          #2  open         │
└─────────────────────────────────────────────┘
```

- `New issue` 버튼은 **오른쪽 상단 초록색 버튼**이다.
- 클릭하면 제목과 내용을 입력하는 폼이 나타난다.

```
제목 입력란: [README 파일이 없어요              ]
내용 입력란: [어떤 프로젝트인지 설명이 필요합니다.]

                                  [ Submit new issue ]
```

---

## 3. branch 선택 메뉴 위치

`Code` 탭(기본 화면)으로 돌아오면 파일 목록 바로 위에 branch 메뉴가 있다.

```
Code 탭 화면
┌─────────────────────────────────────────────┐
│  [ main ▼ ]   브랜치 선택 메뉴 ← 왼쪽        │
│  ─────────────────────────────────────────── │
│  📄 README.md                                │
│  📄 index.html                               │
└─────────────────────────────────────────────┘
```

- `main ▼` 버튼을 클릭하면 브랜치 목록이 나타난다.
- 다른 브랜치 이름을 클릭하면 그 브랜치의 파일로 화면이 바뀐다.

```
브랜치 드롭다운
┌───────────────────┐
│  🔍 브랜치 검색   │
│  ─────────────── │
│  ✔ main           │  ← 현재 브랜치 (체크 표시)
│    feature/login  │
│    fix/typo       │
└───────────────────┘
```

---

## 4. `Compare & pull request` 버튼 — 언제 나타나는가

새 브랜치를 만들고 커밋을 push하면, 저장소 메인 화면 상단에 노란색 안내 배너가 나타난다.

```
┌─────────────────────────────────────────────────────────────┐
│  feature/login had recent pushes  [ Compare & pull request ] │
└─────────────────────────────────────────────────────────────┘
```

- 이 버튼은 **push 직후 잠깐만** 보인다. 안 보이면 `Pull requests` 탭 → `New pull request`로 직접 만들 수 있다.
- 버튼을 클릭하면 어떤 브랜치를 어디에 합칠지 선택하는 화면이 나온다.

```
base: [ main ▼ ]   ←   compare: [ feature/login ▼ ]

제목: [Login 기능 추가                          ]
내용: [로그인 폼과 유효성 검사를 구현했습니다.  ]

                              [ Create pull request ]
```

---

## 따라 하기 실습

### 실습 1 — Issues 탭 찾고 이슈 하나 만들기

1. 자신의 GitHub 저장소(`github.com/내아이디/my-first-repo`)에 접속한다.
2. 상단 탭에서 **`Issues`** 를 클릭한다.
3. 오른쪽 상단 초록 버튼 **`New issue`** 를 클릭한다.
4. 제목에 `README에 프로젝트 설명 추가하기` 를 입력하고 **`Submit new issue`** 를 클릭한다.
5. 이슈 번호(`#1`)가 생성되었는지 확인한다.

### 실습 2 — branch 메뉴에서 새 브랜치 확인하기

1. `Code` 탭을 클릭해 기본 화면으로 돌아온다.
2. 왼쪽 상단 **`main ▼`** 버튼을 클릭한다.
3. 검색창에 `feature/add-readme` 를 입력하고 **`Create branch: feature/add-readme`** 를 클릭한다.
4. 버튼 라벨이 `feature/add-readme ▼` 로 바뀐 것을 확인한다.

### 실습 3 — `Compare & pull request` 흐름 확인하기

1. 로컬에서 `feature/add-readme` 브랜치에 커밋을 만들고 push한다.
   ```bash
   git checkout feature/add-readme
   echo "# My Project" >> README.md
   git add README.md
   git commit -m "docs: README 초안 추가"
   git push origin feature/add-readme
   ```
2. GitHub 저장소 메인 화면을 새로 고침한다.
3. 상단 노란색 배너의 **`Compare & pull request`** 버튼을 클릭한다.
4. base가 `main`, compare가 `feature/add-readme` 인지 확인하고 **`Create pull request`** 를 클릭한다.

---

## 자주 하는 실수

| 실수 | 증상 / 오류 메시지 | 해결 방법 |
|------|-------------------|-----------|
| `Issues` 탭이 안 보인다 | 탭 목록에 `Issues`가 없음 | Settings → Features → Issues 체크 박스 활성화 |
| `Compare & pull request` 버튼이 사라졌다 | 배너가 더 이상 표시되지 않음 | `Pull requests` 탭 → `New pull request` 직접 클릭 |
| branch 드롭다운에 내 브랜치가 없다 | 목록에서 찾을 수 없음 | `git push origin 브랜치명` 을 먼저 실행했는지 확인 |
| `Create pull request` 대신 회색 버튼만 보인다 | 버튼이 비활성화됨 | base와 compare 브랜치가 동일하지 않은지 확인 |
| 이슈를 만들었는데 `Closed` 상태로 보인다 | 목록 필터가 `Closed` 로 설정됨 | 화면 상단 `Open` 필터 탭을 클릭 |

---

## 확인 체크리스트

- [ ] GitHub 저장소 메인 화면에서 `Issues` 탭을 3초 안에 찾을 수 있다.
- [ ] `New issue` 버튼이 이슈 목록 화면 오른쪽 상단에 있다는 것을 안다.
- [ ] `Code` 탭에서 branch 선택 메뉴(`main ▼`)의 위치를 설명할 수 있다.
- [ ] branch 드롭다운에서 다른 브랜치로 전환할 수 있다.
- [ ] `Compare & pull request` 버튼이 **push 직후** 나타난다는 것을 안다.
- [ ] 버튼이 사라졌을 때 `Pull requests` 탭으로 대신 PR을 만들 수 있다.

---

## 한 번 더 생각해 보기

1. `Issues` 탭과 `Pull requests` 탭은 어떻게 다를까? 각각 어떤 상황에서 먼저 열어볼까?
2. branch 드롭다운에서 브랜치를 바꾸면 파일 목록이 달라지는 이유는 무엇일까?
3. `Compare & pull request` 버튼이 사라진 뒤에도 PR을 만들 수 있다. 그 방법을 지금 바로 떠올릴 수 있는가?

---

## 다음 장

다음 장에서는 Pull Request 화면 안에서 코드 리뷰 댓글을 남기고 Merge 버튼을 누르는 전체 흐름을 배운다.