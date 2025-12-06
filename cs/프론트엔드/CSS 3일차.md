# React 스타일 적용 방식 & CSS 우선순위 완벽 정리

## 1. CSS는 어디에 존재하는가?

CSS는 웹 브라우저가 HTML을 렌더링할 때 사용되는 스타일 규칙이다.  
React는 CSS를 적용하는 다양한 방식을 제공할 뿐, CSS 자체의 우선순위 규칙을 변경하지 않는다.

| 구분 | HTML | JavaScript | React |
|------|------|------------|-------|
| `<style>` 태그 | 존재 | 존재하지 않음 (DOM에서 생성 가능) | JSX 자체에는 없음 |
| 외부 CSS 스타일시트 | 존재 | 언어 수준에서는 없음 | import 로 연결 가능 |
| 스타일 규칙 적용 우선순위 | 브라우저 엔진 기준 | 동일 | 동일 |

---

## 2. React에서 CSS 적용 방식

| 방식 | 예제 코드 | 동작 방식 | 우선순위 특성 |
|------|----------|----------|--------------|
| 외부 CSS 파일 + className | `<div className="box" />` | `<link>`를 통해 CSS 규칙 적용 | 클래스 선택자 수준 |
| Inline Style | `<div style={{ color: "red" }} />` | element.style 직접 수정 | 클래스보다 우선 |
| CSS Module | `className={styles.box}` | 클래스명을 유니크하게 변환 | 클래스 선택자 수준 |
| CSS-in-JS | styled-components 등 | JS가 `<style>` 태그 생성 후 삽입 | 클래스 선택자 수준 (대부분 나중 선언 승리) |
| `<style>` 태그 직접 삽입 | `<style>...</style>` | DOM 상단에 CSS 선언 | 선언 순서 영향 |

---

## 3. JavaScript와 `<style>` 태그의 관계

JavaScript는 `<style>` 태그를 직접 포함하지 않지만  
브라우저 DOM API를 통해 동적으로 생성 및 삽입할 수 있다.

```js
const style = document.createElement("style");
style.textContent = `
  .dynamic {
    font-size: 20px;
    color: blue;
  }
`;
document.head.appendChild(style);
```

CSS-in-JS도 동일한 방식으로 내부에서 `<style>` 태그를 생성한다.

Inline Style은 `<style>` 태그 없이 DOM 속성을 직접 수정하는 방식이다.

---

## 4. CSS 우선순위(Specificity)

우선순위 (높음 → 낮음):

1. `!important`
2. Inline Style (`style={{}}`)
3. ID 선택자 (`#id`)
4. 클래스 / CSS Module (`.class`)
5. 태그 선택자 (`div`, `header`)
6. 상속

동일 우선순위이면 나중에 선언된 스타일이 적용된다. (Cascade)

예시:

```html
<h1 id="title" class="text" style="color: blue;">
  Hello
</h1>
```

```css
.text { color: green; }
#title { color: orange; }
```

최종 적용: blue (inline style)

---

## 5. CSS Module 우선순위

CSS Module은 빌드 시 자동으로 유니크한 클래스명으로 변환되며  
그 우선순위는 일반 `.class` 규칙과 동일하다.

| 방식 | 우선순위 |
|------|---------|
| CSS Module | 클래스 |
| Inline Style | CSS Module보다 우선 |
| `!important` | 최상위 |

---

## 6. CSS-in-JS 우선순위 특징

styled-components 예:

```jsx
const Box = styled.div`
  color: purple;
`;
```

결과:

```html
<style data-styled="active">
  .sc-xyz123 { color: purple; }
</style>
```

대부분 렌더링 시점에 뒤쪽에 선언되므로  
같은 클래스 레벨 충돌 시 우선권을 갖는 경우가 많다.

---

## 7. 실무 권장 스타일 가이드

| 상황 | 권장 방식 |
|------|----------|
| 전역 공통 스타일 | Global CSS |
| 컴포넌트 단위 UI | CSS Module / CSS-in-JS |
| 동적 스타일 변경 | Inline Style or class toggle |
| 타 라이브러리 UI Override | 최소한의 `!important` 활용 |

---

## 8. 핵심 요약

- CSS는 React에서도 브라우저 기반으로 동일하게 동작한다.
- CSS 우선순위는 변하지 않는다.
- Inline > ID > Class(CSS Module 포함) > 태그 선택자
- 동일 우선순위에서는 나중에 선언된 스타일이 적용된다.
- CSS Module = “클래스 선택자” 우선순위
- CSS-in-JS는 `<style>` 태그 생성 기반