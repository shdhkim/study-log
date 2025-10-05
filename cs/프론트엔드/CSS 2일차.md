# 부모/자식 position 조합 정리 (relative vs absolute)

## 1) 부모 relative + 자식 relative
- **기준**: 자식 자신의 원래 자리(normal flow 자리)
- **흐름**: 부모·자식 둘 다 흐름 유지 (자리 차지 O)
- **동작**: 자식은 자기 자리에서만 `top/left`만큼 이동
- **영향**: 형제 레이아웃 안 깨짐

```css
.parent { position: relative; background: lightgray; }
.child  { position: relative; top: 10px; left: 10px; background: skyblue; }
```

---

## 2) 부모 relative + 자식 absolute  (실무 최빈)
- **기준**: 부모의 패딩 박스 좌상단 (0,0)
- **흐름**: 부모는 흐름 유지, 자식은 흐름 이탈 (자리 차지 X)
- **동작**: 자식은 부모 기준 좌표에 정확히 위치
- **영향**: 형제 요소와 겹칠 수 있음

```css
.parent { position: relative; background: lightgray; }
.child  { position: absolute; top: 10px; right: 10px; background: pink; }
```

 뱃지, 닫기 버튼, 중앙 정렬, 오버레이 등 **90% 케이스**

---

## 3) 부모 absolute + 자식 relative
- **기준(부모)**: 부모는 상위 포지셔닝 조상(없으면 body) 기준으로 이동
- **기준(자식)**: 자식은 자기 자리 기준에서만 이동
- **흐름**: 부모는 흐름 이탈, 자식은 흐름 유지
- **동작**: 떠 있는 박스(부모) 안에서 자식이 자기 자리 기준으로만 살짝 이동

```css
.parent { position: absolute; top: 40px; left: 40px; background: lightgray; }
.child  { position: relative; top: 5px; left: 5px; background: skyblue; }
```

---

## 4) 부모 absolute + 자식 absolute
- **기준(부모)**: 부모는 상위 포지셔닝 조상(없으면 body) 기준
- **기준(자식)**: 자식은 부모 패딩 박스 좌상단 기준
- **흐름**: 부모·자식 모두 흐름 이탈
- **동작**: 부모도 자유롭게 뜨고, 그 안에서 자식도 절대 좌표 배치
- **영향**: 둘 다 흐름 밖이라 다른 형제와 겹치기 쉬움

```css
.parent { position: absolute; top: 40px; left: 40px; background: lightgray; }
.child  { position: absolute; bottom: 10px; right: 10px; background: pink; }
```

---

#  비교 표

| 부모 \\ 자식 | relative | absolute |
|--------------|----------|----------|
| **relative** | 자식 자기 자리에서 살짝 이동 (흐름 유지) | 부모 기준 절대 배치 (흐름 이탈) |
| **absolute** | 부모는 떠 있고, 자식은 자기 자리 기준 보정 | 부모도 떠 있고, 자식도 부모 기준 절대 배치 |

---

#  정리
- **relative**: 자기 자리 기준, 흐름 유지
- **absolute**: 부모(조상) 기준, 흐름 이탈
- 가장 많이 쓰는 건 **부모 relative + 자식 absolute** → 기준점 안정 + 자유 배치 가능
