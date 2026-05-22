# Chapter 08: PR 리뷰 가이드

## 이 장에서 배우는 것

- PR 리뷰가 왜 팀 개발에서 필수인지
- Files changed 탭 읽는 법 (초록/빨간 줄의 의미, 파일 탐색, Viewed 체크)
- 줄 단위 코멘트 달기 (마우스를 올리면 나오는 `+` 버튼 위치)
- Review 제출 방법 (Comment / Approve / Request changes 차이)
- AI 코드 체크리스트와 사람 리뷰의 차이
- 좋은 코멘트 vs 나쁜 코멘트 예시
- 리뷰어와 PR 작성자의 각 역할

---

## 왜 PR 리뷰가 필요할까

코드를 혼자 쓰면 자신의 실수를 자신이 잡기 어렵다. 내가 작성한 코드는 내 머릿속 의도대로 읽힌다. 그래서 명백한 버그도 놓치는 경우가 많다.

PR 리뷰는 코드를 main에 합치기 전에 다른 시각으로 검토하는 과정이다.

리뷰가 없으면 생기는 문제:
- 실수가 발견되지 않고 바로 배포된다
- 왜 이렇게 만들었는지 기록이 남지 않는다
- 팀원이 무슨 코드가 추가됐는지 파악하지 못한다
- 나쁜 습관이 코드베이스에 쌓인다

리뷰가 있으면:
- 버그를 배포 전에 잡는다
- 코드의 의도가 문서화된다
- 팀원이 서로의 작업을 이해한다
- 더 나은 방법을 함께 찾을 수 있다

AI가 생성한 코드라도 마찬가지다. AI는 문법적으로 올바른 코드를 만들지만, 논리적 오류, 엣지 케이스 누락, 보안 문제는 사람이 확인해야 한다.

---

## 1. PR 페이지 구성

PR을 열면 여러 탭이 있다.

```
Conversation   Commits   Checks   Files changed
```

| 탭 | 내용 |
|----|------|
| **Conversation** | 전체 대화, 리뷰 코멘트, PR 설명, merge 버튼 |
| **Commits** | 이 PR에 포함된 commit 목록 |
| **Checks** | CI/CD 자동 검사 결과 (통과/실패) |
| **Files changed** | 변경된 파일의 실제 코드 diff |

리뷰할 때는 **Files changed** 탭을 주로 사용한다.

---

## 2. Files changed 탭 읽기

### 2-1. 탭 열기

PR 페이지 상단에서 **Files changed** 탭을 클릭한다.

탭 이름 옆에 숫자가 있다 (예: `Files changed 3`). 이 숫자가 변경된 파일 수다.

### 2-2. diff 읽기

각 파일은 "diff" 형식으로 표시된다. diff는 "변경 전과 후의 차이"를 보여준다.

```diff
  def add_memo(memos, text):
-     memos.append(text)
+     if not text.strip():
+         return False
+     memos.append(text.strip())
+     return True
```

| 표시 | 색상 | 의미 |
|------|------|------|
| `-` (앞에 마이너스) | 빨간색 배경 | 삭제된 줄 |
| `+` (앞에 플러스) | 초록색 배경 | 추가된 줄 |
| (기호 없음) | 흰색/회색 배경 | 변경 없는 맥락 줄 (주변 코드) |

맥락 줄은 변경된 줄의 위아래로 3~5줄 정도 보여준다. 변경이 어떤 코드 사이에 위치하는지 파악할 수 있다.

### 2-3. 파일 헤더 읽기

각 파일 맨 위에 헤더가 있다.

```
memo.py                                    Viewed  □
  +14 -3
```

- 파일 이름: `memo.py`
- `+14`: 14줄이 추가됨
- `-3`: 3줄이 삭제됨
- **Viewed 체크박스**: 이 파일을 다 확인했으면 체크

### 2-4. 왼쪽 파일 목록

Files changed 탭 왼쪽에 변경된 파일 목록이 사이드바로 나타난다. 파일 이름을 클릭하면 해당 파일의 diff로 바로 이동한다.

파일이 많을 때 유용하다.

### 2-5. diff를 통해 확인해야 할 것

Files changed를 읽을 때 다음을 확인한다.

- **삭제된 줄(빨간색)**: 기존 기능이 의도치 않게 삭제되지 않았는가
- **추가된 줄(초록색)**: 새로 추가된 로직이 올바른가
- **엣지 케이스**: 빈 값, null, 범위를 벗어난 숫자 등의 입력이 처리됐는가
- **전체 맥락**: 변경된 줄이 전체 함수에서 어떤 역할인지 이해되는가

---

## 3. 줄 단위 코멘트 달기

### 3-1. `+` 버튼 찾기

Files changed 탭에서 코멘트를 달고 싶은 줄에 마우스를 올린다. 그 줄 왼쪽 끝에 **파란색 `+` 버튼**이 나타난다.

```
  if not text.strip():    ← 마우스를 이 줄 위로 올리면
[+]     return False      ← 맨 왼쪽에 + 버튼이 나타남
```

`+` 버튼을 클릭하면 코멘트 입력창이 나타난다.

### 3-2. 여러 줄 선택해서 코멘트 달기

하나의 코멘트가 여러 줄에 걸쳐야 할 때:

1. 시작 줄의 줄 번호 왼쪽을 클릭하고 드래그해서 끝 줄까지 선택한다
2. 선택된 영역에 `+` 버튼이 나타난다

### 3-3. 코멘트 입력창 사용법

`+` 버튼을 클릭하면 입력창이 나타난다.

```
┌────────────────────────────────────────────────────────┐
│  Leave a comment                                       │
│                                                        │
│  [코멘트 입력 공간]                                     │
│                                                        │
│  [Add single comment]  [Start a review]               │
└────────────────────────────────────────────────────────┘
```

- **Add single comment**: 코멘트 하나만 즉시 제출 (상대방에게 바로 알림이 간다)
- **Start a review**: 리뷰 모드로 코멘트 작성 시작 (모든 코멘트를 모아서 한 번에 제출)

여러 곳에 코멘트를 달 예정이라면 **Start a review**를 사용한다. 코멘트가 하나씩 알림으로 가지 않고, 리뷰 완료 시 한 번에 전달된다.

### 3-4. 코멘트 예시

코멘트를 달 때 단순 지적보다 이유와 대안을 함께 제시하는 것이 좋다.

```
빈 문자열 외에 None이 들어오는 경우도 처리해야 할 것 같습니다.
text가 None이면 text.strip()에서 AttributeError가 발생합니다.

아래처럼 수정하면 어떨까요?
if not text or not text.strip():
    return False
```

### 3-5. Suggest change (수정 제안)

코멘트 입력창에서 **Suggest change** 버튼을 클릭하면 직접 수정 코드를 제안할 수 있다.

```suggestion
    if not text or not text.strip():
        return False
```

이렇게 작성하면 PR 작성자가 이 제안 아래 **Commit suggestion** 버튼 한 번으로 제안된 코드를 적용할 수 있다.

---

## 4. Review 제출하기

코멘트를 모두 달았으면 리뷰를 완료해야 한다.

### 4-1. Review changes 버튼 찾기

Files changed 탭 오른쪽 상단에 초록색 **Review changes** 버튼이 있다. 클릭한다.

팝업이 나타난다:

```
┌────────────────────────────────────────────────────────┐
│  Review changes                                        │
├────────────────────────────────────────────────────────┤
│  [전체 의견 입력창 (선택사항)]                           │
│                                                        │
│  ○ Comment                                            │
│    Submit general feedback without explicit approval   │
│                                                        │
│  ○ Approve                                            │
│    Submit feedback and approve merging these changes   │
│                                                        │
│  ○ Request changes                                    │
│    Submit feedback that must be addressed              │
│    before merging                                      │
│                                                        │
│  [Submit review]                                      │
└────────────────────────────────────────────────────────┘
```

### 4-2. 세 가지 옵션의 차이

| 옵션 | 의미 | 언제 사용하나 |
|------|------|-------------|
| **Comment** | 의견만 남기고 승인/거절은 보류 | 궁금한 점이 있거나 제안이 있지만 merge를 막고 싶지 않을 때 |
| **Approve** | "이 코드 좋습니다, merge해도 됩니다" | 코드를 검토했고 특별한 문제가 없을 때 |
| **Request changes** | "이 부분을 고쳐야 merge할 수 있습니다" | 수정이 반드시 필요한 문제를 발견했을 때 |

**Approve**를 받으면 PR 작성자가 merge할 수 있다 (저장소 설정에 따라 다름).

**Request changes**를 선택하면 PR에 빨간 X 표시가 생기고, 작성자가 수정 후 다시 리뷰를 요청해야 merge할 수 있다.

### 4-3. 전체 의견 입력 (선택사항)

팝업 상단 텍스트 입력창에 전체적인 리뷰 의견을 적을 수 있다.

```
전체적으로 잘 작성됐습니다. 한 가지만 확인 부탁드립니다.
```

개별 줄 코멘트로 다루지 않은 전체적인 인상이나 큰 그림의 피드백을 여기에 쓴다.

---

## 5. AI 코드 체크리스트와 사람 리뷰의 차이

AI(Claude Code, Copilot 등)가 작성한 코드가 포함된 PR을 리뷰할 때 추가로 확인해야 하는 항목이 있다.

### 5-1. AI 코드가 사람 코드와 다른 점

| 특성 | AI 코드 | 사람 코드 |
|------|---------|---------|
| 문법 오류 | 거의 없음 | 가끔 있음 |
| 논리 오류 | 있을 수 있음 | 있을 수 있음 |
| 엣지 케이스 | 요청하지 않으면 누락 가능 | 경험에 따라 다름 |
| 존재하지 않는 함수 사용 | 가끔 발생 (hallucination) | 드물게 발생 |
| 테스트 여부 | 실행 안 해봤을 가능성 있음 | 보통 테스트함 |

### 5-2. AI 코드 PR 기본 체크리스트

```
□ 로컬에서 실제로 실행해서 동작을 확인했는가
□ 오류 케이스가 처리됐는가 (빈 값, 잘못된 타입, None)
□ AI가 만든 코드임이 commit 메시지 또는 PR 설명에 명시됐는가
□ 코드 설명(주석, PR 설명)이 실제 동작과 일치하는가
□ 불필요한 코드나 임시 주석이 없는가
```

### 5-3. AI 특이 추가 체크

```
□ 존재하지 않는 함수나 메서드를 사용하지 않는가 (import해도 작동하는지 확인)
□ 라이브러리 버전이 현재 환경과 호환되는가
□ 보안 민감 정보(API 키, 비밀번호, 개인정보)가 코드에 하드코딩되지 않는가
□ 외부 의존성(API 호출 등)이 테스트 환경에서 올바르게 모킹됐는가
```

### 5-4. 사람만 할 수 있는 리뷰

AI 도구가 아무리 발전해도 사람이 해야 하는 부분이 있다.

- "이 기능이 사용자 입장에서 직관적인가?"
- "이 코드가 프로젝트의 전체 맥락과 맞는가?"
- "팀 컨벤션과 일관성이 있는가?"
- "이 변경이 나중에 유지보수하기 쉬운가?"

---

## 6. 좋은 리뷰 코멘트 vs 나쁜 코멘트

### 나쁜 코멘트의 패턴

```
"이거 틀렸어요"
"코드가 별로예요"
"왜 이렇게 했어요?"
"그냥 다시 짜세요"
```

왜 나쁜가:
- 무엇이 문제인지 설명하지 않는다
- 어떻게 고쳐야 하는지 모른다
- 받는 사람이 방어적이 된다

### 좋은 코멘트의 패턴

```
"빈 리스트가 들어오면 IndexError가 발생합니다.
 if not items: return [] 를 추가하면 어떨까요?"

"이 함수가 두 가지 역할(파싱 + 저장)을 하는 것 같습니다.
 분리하면 테스트하기 더 쉬울 것 같은데, 어떻게 생각하세요?"

"여기서 for 루프 대신 리스트 컴프리헨션을 쓰면
 더 Pythonic하고 읽기 쉬울 것 같습니다:
 result = [item for item in items if condition]"
```

왜 좋은가:
- 구체적인 문제와 이유를 설명한다
- 가능하면 개선 방안도 함께 제시한다
- 질문 형식으로 대화를 유도한다

### 리뷰 코멘트 작성 원칙

1. **구체적으로**: "이게 문제입니다" 대신 "N번 줄에서 X가 발생합니다"
2. **이유를 설명**: "왜 이게 문제인지" 설명
3. **대안 제시**: 가능하면 "이렇게 바꾸면 어떨까요?" 제안
4. **nit 표시**: 작은 스타일 제안은 앞에 `nit:` 붙이기 (`nit: 변수명을 더 명확하게...`)
5. **칭찬도 포함**: 좋은 코드를 보면 "이 부분 깔끔하네요" 한 마디도 OK

---

## 7. 리뷰어와 작성자의 역할

### 리뷰어 (Reviewer)

리뷰를 요청받은 사람이다.

해야 할 일:
1. PR 설명을 먼저 읽고 변경 목적을 이해한다
2. Commits 탭에서 commit 메시지로 변경 흐름을 파악한다
3. Files changed에서 각 변경사항을 확인한다
4. 가능하면 코드를 로컬에서 실행해서 동작을 확인한다
5. 명확하고 건설적인 코멘트를 남긴다
6. Approve 또는 Request changes로 리뷰를 완료한다

리뷰 시간 목표: 보통 24시간 이내. 팀 상황에 따라 다르지만 너무 오래 두면 PR이 쌓인다.

### 작성자 (Author)

PR을 만든 사람이다.

해야 할 일:
1. PR 설명에 변경 목적, 테스트 방법, 스크린샷(UI 변경 시)을 포함한다
2. 리뷰 코멘트에 빠르게 응답한다 (동의, 질문, 반론 등)
3. 수정이 필요하면 수정 후 커밋하고 다시 리뷰를 요청한다
4. 해결된 코멘트는 **Resolve conversation** 버튼을 클릭한다
5. 모든 코멘트가 해결되면 merge한다

---

## 실습

### 실습 1. 따라 하기: PR 만들고 Files changed 확인

목표: PR을 만들고 Files changed 탭에서 diff를 읽는 법을 익힌다.

1. `memo-service` 저장소에서 새 branch를 만든다:
   ```bash
   git checkout -b practice/review-test
   ```

2. `memo.py`에 아래 함수를 추가한다:
   ```python
   def add_memo(memos, text):
       memos.append(text)   # 빈 문자열 체크 없음
       return True
   ```

3. commit하고 push한다:
   ```bash
   git add memo.py
   git commit -m "feat: add_memo 함수 추가 (리뷰 실습용)"
   git push origin practice/review-test
   ```

4. GitHub에서 PR을 만든다. 제목: `리뷰 실습용 PR`

5. PR 페이지에서 **Files changed** 탭을 클릭한다.

6. 추가된 코드(초록색 줄)를 확인한다.

7. 파일 오른쪽 **Viewed** 체크박스를 체크한다.

---

### 실습 2. 따라 하기: 줄 코멘트 달기

목표: Files changed에서 특정 줄에 코멘트를 달고 Review를 제출한다.

실습 1에서 만든 PR이 열려 있는 상태에서:

1. **Files changed** 탭에서 `memos.append(text)` 줄에 마우스를 올린다.
2. 줄 왼쪽에 파란 `+` 버튼이 나타나면 클릭한다.
3. 아래 코멘트를 입력한다:
   ```
   빈 문자열이 들어오면 그대로 추가됩니다.
   text.strip()이 빈 문자열인 경우 추가하지 않도록 처리가 필요할 것 같습니다.
   ```
4. **Start a review** 버튼을 클릭한다.
5. **Review changes** 버튼 클릭 → **Comment** 선택 → **Submit review** 클릭.
6. Conversation 탭으로 이동해서 코멘트가 제출됐는지 확인한다.

---

### 실습 3. 직접 해보기: Suggest change 사용

목표: Suggest change 기능으로 수정 코드를 직접 제안한다.

1. 실습 1의 PR에서 Files changed를 연다.
2. `memos.append(text)` 줄에 `+` 버튼을 클릭한다.
3. 코멘트 입력창에서 **Suggest change** 버튼(사각형 안에 +-가 그려진 아이콘)을 클릭한다.
4. 아래처럼 제안 코드를 작성한다:
   ```suggestion
       if not text or not text.strip():
           return False
       memos.append(text.strip())
       return True
   ```
5. 코멘트와 함께 **Start a review** → **Review changes** → **Request changes** → **Submit review**.
6. PR Conversation 탭에서 리뷰 결과와 수정 제안이 표시되는지 확인한다.

---

## 자주 하는 실수

| 상황 | 어떤 일이 생기나 | 해결 방법 |
|------|----------------|----------|
| Files changed를 보지 않고 merge | 오류가 있는 코드가 main에 합쳐짐 | 반드시 Files changed 탭에서 diff 확인 |
| 줄 코멘트 대신 Add single comment 사용 | 여러 코멘트가 하나씩 알림으로 전달됨 | Start a review로 시작해서 한 번에 제출 |
| Approve를 이유 없이 클릭 | 코드 품질 보장이 안 됨 | 실제로 diff를 읽고 나서 Approve |
| 코멘트에 이유 없이 "수정해주세요" | 작성자가 어떻게 고쳐야 할지 모름 | 왜 문제인지, 어떻게 고칠지 함께 작성 |
| Request changes 후 응답 안 함 | PR이 막힌 상태로 방치됨 | 수정이 되면 re-request review 또는 직접 확인 |

---

## 확인 체크리스트

- [ ] Files changed 탭에서 초록(추가)/빨간(삭제) diff를 읽을 수 있는가
- [ ] 특정 줄에 마우스를 올려 `+` 버튼으로 코멘트를 달 수 있는가
- [ ] Start a review로 여러 코멘트를 모아서 한 번에 제출할 수 있는가
- [ ] Comment / Approve / Request changes 중 언제 어떤 것을 사용하는지 아는가
- [ ] AI 코드 PR 체크리스트의 항목들을 직접 확인할 수 있는가
- [ ] Suggest change로 수정 코드를 제안할 수 있는가

---

## 한 번 더 생각해 보기

1. 리뷰 없이 코드를 main에 merge하면 어떤 위험이 있을까?
2. AI가 생성한 코드를 리뷰할 때 사람이 직접 작성한 코드보다 더 주의해야 하는 이유는 무엇일까?
3. "Request changes"를 받은 작성자가 동의하지 않는 경우 어떻게 대응하는 것이 좋을까?
4. 혼자 작업하는 프로젝트에서도 PR을 만들어 셀프 리뷰를 하는 것이 왜 도움이 될까?

---

## 참고 자료

- GitHub Docs: Reviewing proposed changes in a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-proposed-changes-in-a-pull-request
- GitHub Docs: Commenting on a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/commenting-on-a-pull-request
- GitHub Docs: Incorporating feedback in your pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/incorporating-feedback-in-your-pull-request
