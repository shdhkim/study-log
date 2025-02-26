# **HTML, CSS, JavaScript의 실행 방식**
✅ **HTML**  
   - **위에서 아래로 순차적으로 해석되며 DOM을 생성**.  

✅ **CSS**  
   - **비동기적으로 로드되지만, 일부 CSS는 렌더링을 차단할 수 있음**.  

✅ **JavaScript**  
   - **실행되면 HTML 파싱이 중단될 수 있음 (렌더링 차단 가능)**.  
   - `defer` 또는 `async` 속성을 사용하면 **JavaScript가 HTML 파싱을 방해하지 않도록 조절 가능**.  

📌 **JavaScript 로딩 속성 비교**
| 속성 | 실행 시점 | 특징 |
|---|---|---|
| **기본 (없음)** | HTML 파싱 중 중단 후 실행 | 렌더링 차단 발생 가능 |
| **defer** | HTML 파싱 완료 후 실행 | 실행 순서 보장 |
| **async** | HTML과 병렬로 다운로드 후 실행 | 실행 순서 보장 X |

---

# **JavaScript 실행 방식**
✅ **JavaScript는 기본적으로 순차 실행**.  
✅ 하지만 다음과 같은 예외가 있음.  

📌 **JavaScript 실행 예외**
| 개념 | 설명 |
|---|---|
| **호이스팅(Hoisting)** | `var` 선언이 끌어올려짐 |
| **비동기 실행(Async)** | `setTimeout`, `Promise` 등이 비동기로 동작 |
| **이벤트 루프(Event Loop)** | 비동기 작업이 동기 코드 실행 후 실행됨 |

---

# **후행 쉼표 (Trailing Comma)**
✅ **JavaScript 객체에서 마지막 요소 뒤에 쉼표(Comma, `,`)를 추가해도 유효한 문법**.  

📌 **예제**
```javascript
const obj = {
    name: "Alice",
    age: 25,  // 후행 쉼표 (문법적으로 유효)
};