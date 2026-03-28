# React / JS Module System 정리

## 1. default export

### 개념

- 파일에서 하나의 대표값을 export하는 방식
- 파일당 하나만 가능

### 예시

```jsx
export default function App() {
  return <div>Hello</div>;
}
```

### import

```jsx
import App from "./App";
```

### 특징

- import 시 이름 자유롭게 변경 가능

```jsx
import MyComponent from "./App";
```

---

## 2. named export

### 개념

- 여러 개 export 가능

### 예시

```jsx
export const A = () => {};
export const B = () => {};
```

### import

```jsx
import { A, B } from "./file";
```

### 특징

- 이름 반드시 동일해야 함

---

## 3. default vs named 비교

| 구분        | default       | named           |
| ----------- | ------------- | --------------- |
| 개수        | 1개           | 여러개          |
| import 이름 | 자유          | 고정            |
| 사용        | 대표 컴포넌트 | 유틸, 여러 함수 |

---

## 4. import 문법 정리

### default import

```jsx
import A from "./file";
```

### named import

```jsx
import { B } from "./file";
```

### 전체 import

```jsx
import * as obj from "./file";
```

- default는 obj.default로 접근

---

## 5. index.js (barrel 패턴)

### 개념

- 여러 파일의 export를 하나로 모으는 파일

### 구조

```
components/
  Button.jsx
  Input.jsx
  index.js
```

### 예시

#### Button.jsx

```jsx
export default function Button() {}
```

#### index.js

```jsx
export { default as Button } from "./Button";
```

### 사용

```jsx
import { Button } from "./components";
```

---

## 6. 동작 원리

```jsx
import { Button } from "./components";
```

동작 과정:

1. ./components는 폴더
2. 내부에서 index.js 찾음
3. index.js 실행

즉:

```jsx
import { Button } from "./components/index.js";
```

---

## 7. index.js 역할

- 폴더를 하나의 모듈처럼 만들어줌
- import 경로 단순화
- 구조 캡슐화

---

## 8. export 재수출 (re-export)

```jsx
export { default as Button } from "./Button";
```

동작:

1. Button의 default import
2. Button이라는 이름으로 다시 export

---

## 9. export \*

```jsx
export * from "./utils";
```

- named export만 포함
- default는 포함 안됨

---

## 10. default 재수출

```jsx
export { default as Button } from "./Button";
```

---

## 11. index.js 장점

1. import 경로 단순화
2. 내부 구조 변경 영향 감소
3. 코드 가독성 향상

---

## 12. 단점

1. 순환참조 위험
2. 실제 위치 추적 어려움

---

## 13. 순환참조 예시

```jsx
// index.js
export { default as Button } from "./Button";

// Button.jsx
import { Button } from "./index";
```

문제:

- index → Button → index 반복

---

## 14. 결론

- index.js는 export 이름이 아니라 파일 이름 규칙
- 폴더 import 시 자동으로 index.js가 실행됨
- export와 import는 완전히 다른 개념
