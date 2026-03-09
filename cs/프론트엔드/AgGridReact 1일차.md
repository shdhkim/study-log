# React + Redux + AG Grid 데이터 흐름 정리

## 1. JavaScript 객체 참조(Reference) 개념

JavaScript에서 객체와 배열은 **값이 아니라 참조(reference)** 로
전달된다.

예시:

``` javascript
const obj = { name: "Kim" };

const a = obj;
const b = obj;

a.name = "Park";

console.log(b.name); // Park
```

메모리 구조:

    a ----\
           ----> obj { name: "Park" }
    b ----/

즉 같은 객체를 여러 변수가 참조하면 한 곳에서 수정했을 때 모든 곳에서
변경이 보인다.

이 현상을 보통 **참조 공유(reference sharing)** 라고 한다.

------------------------------------------------------------------------

# 2. React 상태(State) 업데이트 원리

React는 상태 변경을 감지할 때 **깊은 비교(deep comparison)** 를 하지
않는다.

대신 **참조 비교(shallow comparison)** 를 사용한다.

예시:

``` javascript
const [rowData, setRowData] = useState([
  { id: 1, name: "Kim" },
  { id: 2, name: "Lee" }
]);
```

현재 메모리 구조:

    rowData -> ArrayA
                |
                |-- Obj1 { id:1, name:"Kim" }
                |
                |-- Obj2 { id:2, name:"Lee" }

React는 다음과 같은 방식으로 상태 변경을 판단한다.

    prevState === nextState ?

### 상태 변경 감지

새 배열 생성:

``` javascript
setRowData([
  { id:1, name:"Park" },
  { id:2, name:"Lee" }
]);
```

구조:

    rowData -> ArrayB

ArrayA !== ArrayB 이므로 React는 상태가 변경되었다고 판단한다.

### 상태 변경 미감지

객체 직접 수정:

``` javascript
rowData[0].name = "Park";
setRowData(rowData);
```

구조:

    rowData -> ArrayA

참조가 동일하므로 React는 상태 변경을 감지하지 못할 수 있다.

------------------------------------------------------------------------

# 3. React 불변성(Immutable) 패턴

React에서는 다음 패턴이 권장된다.

``` javascript
setRowData(prev =>
  prev.map(row =>
    row.id === 1
      ? { ...row, name: "Park" }
      : row
  )
);
```

동작:

1.  새 배열 생성
2.  수정된 행은 새 객체 생성
3.  나머지 행은 기존 객체 재사용

메모리 구조:

    ArrayB
      |
      |-- Obj3 { id:1, name:"Park" }
      |
      |-- Obj2 { id:2, name:"Lee" }

이 방식은 React의 상태 변경 감지와 잘 맞는다.

------------------------------------------------------------------------

# 4. Redux 상태 오염 문제

Redux에서는 다음 규칙이 중요하다.

    state는 reducer 밖에서 직접 수정하면 안된다.

그러나 AG Grid에서 다음과 같은 코드가 있을 수 있다.

``` javascript
params.data.name = "Park";
```

만약 params.data가 Redux store 객체와 같은 참조라면

    store.users[0] === params.data

이 코드는 사실상 다음과 같다.

    store.users[0].name = "Park"

즉 reducer나 dispatch 없이 store가 수정된다.

이를 **상태 오염(state mutation)** 이라고 한다.

------------------------------------------------------------------------

# 5. AG Grid 내부 구조

AG Grid는 단순한 테이블이 아니라 내부적으로 여러 레이어를 가진다.

    rowData (외부 데이터)
       |
       v
    rowNode (Grid 내부 행 객체)
       |
       v
    Cell Renderer

## rowData

Grid에 전달되는 원본 데이터 배열

``` javascript
const rowData = [
  { id:1, name:"Kim" },
  { id:2, name:"Lee" }
];
```

## rowNode

Grid 내부에서 각 행을 관리하는 객체

예시 구조:

    rowNode
      |
      |-- data
      |-- rowIndex
      |-- selected
      |-- editing
      |-- methods (setDataValue 등)

## params

Grid 콜백에서 전달되는 실행 컨텍스트 객체

대표 속성:

    params.data
    params.node
    params.value
    params.api
    params.rowIndex

------------------------------------------------------------------------

# 6. AG Grid 데이터 수정 방법

React + AG Grid 환경에서는 세 가지 주요 수정 방식이 있다.

## 1. 객체 직접 수정

``` javascript
params.data.name = "Park";
```

특징:

-   Grid 객체 직접 변경
-   React 상태 변경 없음
-   Redux 상태 오염 가능
-   rerender 보장 안됨

------------------------------------------------------------------------

## 2. applyTransaction / updateRowData

``` javascript
api.applyTransaction({
  update: [{ id:1, name:"Park" }]
});
```

특징:

-   Grid 내부 데이터 업데이트
-   change detection 수행
-   React state 자동 변경 없음

------------------------------------------------------------------------

## 3. setRowData

``` javascript
setRowData(prev =>
  prev.map(row =>
    row.id === 1
      ? { ...row, name: "Park" }
      : row
  )
);
```

특징:

-   React 상태 공식 업데이트
-   Redux와 동기화 가능
-   rerender 정상 발생

------------------------------------------------------------------------

# 7. 세 가지 방식 비교

  방식               변경 위치          React 상태   Redux 상태   Grid 화면
  ------------------ ------------------ ------------ ------------ -----------
  params.data 수정   Grid 객체          변경 없음    오염 가능    변경
  applyTransaction   Grid 내부 데이터   변경 없음    변경 없음    변경
  setRowData         React state        변경         변경         변경

------------------------------------------------------------------------

# 8. React + AG Grid 권장 구조

React 기반 애플리케이션에서는 다음 구조가 가장 안정적이다.

    React state (source of truth)
            |
            v
          rowData
            |
            v
         AG Grid

즉 데이터의 진짜 원본은 React state이며

    Grid = UI layer

역할을 한다.

------------------------------------------------------------------------

# 9. 핵심 정리

1.  JavaScript 객체는 참조로 전달된다.
2.  React는 참조 변경으로 상태 변경을 감지한다.
3.  객체 직접 수정은 React 상태 변경을 감지하지 못할 수 있다.
4.  Redux state는 reducer 밖에서 수정하면 안 된다.
5.  AG Grid는 rowNode라는 내부 객체로 행을 관리한다.
6.  params.data는 실제 row 객체 참조일 수 있다.
7.  applyTransaction은 Grid 내부 업데이트 방식이다.
8.  setRowData는 React 상태 업데이트 방식이다.

------------------------------------------------------------------------

# 10. 결론

React + AG Grid 환경에서는 다음 원칙이 가장 안정적이다.

    데이터 원본은 React state 또는 Redux store
    Grid는 화면 표시 역할
    데이터 수정은 setState 또는 dispatch로 수행

직접 객체를 수정하는 방식은 가능하지만 상태 관리와 충돌할 가능성이 높다.
