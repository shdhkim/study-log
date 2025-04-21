# Vue 3 Composition API 예제 모음

---

## ✅ v-model (양방향 데이터 바인딩)

```vue
<!-- v-model -->
<!-- 양방향 데이터 바인딩을 제공 -->
<template>
  <input v-model="message" />
  <button @click="message = '강제 변경됨!'">변수 바꾸기</button>
</template>
<script setup>
import { ref } from 'vue'
const message = ref('초기값')
</script>
<!-- 실행 흐름:
  • input에 "Hello"를 입력하면 message 값이 "Hello"가 되며,
  • 버튼 클릭 시 message = '강제 변경됨!' → input 값도 "강제 변경됨!"으로 자동 변경 -->
```

---

## ✅ ref (원시 타입 반응형)

```vue
<!-- ref -->
<!-- 원시 타입(문자, 숫자 등)이나 단일 값을 반응형으로 만들 때 사용하며,
     .value로 접근/수정 (템플릿에서는 .value 생략 가능).
     ref(123)는 내부적으로 { value: 123 } 형태로 만들어짐. -->
<template>
  <p>{{ count }}</p>
  <button @click="count++">증가</button>
</template>
<script setup>
import { ref } from 'vue'
const count = ref(0)
</script>
```

---

## ✅ reactive (객체 전체 반응형)

```vue
<!-- reactive -->
<!-- 객체 전체가 반응형이 되며, .value 없이 직접 속성 접근 가능 -->
<script setup>
import { reactive } from 'vue'
const state = reactive({
  name: '홍길동',
  age: 27
})
</script>
<template>
  <p>{{ state.name }} ({{ state.age }}세)</p>
  <button @click="state.age++">한 살 추가</button>
</template>
```

---

## ✅ computed (계산된 속성)

```vue
<!-- computed (계산된 속성) -->
<!-- 어떤 값을 다른 값으로 계산해서 자동으로 업데이트하고 싶을 때 사용 -->
<template>
  <input v-model="firstName" placeholder="이름" />
  <input v-model="lastName" placeholder="성" />
  <p>전체 이름: {{ fullName }}</p>
</template>
<script setup>
import { ref, computed } from 'vue'
const firstName = ref('홍')
const lastName = ref('길동')
// firstName 또는 lastName이 바뀌면 fullName이 자동으로 다시 계산됨
const fullName = computed(() => {
  return firstName.value + ' ' + lastName.value
})
</script>
```

---

## ✅ watch (값 변경 감지)

```vue
<!-- watch (감시자) -->
<!-- 값이 바뀔 때 어떤 동작을 실행하고 싶을 때 사용 -->
<template>
  <input v-model.number="count" type="number" />
  <p>카운트: {{ count }}</p>
</template>
<script setup>
import { ref, watch } from 'vue'
const count = ref(0)
watch(count, (newVal, oldVal) => {
  console.log(`count가 ${oldVal} → ${newVal}로 변경됨`)
})
</script>
```

---

