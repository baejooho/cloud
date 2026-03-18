# .gitignore 활용 방법

## .gitignore란?
깃이 **무시할 파일/폴더**를 지정하는 파일.
커밋/푸시할 때 해당 파일들은 제외됨.

---

## 기본 사용법

### 1. 파일 생성
프로젝트 루트에 `.gitignore` 파일 생성

### 2. 작성 규칙

---

## 자주 쓰는 패턴

| 패턴 | 설명 |
|------|------|
| `*.env` | 환경변수 파일 |
| `node_modules/` | npm 패키지 폴더 |
| `dist/` | 빌드 결과물 |
| `*.log` | 로그 파일 |
| `.DS_Store` | Mac 시스템 파일 |
| `__pycache__/` | Python 캐시 |

---

## 주의사항
- 이미 커밋된 파일은 `.gitignore`에 추가해도 무시 안 됨
- 이미 올라간 파일 제거하려면 아래 명령어 사용

- git rm --cached 파일명

- ---

## 템플릿 참고
언어/프레임워크별 템플릿 자동 생성:
👉 https://www.toptal.com/developers/gitignore
