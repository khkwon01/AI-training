# 빠른 명령어 참조

## 매일 쓰는 기본 명령어

```bash
git status                    # 현재 상태 확인 (가장 자주 씀)
git log --oneline             # 최근 commit 목록 보기
git diff                      # 수정된 내용 보기 (add 전)
git diff --staged             # add된 내용 보기 (commit 전)
```

---

## 작업 흐름 (매번 이 순서)

```bash
git status                    # 1. 상태 확인
git add 파일이름               # 2. 기록 대상 선택
git status                    # 3. add 됐는지 확인
git commit -m "변경 설명"      # 4. 기록 저장
git push origin 브랜치이름    # 5. GitHub에 전송
```

---

## Branch 작업

```bash
git branch                    # 현재 branch 목록
git checkout -b feature/기능  # 새 branch 만들고 전환
git checkout main             # main으로 돌아가기
git merge feature/기능        # 현재 branch에 합치기
git branch -d feature/기능   # branch 삭제 (merge 후)
```

---

## 되돌리기

```bash
git restore 파일이름           # 수정 전으로 되돌리기 (저장 안 된 변경)
git restore --staged 파일이름  # add 취소 (commit은 유지)
git pull origin main          # GitHub 최신 내용 가져오기
```

---

## 상황별 해결

| 상황 | 명령어 |
|------|--------|
| commit 전으로 파일 되돌리기 | `git restore 파일이름` |
| add 취소 | `git restore --staged 파일이름` |
| 최신 코드 가져오기 | `git pull origin main` |
| 현재 branch 확인 | `git branch` |
| 특정 commit 내용 보기 | `git show commit아이디` |

---

## PR 체크리스트

PR을 만들기 전에 확인:

```
□ 로컬에서 테스트가 통과했는가
□ commit 메시지가 명확한가 ("update" X → "메모 삭제 기능 추가" O)
□ PR 제목이 무엇을 했는지 설명하는가
□ PR 설명에 변경 내용, 테스트 방법이 있는가
□ 관련 issue가 있으면 "Closes #번호" 포함했는가
```

PR 리뷰 받을 때 확인:

```
□ 리뷰어 코멘트에 응답했는가
□ 수정 요청을 반영하고 다시 push했는가
□ 완료된 코멘트는 "Resolve" 눌렀는가
```

---

## commit 메시지 패턴

| 유형 | 형식 | 예시 |
|------|------|------|
| 기능 추가 | `feat: 내용` | `feat: 메모 검색 기능 추가` |
| 버그 수정 | `fix: 내용` | `fix: 빈 메모 추가 시 오류 수정` |
| 문서 수정 | `docs: 내용` | `docs: README 설치 방법 추가` |
| 리팩토링 | `refactor: 내용` | `refactor: add_memo 함수 분리` |
| 테스트 | `test: 내용` | `test: add_memo 단위 테스트 추가` |
| 설정 변경 | `chore: 내용` | `chore: .gitignore .env 추가` |
