# Chapter 03: 서비스에 테스트 추가하기

## 이 장에서 배우는 것

- 테스트가 왜 필요한지
- `assert`로 빠른 테스트 작성하기
- Python `unittest`로 구조적인 테스트 만들기
- 테스트 파일을 분리하는 방법
- 어떤 부분을 테스트해야 하는지 판단하기

---

## 먼저 쉬운 설명

코드를 수정할 때마다 이전에 잘 되던 기능이 깨질 수 있다.

이것을 **regression(회귀 버그)**이라고 한다. 테스트가 없으면 이런 버그를 바로 발견하기 어렵다.

테스트는 "이 함수가 이 입력을 받으면 이 결과를 내야 한다"를 코드로 기록해두는 것이다. 코드를 수정한 뒤 테스트를 실행하면 이전 기능이 깨졌는지 바로 알 수 있다.

---

## 1. assert로 빠른 테스트

가장 간단한 방법이다. 함수를 만들었을 때 바로 검증할 때 쓴다.

```python
# memo_service.py 아래에 테스트 추가

def add_memo(memos, text):
    if not text.strip():
        return False
    memos.append(text.strip())
    return True

def search_memo(memos, keyword):
    return [m for m in memos if keyword.lower() in m.lower()]


# 빠른 테스트 (파일 직접 실행 시에만)
if __name__ == "__main__":
    test_memos = []

    # add_memo 테스트
    assert add_memo(test_memos, "Python 공부") == True
    assert len(test_memos) == 1
    assert add_memo(test_memos, "") == False      # 빈 메모
    assert len(test_memos) == 1                   # 추가 안 됨

    # search_memo 테스트
    add_memo(test_memos, "GitHub 실습")
    add_memo(test_memos, "python 연습")
    results = search_memo(test_memos, "python")
    assert len(results) == 2                      # 대소문자 무관

    print("모든 assert 테스트 통과!")
```

---

## 2. unittest로 구조적인 테스트

규모가 커지면 테스트를 별도 파일로 분리하고 체계적으로 관리한다.

### 테스트 파일 구조

```
memo_app/
├── memo_service.py
└── test_memo_service.py   ← 테스트 파일
```

### test_memo_service.py 작성

```python
import unittest
from memo_service import add_memo, search_memo, delete_memo


class TestAddMemo(unittest.TestCase):
    """add_memo 함수 테스트"""

    def setUp(self):
        """각 테스트 전에 실행 - 빈 메모 리스트 초기화"""
        self.memos = []

    def test_add_normal(self):
        """정상적인 메모 추가"""
        result = add_memo(self.memos, "Python 공부")
        self.assertTrue(result)
        self.assertEqual(len(self.memos), 1)
        self.assertEqual(self.memos[0], "Python 공부")

    def test_add_empty(self):
        """빈 문자열 추가 시도"""
        result = add_memo(self.memos, "")
        self.assertFalse(result)
        self.assertEqual(len(self.memos), 0)

    def test_add_whitespace_only(self):
        """공백만 있는 문자열"""
        result = add_memo(self.memos, "   ")
        self.assertFalse(result)

    def test_add_strips_whitespace(self):
        """앞뒤 공백 제거 확인"""
        add_memo(self.memos, "  Python  ")
        self.assertEqual(self.memos[0], "Python")


class TestSearchMemo(unittest.TestCase):
    """search_memo 함수 테스트"""

    def setUp(self):
        self.memos = ["Python 공부", "GitHub 실습", "python 연습", "AWS 배포"]

    def test_search_case_insensitive(self):
        """대소문자 무관 검색"""
        results = search_memo(self.memos, "python")
        self.assertEqual(len(results), 2)

    def test_search_no_result(self):
        """결과 없는 검색"""
        results = search_memo(self.memos, "Django")
        self.assertEqual(results, [])

    def test_search_empty_keyword(self):
        """빈 키워드 - 전체 반환"""
        results = search_memo(self.memos, "")
        self.assertEqual(len(results), 4)


class TestDeleteMemo(unittest.TestCase):
    """delete_memo 함수 테스트"""

    def setUp(self):
        self.memos = ["Python 공부", "GitHub 실습", "AWS 배포"]

    def test_delete_valid(self):
        """정상 삭제"""
        result = delete_memo(self.memos, 2)
        self.assertTrue(result)
        self.assertEqual(len(self.memos), 2)
        self.assertNotIn("GitHub 실습", self.memos)

    def test_delete_out_of_range(self):
        """범위 밖 번호"""
        result = delete_memo(self.memos, 10)
        self.assertFalse(result)
        self.assertEqual(len(self.memos), 3)


if __name__ == "__main__":
    unittest.main()
```

### 테스트 실행

```bash
python3 test_memo_service.py
```

출력:
```
......
----------------------------------------------------------------------
Ran 8 tests in 0.001s

OK
```

---

## 3. unittest 주요 메서드

| 메서드 | 확인하는 것 | 예시 |
|--------|-----------|------|
| `assertEqual(a, b)` | a == b | `assertEqual(result, "A")` |
| `assertNotEqual(a, b)` | a != b | `assertNotEqual(result, None)` |
| `assertTrue(x)` | x가 True | `assertTrue(is_valid)` |
| `assertFalse(x)` | x가 False | `assertFalse(is_empty)` |
| `assertIn(a, b)` | a가 b에 포함 | `assertIn("Python", memos)` |
| `assertNotIn(a, b)` | a가 b에 미포함 | `assertNotIn("없는것", memos)` |
| `assertRaises(오류, 함수, ...)` | 특정 오류 발생 | `assertRaises(ValueError, int, "abc")` |

---

## 4. 무엇을 테스트해야 할까

모든 줄을 테스트할 필요는 없다. 아래 기준으로 우선순위를 정한다.

**반드시 테스트해야 하는 것:**
- 함수의 핵심 동작 (정상 케이스)
- 엣지 케이스 (빈 값, 경계값)
- 이전에 버그가 발생했던 부분

**테스트하지 않아도 되는 것:**
- `print()` 출력 내용
- 단순한 변수 저장
- Python 내장 함수의 동작

---

## 5. 따라 하기 실습

### 실습 1. memo_service.py 정리하기

함수들이 리스트를 인자로 받도록 수정한다 (전역 변수 대신).

```python
# memo_service.py

def add_memo(memos, text):
    if not text or not text.strip():
        return False
    memos.append(text.strip())
    return True

def show_memos(memos):
    if not memos:
        return []
    return list(memos)

def delete_memo(memos, number):
    if number < 1 or number > len(memos):
        return False
    memos.pop(number - 1)
    return True

def search_memo(memos, keyword):
    if not keyword:
        return list(memos)
    return [m for m in memos if keyword.lower() in m.lower()]
```

### 실습 2. 테스트 파일 작성하기

위 함수들에 대한 `test_memo_service.py`를 작성하고 실행한다.

- `add_memo`: 정상, 빈 문자열, 공백만
- `delete_memo`: 정상, 범위 초과, 빈 리스트
- `search_memo`: 정상, 대소문자 무관, 결과 없음

### 실습 3. 테스트 통과시키기

테스트가 실패한다면 `memo_service.py` 코드를 수정해서 모두 통과시킨다.

```bash
python3 test_memo_service.py -v   # 상세 결과 출력
```

---

## 자주 하는 실수

| 상황 | 증상 | 해결 방법 |
|------|------|----------|
| `setUp`에서 상태 공유 | 테스트 간 데이터 오염 | `setUp`에서 매번 새 리스트 만들기 |
| 테스트 이름이 `test_` 로 시작 안 함 | 테스트가 실행 안 됨 | 반드시 `def test_` 로 시작 |
| import 경로 오류 | `ModuleNotFoundError` | 같은 폴더에서 실행, 경로 확인 |

---

## 확인 체크리스트

- [ ] `assert`로 간단한 함수 테스트를 작성할 수 있는가
- [ ] `unittest.TestCase`를 상속해서 테스트 클래스를 만들 수 있는가
- [ ] `setUp`이 각 테스트 전에 실행됨을 이해하는가
- [ ] 테스트를 실행하고 결과를 읽을 수 있는가

---

## 다음 장

다음 장에서는 이 서비스를 AWS Lambda에 배포한 후 API를 통해 실제로 사용하고 모니터링하는 방법을 배운다.

---

## 참고 자료

- Python unittest — https://docs.python.org/3/library/unittest.html
