# React + AG Grid 심화 정리 (렌더링 흐름, 편집, 내부 동작)

이 문서는 다음 내용을 심화 정리한다.

1.  AG Grid 렌더링 흐름
2.  rowData → rowNode 변환 과정
3.  Grid Change Detection
4.  refreshCells vs redrawRows
5.  cellRenderer / valueGetter / editable 동작 원리
6.  React + AG Grid CRUD 패턴
7.  useSelector + AG Grid 충돌 문제
8.  실무 안정 패턴

------------------------------------------------------------------------

# 1. AG Grid 렌더링 흐름

AG Grid는 단순 테이블 렌더링이 아니라 다음 단계로 동작한다.

    rowData 입력
         |
         v
    rowNode 생성
         |
         v
    컬럼 정의 처리
         |
         v
    valueGetter 계산
         |
         v
    cellRenderer 실행
         |
         v
    DOM 렌더링

즉 실제 화면 렌더링은 **rowNode 기반**으로 이루어진다.

------------------------------------------------------------------------

# 2. rowData → rowNode 변환

외부 데이터:

``` javascript
const rowData = [
  { id:1, name:"Kim", age:25 },
  { id:2, name:"Lee", age:30 }
]
```

Grid 내부 변환:

    rowNode1
      data -> { id:1, name:"Kim", age:25 }
      rowIndex -> 0
      selected -> false
      editing -> false

    rowNode2
      data -> { id:2, name:"Lee", age:30 }
      rowIndex -> 1
      selected -> false
      editing -> false

즉 rowNode는

    행 데이터 + 행 상태 + 행 메서드

구조다.

------------------------------------------------------------------------

# 3. Grid Change Detection

Grid는 데이터 변경을 감지하면 다음 과정을 수행한다.

    데이터 변경
         |
         v
    change detection
         |
         v
    valueGetter 재계산
         |
         v
    cellRenderer refresh
         |
         v
    DOM 업데이트

데이터 변경 방식에 따라 change detection이 다르게 작동한다.

------------------------------------------------------------------------

# 4. refreshCells vs redrawRows

## refreshCells

``` javascript
api.refreshCells()
```

의미:

    기존 행 구조 유지
    셀 값만 다시 계산
    renderer refresh 시도

특징:

-   가벼운 업데이트
-   valueGetter 재계산
-   셀 UI만 갱신

------------------------------------------------------------------------

## redrawRows

``` javascript
api.redrawRows()
```

의미:

    행 DOM 전체 재생성

특징:

-   renderer 재생성
-   행 전체 재렌더링
-   refreshCells보다 비용 큼

------------------------------------------------------------------------

# 5. cellRenderer 동작 원리

컬럼 정의:

``` javascript
{
  field: "name",
  cellRenderer: (params) => {
    return `<b>${params.value}</b>`
  }
}
```

렌더링 과정:

    rowNode.data.name
           |
           v
    valueGetter 계산
           |
           v
    cellRenderer 실행
           |
           v
    DOM 생성

renderer는 **셀 UI 생성 책임**을 가진다.

------------------------------------------------------------------------

# 6. valueGetter 동작

valueGetter는 셀 값을 계산한다.

``` javascript
valueGetter: (params) => {
  return params.data.firstName + " " + params.data.lastName
}
```

동작:

    rowNode.data
          |
          v
    valueGetter 실행
          |
          v
    value 반환

valueGetter는 데이터 변경 시 재실행될 수 있다.

------------------------------------------------------------------------

# 7. editable 동작

컬럼 설정:

``` javascript
editable: (params) => params.data.editing
```

동작:

    셀 클릭
         |
         v
    editable 함수 실행
         |
         v
    true -> 편집 가능
    false -> 편집 불가

문제:

객체 직접 수정 후 refresh가 안 되면 editable 재평가가 안 될 수 있다.

------------------------------------------------------------------------

# 8. React + AG Grid CRUD 패턴

가장 안정적인 패턴:

    React state = source of truth
    Grid = UI

예시 구조:

    React state
        |
        v
    rowData
        |
        v
    AgGridReact

------------------------------------------------------------------------

## 행 추가

``` javascript
setRowData(prev => [
  ...prev,
  { id:3, name:"Park", age:28 }
])
```

------------------------------------------------------------------------

## 행 수정

``` javascript
setRowData(prev =>
  prev.map(row =>
    row.id === 1
      ? { ...row, name:"Park" }
      : row
  )
)
```

------------------------------------------------------------------------

## 행 삭제

``` javascript
setRowData(prev =>
  prev.filter(row => row.id !== 1)
)
```

------------------------------------------------------------------------

# 9. useSelector + AG Grid 충돌 문제

예:

``` javascript
const users = useSelector(state => state.users)

<AgGridReact rowData={users} />
```

문제:

    params.data === store.users[x]

즉 같은 객체 참조일 수 있다.

그래서

``` javascript
params.data.name = "Park"
```

하면

    Redux store 직접 수정

문제가 발생한다.

------------------------------------------------------------------------

# 10. 안전한 패턴

Grid에 전달하기 전에 복사한다.

``` javascript
const gridData = users.map(u => ({ ...u }))
```

구조:

    Redux users ----> original objects

    gridData ----> new objects

이렇게 하면 Grid 수정이 Redux에 영향을 주지 않는다.

------------------------------------------------------------------------

# 11. React + AG Grid 안정 패턴

실무에서 많이 쓰는 구조:

    Redux / React state
           |
           v
         rowData
           |
           v
          Grid

수정 흐름:

    사용자 편집
         |
         v
    React state 업데이트
         |
         v
    rowData 변경
         |
         v
    Grid rerender

즉

    Grid는 데이터의 source가 아니라
    UI 표현 계층이다.

------------------------------------------------------------------------

# 12. 핵심 요약

AG Grid 핵심 구조:

    rowData = 외부 데이터
    rowNode = Grid 내부 행 객체
    params = 실행 컨텍스트
    transaction = Grid 데이터 업데이트 API

React 환경 핵심 원칙:

    state는 불변성 유지
    객체 직접 수정 지양
    React state를 source of truth로 사용
