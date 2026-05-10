# AG Grid React 정리

## 1. AG Grid란

AG Grid는 React에서 가장 많이 사용하는 고성능 Grid 라이브러리 중 하나다.

특징:

- 대용량 데이터 처리 가능
- virtual scrolling 지원
- sorting/filter/pagination 기본 지원
- cell editing 지원
- row selection 지원
- transaction update 지원
- React/Vue/Angular 모두 지원

React에서는 보통:

```jsx
<AgGridReact />
```

컴포넌트 형태로 사용한다.

---

# 2. 기본 구조

```jsx
<AgGridReact rowData={rowData} columnDefs={columnDefs} />
```

핵심은:

- rowData
- columnDefs
- Grid 옵션

3개다.

---

# 3. rowData

실제 데이터.

```jsx
const rowData = [
  { id: 1, name: "kim", age: 20 },
  { id: 2, name: "lee", age: 30 },
];
```

각 객체가 row 1개가 된다.

---

# 4. columnDefs

컬럼 정의.

```jsx
const columnDefs = [{ field: "id" }, { field: "name" }, { field: "age" }];
```

field는 rowData의 key와 연결된다.

즉:

```jsx
{
  field: "name";
}
```

이면:

```jsx
row.name;
```

값을 보여준다.

---

# 5. 자주 쓰는 column 옵션

## headerName

헤더 이름 변경.

```jsx
{
  field: "name",
  headerName: "이름"
}
```

## width

고정 너비.

```jsx
{
  field: "name",
  width: 200
}
```

## flex

남은 공간 비율 분배.

```jsx
{
  field: "name",
  flex: 1
}
```

## sortable

정렬 가능.

```jsx
{
  field: "age",
  sortable: true
}
```

## filter

필터 가능.

```jsx
{
  field: "name",
  filter: true
}
```

## editable

셀 수정 가능.

```jsx
{
  field: "name",
  editable: true
}
```

## hide

컬럼 숨김.

```jsx
{
  field: "id",
  hide: true
}
```

## resizable

컬럼 드래그 크기 조절 가능.

```jsx
{
  field: "name",
  resizable: true
}
```

---

# 6. defaultColDef

공통 컬럼 옵션.

```jsx
const defaultColDef = {
  sortable: true,
  filter: true,
  resizable: true,
};
```

---

# 7. rowSelection

## single

```jsx
rowSelection = "single";
```

한 번에 1개 행만 선택 가능.

## multiple

```jsx
rowSelection = "multiple";
```

다중 선택 가능.

기본적으로 Ctrl + 클릭 사용.

## rowMultiSelectWithClick

```jsx
rowMultiSelectWithClick={true}
```

Ctrl 없이 클릭만으로 다중 선택 가능.

---

# 8. suppressRowClickSelection

```jsx
suppressRowClickSelection={true}
```

행 클릭 선택 막기.

보통 checkboxSelection과 같이 사용한다.

---

# 9. checkboxSelection

```jsx
{
  checkboxSelection: true;
}
```

## headerCheckboxSelection

```jsx
{
  checkboxSelection: true,
  headerCheckboxSelection: true
}
```

---

# 10. onGridReady

```jsx
const onGridReady = (params) => {
  console.log(params.api);
};
```

gridApi와 columnApi를 얻는다.

---

# 11. Grid API

## refreshCells

```jsx
api.refreshCells({
  force: true,
});
```

## selectAll

```jsx
api.selectAll();
```

## deselectAll

```jsx
api.deselectAll();
```

---

# 12. cellRenderer

```jsx
{
  field: "name",
  cellRenderer: (params) => {
    return <b>{params.value}</b>;
  }
}
```

---

# 13. cellStyle

```jsx
{
  field: "status",
  cellStyle: (params) => {
    if (params.value === "Y") {
      return {
        backgroundColor: "gray"
      };
    }

    return null;
  }
}
```

---

# 14. colSpan

```jsx
{
  field: "name",
  colSpan: (params) => {
    if (params.data.useYn === "Y") {
      return 2;
    }

    return 1;
  }
}
```

---

# 15. rowSpan

```jsx
{
  field: "group",
  rowSpan: (params) => {
    return params.data.span;
  }
}
```

---

# 16. 객체 직접 수정 vs applyTransaction

## 객체 직접 수정

```jsx
params.data.name = "lee";
```

JS 객체 메모리 값만 변경.

문제:

- Grid가 변경 감지 못할 수 있음
- 화면 refresh 안 될 수 있음
- sort/filter 재계산 안 될 수 있음

## applyTransaction

```jsx
api.applyTransaction({
  update: [{ id: 1, name: "lee" }],
});
```

AG Grid에 공식적으로 row 변경 통지.

내부적으로:

- rowNode 갱신
- cell refresh
- sorting 반영
- filtering 반영
- animation 반영

등 처리한다.

---

# 17. React state와의 관계

applyTransaction은 AG Grid 내부 상태만 변경할 수 있다.

React state는 자동 변경 안 된다.

---

# 18. getRowId

```jsx
getRowId={(params) => params.data.id}
```

transaction에서 매우 중요하다.

---

# 19. 실무 권장 패턴

## React state 중심

```jsx
setRowData(newData);
```

## 대용량 실시간 처리

```jsx
applyTransaction;
```

조합 사용.

---

# 20. 최종 핵심 정리

| 기능                      | 의미                       |
| ------------------------- | -------------------------- |
| rowData                   | 실제 데이터                |
| columnDefs                | 컬럼 정의                  |
| defaultColDef             | 공통 컬럼 옵션             |
| sortable                  | 정렬                       |
| filter                    | 필터                       |
| editable                  | 수정                       |
| resizable                 | 컬럼 드래그 크기 조절      |
| rowSelection              | row 선택                   |
| suppressRowClickSelection | row 클릭 선택 막기         |
| checkboxSelection         | 체크박스 선택              |
| onGridReady               | Grid 초기화 이벤트         |
| cellRenderer              | 셀 커스텀 렌더링           |
| cellStyle                 | 셀 스타일                  |
| colSpan                   | 가로 병합                  |
| rowSpan                   | 세로 병합                  |
| refreshCells              | 셀 강제 refresh            |
| applyTransaction          | Grid 내부 데이터 안전 갱신 |
| getRowId                  | row 고유 id 식별           |
