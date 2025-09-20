<script setup lang="ts">
import { ref, isRef, shallowRef, triggerRef, customRef, reactive, readonly, shallowReactive, toRef, toRefs, toRaw } from 'vue'
// ref做深层次响应 shallow做浅层次响应(只到.value)
// ref底层更新逻辑会调用triggerRef
// import type { Ref } from 'vue'
// 带shallow的都是浅层的 节约性能用的
// ref 取值 赋值 都需要加.value reactive不需要加.value
// reactive 是proxy的 不能直接赋值 否则会破坏响应式对象
// 上一条的解决方案 数组可以 <push 解构> 来添加一个对象 a.push(...b)
// 或者把数组作为一个属性去操作 如 a.b = newarr

// toRef 只能修改响应式对象的值 非响应式视图毫无变化 可以用来修改修改某个对象的属性
// 如 const foo = reactive({ bar: '1', a: '2', c: '3' })
// const d = toRef(foo, 'bar')
// d.value = 'hahah' 同时会触发 foo对象中对应字段的更新
// 使用场景 把响应式的某个对象传递出去又不想丢失响应

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
const man = reactive({ name: '1', age: 2, like: 3, addr: '4' })
const addr = toRef(man, 'addr')
const { name, age, like } = toRefs(man)
const change = () => {
  console.log(name, age, like)
  console.log(addr)
}
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
  <button @click="change">点我</button>
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
