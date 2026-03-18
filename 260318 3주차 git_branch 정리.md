# 🌿 Git Branch

## Branch란?

특정 시점에서 코드의 버전(상태)을 나누어 **별도로 관리**할 수 있도록 하는 기능

- 기본 브랜치(Main, Master)에서 나뭇가지처럼 새로운 브랜치를 생성
- 기능 추가, 오류 수정 등을 **독립적**으로 진행 가능
- 특정 시점에서 갈라져서 서로 영향을 주지 않음

---

## 📋 Branch 명령어

```bash
git branch                          # 브랜치 목록 확인
git branch <브랜치명>                # 브랜치 생성
git switch <브랜치명>                # 브랜치 이동
git switch -c <브랜치명>             # 브랜치 생성 + 이동
git branch -m <기존이름> <새이름>    # 브랜치 이름 수정
git branch -d <브랜치명>             # 브랜치 삭제
```

전체 브랜치 이력 확인:
```bash
git log --all --decorate --oneline --graph
```

---

## 🔀 Branch 병합 - merge

두 브랜치의 변경 이력을 합치는 방식. **병합 커밋**이 생성됨.

```bash
git switch main       # main 브랜치로 이동
git merge <브랜치명>  # 대상 브랜치를 현재 브랜치로 병합
```

병합 취소:
```bash
git merge --abort
```

병합 완료 후 브랜치 삭제:
```bash
git branch -d <브랜치명>
```

---

## ♻️ Branch 병합 - rebase

브랜치의 커밋들을 대상 브랜치의 **최신 커밋 위에 다시 쌓는** 방식. 히스토리가 선형으로 정리됨.

```bash
git switch <브랜치명>  # 재배치할 브랜치로 이동
git rebase main        # main 기준으로 rebase
```

rebase 후 main이 뒤처진 경우, fast-forward merge로 따라붙힘:
```bash
git switch main
git merge <브랜치명>
```

rebase 취소:
```bash
git rebase --abort
```

---

## ⚡ 충돌(Conflict) 해결

### merge 충돌

같은 파일의 같은 부분을 서로 다른 브랜치에서 수정한 경우 발생.

```
<<<<<<< HEAD (Current Change)
- Son
=======
- Min
>>>>>>> test (Incoming Change)
```

**해결 순서**
1. 파일 열어서 원하는 내용으로 직접 수정
2. `git add .`
3. `git commit`

취소: `git merge --abort`

---

### rebase 충돌

rebase 중 충돌이 발생하면 **커밋 단위**로 충돌을 해결해야 함.

**해결 순서**
1. 충돌 파일 직접 수정
2. `git add .`
3. `git rebase --continue`

취소: `git rebase --abort`

---

## 🧪 실습 브랜치 흐름

```
main   ──●──●──────────────────────●── (Sunny add)
          │                         │
leader    └──●──●──●──●            │  → merge → main
                                    │
manager        └──●──●──●──●  → rebase → merge → main
```

| 브랜치 | 주요 작업 |
|--------|-----------|
| main | dept.yaml 생성, Harry/Sunny 추가 |
| leader | dept.yaml에 leader: Kim 추가, leader.yaml 생성 및 Lee/Park 추가 |
| manager | dept.yaml에 manager: Choi 추가, manager.yaml 생성 및 Woo/Min 추가 |
