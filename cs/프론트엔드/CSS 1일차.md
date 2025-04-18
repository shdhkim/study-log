# **그리드(Grid) vs 플렉스(Flex)**
✅ **그리드(Grid)**  
   - **행(row)과 열(column)을 동시에 제어할 수 있는 2차원 레이아웃 시스템**.  
   - `display: grid;`를 사용하여 컨테이너 내의 요소들을 **행과 열로 배치**.  
   - 복잡한 웹 페이지 레이아웃 구성에 적합.  

✅ **플렉스(Flex)**  
   - **1차원 레이아웃 시스템** (주축(main-axis)을 기준으로 요소들을 정렬).  
   - `display: flex;`를 사용하여 **가로(또는 세로) 방향으로 정렬 & 배치 가능**.  
   - 주로 **수평 또는 수직 정렬을 손쉽게 하기 위해 사용**.  

📌 **정리**
| | **그리드(Grid)** | **플렉스(Flex)** |
|---|---|---|
| **레이아웃 방향** | 2차원 (행 & 열) | 1차원 (주축 기준) |
| **적합한 용도** | 페이지 전체 레이아웃 | 요소 정렬 (내비게이션, 버튼 그룹 등) |
| **주요 속성** | `grid-template-rows`, `grid-template-columns` | `flex-direction`, `justify-content`, `align-items` |

---

# **JavaScript 로딩 최적화**
✅ **속도 최적화 & DOM 로딩 문제 방지**를 위해 `<script>`는 **`<body>` 끝에 배치**하는 것이 일반적으로 가장 좋은 방법.  
✅ 하지만 `defer` 속성을 사용하면 `<head>`에 두어도 **유사한 효과를 얻을 수 있음**.  

📌 **JavaScript 로딩 방식 비교**
| 방식 | 설명 | 장점 | 단점 |
|---|---|---|---|
| `<script>` (기본) | HTML 파싱 중 JavaScript 실행 | 즉시 실행 가능 | HTML 로딩 중단 가능 |
| `<script defer>` | HTML 파싱이 끝난 후 실행 | HTML 파싱 방해 없음 | 이벤트 처리 전까지 실행 지연 |
| `<script async>` | 다운로드 즉시 실행 | 병렬 다운로드 가능 | 실행 순서 보장 안됨 |

💡 **추천 방법**
- **일반적으로 `defer` 속성을 사용하여 `<head>`에 배치하는 것이 효율적**.  
- **크리티컬한 스크립트(필수 기능 포함)는 `<body>` 끝에 배치**.  

---

# **body 태그의 onload 속성**
✅ **`onload` 속성은 HTML 문서의 모든 요소(이미지, 스크립트, 스타일 등 포함)가 완전히 로드된 후 실행**.  
✅ 특정 JavaScript 함수를 실행하도록 지시하는 **이벤트 핸들러**.  

📌 **예제**
```html
<body onload="init()">
    <h1>웹 페이지 로드 완료</h1>
</body>

<script>
    function init() {
        alert("페이지가 완전히 로드되었습니다!");
    }
</script>