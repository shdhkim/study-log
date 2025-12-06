## Git Rebase 상세 설명 (Origin / Local 흐름)

## 시작 상태 (원격 저장소만 존재)

원격 저장소(origin)에 main 브랜치만 존재

origin/main:  
A --- B --- C

로컬 상태

local/main:  
A --- B --- C  
origin/main:  
A --- B --- C

## feature 브랜치에서 로컬 커밋(D, E) 생성

git checkout -b feature  
git add .  
git commit -m "D: 작업 1"  
git add .  
git commit -m "E: 작업 2"

상태

origin/main:  
A --- B --- C  
local/feature:  
A --- B --- C --- D --- E  

D, E는 아직 원격에 없음

## D, E 원격 Push (origin/feature 생성)

git push origin feature

상태

origin/main:  
A --- B --- C  
origin/feature:  
A --- B --- C --- D --- E  
local/feature:  
A --- B --- C --- D --- E  

"D, E가 올라갔다"  
→ origin/main이 아니라 origin/feature에 반영됨

## origin/main 업데이트 발생 (F, G 커밋 추가)

origin/main:  
A --- B --- C --- F --- G  
origin/feature:  
A --- B --- C --- D --- E  

로컬 최신 반영

git fetch origin

## feature 브랜치 Rebase 실행

git checkout feature  
git rebase origin/main

Rebase 동작  
1) C 이후의 D, E 분리  
2) F, G 뒤에 재적용  
3) D’, E’ 새 커밋 생성

결과

origin/main:  
A --- B --- C --- F --- G  
local/feature:  
A --- B --- C --- F --- G --- D' --- E'  
origin/feature:  
A --- B --- C --- D --- E

Rebase는 이력 재작성 (커밋 해시 변경)

## Rebase 후 Push 시 문제 발생

현재 비교

origin/feature:  
A --- B --- C --- D --- E  
local/feature:  
A --- B --- C --- F --- G --- D' --- E'

Push 시도

git push origin feature

오류

! [rejected] feature -> feature (non-fast-forward)

이유: 원격 이력을 갈아엎으려는 동작

## Force Push 필요

git push --force-with-lease origin feature

원격 결과

origin/feature:  
A --- B --- C --- F --- G --- D' --- E'

협업 시 큰 충돌 문제 발생 가능

## 핵심 정리

1. Rebase = 이력 재작성 (커밋 새로 생성)
2. 이미 원격에 공유된 브랜치에서 rebase 시
   - 로컬/원격 이력 불일치
   - Force push 필요
   - 팀원 작업 충돌 발생
3. 선택 기준

| 상황 | merge | rebase |
|------|------|--------|
| 개인 feature | 가능 | 권장 |
| 공유 feature | 가능 | 비추천 |
| main / develop | 권장 | 금지 |

## 결론

- "D, E가 올라갔다" = origin/feature에 반영됨  
- Rebase는 깔끔한 이력에 유리하나 공유 브랜치에서 위험  
- 협업 브랜치는 merge 기반 운영 권장