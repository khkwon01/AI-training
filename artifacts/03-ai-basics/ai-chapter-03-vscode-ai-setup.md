# Chapter 03: VS Code에서 AI 도구 연결하기

## 이 장에서 배우는 것

- VS Code에서 사용할 수 있는 AI 코딩 도구 종류와 차이
- GitHub Copilot 설치, 활성화, 기본 사용법
- Copilot으로 코드 자동완성과 코드 설명 받는 방법
- Claude / ChatGPT 등 외부 AI와 VS Code를 함께 쓰는 방법
- AI 제안을 그대로 쓰지 않고 검토하는 습관

---

## 먼저 쉬운 설명

VS Code 혼자서는 코드를 제안하지 않는다.

AI 도구를 연결하면 이런 것들이 가능해진다.

- 코드를 입력하다가 멈추면 다음 줄을 AI가 제안해 준다
- 함수 이름만 쓰면 함수 전체를 AI가 완성해 준다
- 선택한 코드에 대해 "이게 뭐 하는 코드야?"라고 물어볼 수 있다
- 오류가 나면 "이 오류 고쳐줘"라고 요청할 수 있다

AI 도구는 빠른 초안을 만들어 주는 조수다. 최종 판단은 항상 내가 해야 한다.

---

## 1. VS Code AI 도구 비교

| 도구 | 설치 방법 | 무료 여부 | 특징 |
|------|----------|----------|------|
| **GitHub Copilot** | VS Code 확장 | 유료 (학생 무료) | 코드 자동완성, 채팅, 리뷰 |
| **Codeium** | VS Code 확장 | 무료 | Copilot과 비슷한 자동완성 |
| **Cursor** | 별도 앱 | 일부 무료 | VS Code 기반, AI 편집 특화 |
| **Claude / ChatGPT** | 브라우저 | 일부 무료 | 브라우저에서 별도 사용 |

이 장에서는 **GitHub Copilot** (가장 널리 쓰이는 도구)을 기준으로 설명한다.
무료로 시작하고 싶다면 **Codeium**도 설치 방법이 동일하다.

---

## 2. GitHub Copilot 설치

### 전제 조건

- GitHub 계정이 있어야 한다
- VS Code에 GitHub 계정이 연결되어 있어야 한다 (이전 GitHub 연결 장 참고)

### Copilot 구독

GitHub Copilot은 유료 서비스다. 단, 아래 경우에는 무료다.

- **학생**: GitHub Student Developer Pack 신청 시 무료
- **오픈소스 메인테이너**: 일정 조건 충족 시 무료
- **월 2,000 완성 무료**: Copilot Free 플랜 (제한 있음)

구독 또는 무료 플랜 신청: https://github.com/features/copilot

### VS Code에 Copilot 설치

1. Extensions에서 `GitHub Copilot` 검색
2. **GitHub Copilot** (제작자: GitHub) → **Install**
3. 설치 후 오른쪽 하단에 Copilot 아이콘이 나타남

### Copilot 활성화 확인

VS Code 하단 상태 표시줄에 Copilot 아이콘이 있으면 활성화된 상태다.

아이콘에 X가 표시되면 비활성화 상태 — 클릭해서 Enable 선택.

---

## 3. Copilot 자동완성 사용하기

### 기본 사용법

코드를 입력하다가 잠깐 멈추면 Copilot이 회색으로 제안을 표시한다.

```python
def calculate_average(numbers):
    # Copilot이 다음 줄을 제안함
```

- **Tab**: 제안 수락
- **Esc**: 제안 거절
- **Alt + ]** (Mac: `Option + ]`): 다음 제안 보기
- **Alt + [** (Mac: `Option + [`): 이전 제안 보기

### 주석으로 제안 유도하기

주석에 원하는 기능을 설명하면 Copilot이 코드를 제안한다.

```python
# 리스트에서 평균을 계산하고 소수점 2자리로 반올림하는 함수
def calculate_average(numbers):
```

Tab을 누르면 Copilot이 함수 전체를 완성한다.

### 여러 제안 한꺼번에 보기

`Ctrl + Enter` (Mac: `Cmd + Enter`) 를 누르면 오른쪽에 여러 제안 목록이 열린다.

원하는 제안 위에서 **Accept Solution** 클릭.

---

## 4. Copilot Chat 사용하기

Copilot Chat은 코드에 대해 대화하듯 질문할 수 있는 기능이다.

### Chat 열기

- 왼쪽 Activity Bar의 Copilot 아이콘 클릭
- 또는 `Ctrl + Alt + I` (Mac: `Cmd + Option + I`)

### 사용 예시

코드를 선택하고 Chat에서:

```
이 함수가 무엇을 하는지 설명해줘
```

```
이 코드에서 오류가 발생할 수 있는 부분을 찾아줘
```

```
이 함수에 에러 처리를 추가해줘
```

### 인라인 채팅

코드 안에서 바로 질문하려면:
- `Ctrl + I` (Mac: `Cmd + I`)

커서 위치에 인라인 채팅창이 열린다.

---

## 5. 무료 대안: Codeium 설치

Copilot 없이 무료로 AI 자동완성을 쓰고 싶다면 Codeium이 좋은 선택이다.

1. Extensions에서 `Codeium` 검색
2. **Codeium: AI Coding Autocomplete and Chat** → **Install**
3. 설치 후 Codeium 계정 생성 (무료)
4. 사용법은 Copilot과 동일: 입력 후 Tab으로 제안 수락

---

## 6. 외부 AI (Claude, ChatGPT)와 함께 쓰는 방법

Copilot이 없거나, 더 긴 대화가 필요할 때는 브라우저에서 Claude나 ChatGPT를 함께 활용한다.

### 효과적인 방법

**방법 1 — 코드 복사 붙여넣기**

VS Code에서 코드를 복사 → 브라우저 AI에 붙여넣기 → 질문

```
아래 Python 코드를 설명해줘:

def greet(name):
    print(f"안녕하세요, {name}님")
```

**방법 2 — 오류 메시지 붙여넣기**

터미널의 오류 메시지를 복사 → AI에 붙여넣기

```
아래 오류가 발생했어. 원인과 해결 방법을 알려줘:

NameError: name 'greet' is not defined
```

**방법 3 — 기능 설명 후 코드 요청**

```
Python으로 사용자 이름을 입력받아 인사말을 출력하는 함수를 만들어줘.
초보자도 이해할 수 있게 주석도 추가해줘.
```

### AI 결과를 쓸 때 반드시 하는 것

AI가 코드를 줬을 때 바로 붙여넣기 하지 않는다. 아래 순서를 따른다.

1. **읽기** — 각 줄이 무엇을 하는지 이해한다
2. **실행** — 직접 실행해서 원하는 결과가 나오는지 확인한다
3. **수정** — 내 상황에 맞게 고친다

AI는 빠른 초안을 제공한다. 검토하지 않고 쓰면 예상치 못한 오류가 생긴다.

---

## 7. 따라 하기 실습

### 실습 1. Copilot (또는 Codeium) 설치 및 확인

1. Extensions에서 **GitHub Copilot** 또는 **Codeium** 설치
2. 하단 상태 표시줄에 AI 아이콘이 표시되는지 확인
3. `.py` 파일을 열고 주석 입력 후 Tab 제안 확인

```python
# 두 숫자를 더하는 함수
```

### 실습 2. Copilot Chat으로 코드 설명 받기

1. 아래 코드를 `hello.py`에 작성

```python
def greet(name):
    return f"안녕하세요, {name}님! Python 공부 중이군요."

print(greet("Mina"))
```

2. 코드 전체 선택
3. Copilot Chat 열기 (`Cmd/Ctrl + Alt + I`)
4. "이 코드를 설명해줘" 입력

### 실습 3. 외부 AI에 오류 붙여넣기 연습

1. 아래 코드를 실행해서 오류를 발생시킨다

```python
print(message)  # message를 정의하지 않음
```

2. 오류 메시지를 복사
3. Claude (claude.ai) 또는 ChatGPT에 붙여넣고 원인을 물어본다

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| Copilot 아이콘이 비활성화 | 제안이 전혀 안 뜸 | 아이콘 클릭 → Enable Copilot |
| Tab이 제안 대신 들여쓰기 | AI 제안이 사라짐 | Copilot 제안이 뜰 때까지 기다린 후 Tab |
| 제안이 너무 길거나 엉뚱함 | 원하지 않는 코드가 삽입됨 | Esc로 거절 후 좀 더 구체적인 주석으로 유도 |
| Chat이 영어로 답함 | 이해하기 어려움 | "한국어로 답해줘" 추가 |

---

## 확인 체크리스트

- [ ] VS Code에 AI 도구 (Copilot 또는 Codeium)가 설치되어 있는가
- [ ] 주석을 쓰고 Tab으로 코드 제안을 받을 수 있는가
- [ ] Copilot Chat에서 코드 설명을 받을 수 있는가
- [ ] AI가 준 코드를 바로 쓰지 않고 읽고 확인하는 습관이 있는가
- [ ] 오류 메시지를 AI에 붙여넣어 원인을 물어볼 수 있는가

---

## 한 번 더 생각해 보기

1. AI가 제안한 코드를 검토하지 않고 바로 쓰면 어떤 문제가 생길 수 있을까?
2. Copilot 자동완성과 Copilot Chat은 언제 각각 쓰면 좋을까?
3. 같은 질문을 Claude와 ChatGPT에 했을 때 답이 다를 수 있는 이유는 무엇일까?

---

## 다음 장

다음 장에서는 AI가 만든 코드 초안을 읽고 직접 수정하는 방법을 배운다. AI 도구를 설치한 것만으로는 부족하고, AI 결과를 비판적으로 읽는 능력이 더 중요하다.

---

## 참고 자료

- GitHub Copilot 공식 문서 — https://docs.github.com/en/copilot
- GitHub Copilot VS Code 설정 — https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-your-ide/using-github-copilot-in-visual-studio-code
- Codeium — https://codeium.com
- Claude — https://claude.ai
