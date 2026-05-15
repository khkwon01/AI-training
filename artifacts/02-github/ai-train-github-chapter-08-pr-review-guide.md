# ai-train GitHub Chapter 08: PR 리뷰 가이드

## 이 장에서 배우는 것

- PR 리뷰가 왜 중요한지
- Files changed 탭에서 코드 변경사항을 읽는 방법
- 줄 단위로 코멘트를 남기는 방법
- AI-assisted 코드 변경에 대한 리뷰 체크리스트
- 리뷰어와 작성자가 각각 해야 할 일

---

## 먼저 쉬운 설명

코드를 main에 합치기 전에 다른 사람(또는 AI)이 먼저 보는 것이 **PR 리뷰**다.

리뷰 없이 코드를 바로 합치면:
- 실수가 발견되지 않고 배포된다
- 왜 이렇게 만들었는지 기록이 남지 않는다
- 팀원이 무슨 코드가 추가됐는지 모른다

리뷰는 코드를 틀렸다고 지적하는 것이 아니라, **더 좋은 코드로 함께 만들어가는 과정**이다.

---

## 1. Files changed 탭 읽기

PR 페이지에서 **Files changed** 탭을 클릭하면 변경된 모든 파일이 표시된다.

### diff 읽기

```diff
  def add_memo(memos, text):
-     memos.append(text)           ← 삭제된 줄 (빨간색)
+     if not text.strip():         ← 추가된 줄 (초록색)
+         return False
+     memos.append(text.strip())
+     return True
```

- **빨간색 `-`**: 삭제된 줄
- **초록색 `+`**: 추가된 줄
- **흰색**: 변경 없는 맥락 줄

### 파일 탐색

변경된 파일이 여러 개이면 왼쪽 사이드바에서 파일 목록이 보인다. 파일 이름을 클릭하면 해당 파일로 바로 이동한다.

### Viewed 체크

파일을 다 봤으면 오른쪽 **Viewed** 체크박스를 클릭하면 확인 완료 표시가 된다.

---

## 2. 줄 단위 코멘트 남기기

코멘트를 남기고 싶은 줄의 왼쪽 **+** 아이콘을 클릭한다.

```
+ if not text.strip():    ← 이 줄 왼쪽 + 클릭
```

코멘트 입력창이 열리면:

```
빈 문자열 외에 None이 들어오면 어떻게 될까요?
text가 None이면 text.strip()에서 AttributeError가 발생할 것 같습니다.
```

### 코멘트 유형

- **Comment**: 단순 의견이나 질문
- **Suggest change**: 수정 제안 (코드 블록으로 직접 제안)
- **Resolve**: 코멘트가 해결됐으면 표시

### 수정 제안 남기기

```suggestion
    if not text or not text.strip():
        return False
```

**Suggest change** 버튼을 누르면 작성자가 한 클릭으로 제안을 반영할 수 있다.

---

## 3. 리뷰 완료하기

코멘트를 모두 남겼으면 **Review changes** 버튼을 클릭한다.

| 옵션 | 의미 |
|------|------|
| **Comment** | 의견만 남기고 승인/거절은 보류 |
| **Approve** | 코드가 좋다고 승인 (Merge 가능) |
| **Request changes** | 수정이 필요하다 (수정 전 Merge 불가) |

---

## 4. AI-assisted 코드 PR 체크리스트

AI가 만든 코드가 포함된 PR을 리뷰할 때 추가로 확인하는 항목이다.

### 기본 확인

```
□ 코드가 실제로 실행되는가 (로컬에서 테스트했는가)
□ 오류 케이스가 처리되었는가 (빈 값, 잘못된 타입)
□ AI가 만든 코드임이 명시되어 있는가 (commit 메시지 또는 PR 설명)
□ 코드 설명이 실제 동작과 일치하는가
□ 불필요한 코드나 주석이 없는가
```

### AI 특이 확인

```
□ 존재하지 않는 함수나 메서드를 사용하지 않는가
□ 테스트 코드가 있거나 동작을 직접 확인했는가
□ 보안 민감 정보(API 키, 비밀번호)가 포함되지 않는가
□ 라이브러리 버전이 현재 환경과 맞는가
```

---

## 5. 좋은 리뷰 코멘트 vs 나쁜 리뷰 코멘트

| 나쁜 코멘트 | 좋은 코멘트 |
|------------|-----------|
| "이거 틀렸어요" | "빈 리스트가 들어오면 IndexError가 발생합니다. `if not items: return []` 를 추가하면 어떨까요?" |
| "코드가 별로예요" | "이 함수가 하는 일이 2가지인데, 분리하면 테스트하기 더 쉬울 것 같습니다." |
| "왜 이렇게 했어요?" | "여기서 `for` 대신 리스트 컴프리헨션을 쓴 이유가 있나요? 성능 때문인지 궁금합니다." |

좋은 리뷰는:
- 구체적인 문제와 이유를 설명한다
- 가능하면 개선 방안도 함께 제시한다
- 질문 형식으로 열린 대화를 유도한다

---

## 6. 리뷰어와 작성자의 역할

### 리뷰어 (Reviewer)

- PR 설명을 먼저 읽고 변경 목적을 이해한다
- Files changed에서 각 변경사항을 확인한다
- 코드를 실행해볼 수 있으면 실행해서 확인한다
- 명확하고 건설적인 코멘트를 남긴다
- 승인 또는 변경 요청으로 완료한다

### 작성자 (Author)

- PR 설명에 변경 목적, 테스트 방법, 스크린샷(UI 변경 시)을 포함한다
- 리뷰 코멘트에 빠르게 응답한다
- 수정하면 리뷰어에게 다시 리뷰를 요청한다
- 코멘트가 해결되면 **Resolve** 버튼을 클릭한다

---

## 7. 따라 하기 실습

### 실습 1. Files changed 읽기

`memo-service` 저장소에서 이전에 만든 PR(또는 새로 만든 PR)을 열고:

1. **Files changed** 탭 클릭
2. 변경된 줄(빨간/초록) 확인
3. 파일을 다 보면 **Viewed** 체크

### 실습 2. 코멘트 남기기

아래 코드로 PR을 만든다:

```python
def add_memo(memos, text):
    memos.append(text)   # 빈 문자열 체크 없음
    return True
```

이 PR에 코멘트를 남긴다:

```
빈 문자열이 들어오면 그대로 추가됩니다.
text.strip()이 빈 문자열이면 추가하지 않도록 처리가 필요할 것 같습니다.
```

**Suggest change** 버튼으로 수정 제안도 남겨본다.

### 실습 3. AI PR 체크리스트 적용

AI가 만든 코드로 PR을 만들고, 위 체크리스트의 각 항목을 직접 확인한다.

---

## 확인 체크리스트

- [ ] Files changed 탭에서 빨간/초록 diff를 읽을 수 있는가
- [ ] 특정 줄에 코멘트를 남길 수 있는가
- [ ] Approve / Request changes를 언제 쓰는지 구분할 수 있는가
- [ ] AI-assisted 코드 PR 체크리스트를 적용할 수 있는가

---

## 참고 자료

- GitHub Docs: Reviewing proposed changes in a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/reviewing-proposed-changes-in-a-pull-request
- GitHub Docs: Commenting on a pull request — https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/commenting-on-a-pull-request
