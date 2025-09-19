<script setup lang="ts">
import { ref, isRef, shallowRef, triggerRef, customRef, reactive } from 'vue'
// ref做深层次响应 shallow做浅层次响应(只到.value)
// ref底层更新逻辑会调用triggerRef
// import type { Ref } from 'vue'

function MyRef<T>(value: T) {
  return customRef((track, trigger) => {
    return {
      get() {
        track()
        return value
      },
      set(newVal) {
        value = newVal
        trigger()
      }
    }
  })
}

const obj = MyRef<string>('自定义ref更新')

defineProps<{ msg: string }>()

const count = ref(0) // 用ref或reactive包裹才是响应式
// const count:string = 0

const reduceCount = () => {
  console.log(isRef(count))
  if (count.value === 0) return
  count.value--
  obj.value = '更新'
}
// type M = { name: string }
// const Man1 = ref<M>({ name: '第一种定义方式' })
// const Man2:Ref<M> = ref({ name: '第二种定义方式' })
// const Man3 = ref({ name: '第三种方式 范式 会自己做类型推导' })
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <span class="p10 m10 noselect" @click="reduceCount">-</span>
    <button type="button" class="noselect">count is {{ count }}</button>
    <span class="p10 m10 noselect" @click="count++">+</span>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Learn more about IDE Support for Vue in the
    <a
      href="https://vuejs.org/guide/scaling-up/tooling.html#ide-support"
      target="_blank"
      >Vue Docs Scaling up Guide</a
    >.
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
.p10 {
  padding: 10px;
}
.m10 {
  margin: 10px;
}
.noselect {
  -webkit-touch-callout: none; /* iOS Safari */
  -webkit-user-select: none;   /* Chrome/Safari */
  -khtml-user-select: none;    /* Konqueror */
  -moz-user-select: none;      /* Firefox */
  -ms-user-select: none;       /* Internet Explorer/Edge */
  user-select: none;           /* Non-prefixed version, currently
                                  supported by Chrome and Opera */
}
</style>
