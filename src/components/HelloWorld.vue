<script setup lang="ts">
import { ref, isRef, shallowRef, triggerRef, customRef, reactive, readonly, shallowReactive, toRef, toRefs, toRaw, computed, watch, watchEffect, stop } from 'vue'
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

// toRaw 将一个响应式对象还原成原始对象(剥离proxy那层)

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

const props = defineProps<{ msg: string }>()

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
// refs适用于解构reactive赋值场景
const { name, age, like } = toRefs(man)
const change = () => {
  console.log(name, age, like)
  console.log(addr)
}

// Vue3 和 Vue2 的不同 核心就是响应式原理改变了
// Vue3 使用的是 Proxy Vue2 使用的是 Object.defineProperty
// Vue2 的不足 对象只能劫持 设置好的数据/新增的数据需要Vue.set()操作 数组只能操作七种方法 修改某一项值无法劫持
// 小满 Vue3+vite 教程有一章(第八章 第九章之间)专讲实现

// computed
// 当依赖的<属性的值>发生改变 才会触发改变 依赖的值不发生变化 则使用<缓存>中的属性值
let firstName = ref<string>('张')
let lastName = ref<string>('三')
let computName = computed<string>({
  get() {
    return firstName.value + '-' + lastName.value
  },
  set(newval) {
    console.log(newval)
    // [firstName.value, lastName.value] = newval.split('-')
    const parts = newval.split('-')
    firstName.value = parts[0] || ''
    lastName.value = parts[1] || ''
  },
})

const changeName = () => {
  computName.value = '更改-名字'
}

let keyWord = ref<string>('')

interface Data {
  name: string,
  price: number,
  num: number
}

let data = reactive<Data[]>([
  {
    name: '小黄鸭',
    price: 12,
    num: 2
  },
  {
    name: '大菜狗',
    price: 20,
    num: 3
  },
  {
    name: '小花猫',
    price: 16,
    num: 5
  },
  {
    name: '大黑猩',
    price: 32,
    num: 2
  },
])

// let $total = ref<number>(0)

// const total = () => {
//   $total.value = data.reduce((prev:number, next:Data) => {
//     return prev + next.num * next.price
//   }, 0)
// }

const total = computed(() => {
  // 没添加搜索版本
  // return data.reduce((prev:number, next:Data) => {
  //   return prev + next.num * next.price
  // }, 0)
  // 添加搜索后版本
  return data.filter(i => i.name.includes(keyWord.value)).reduce((prev:number, next:Data) => {
    return prev + next.num * next.price
  }, 0)
})

const del = (index:number) => {
  data.splice(index, 1)
}

const searchData = computed(() => {
  return data.filter((item:Data) => {
    return item.name.includes(keyWord.value)
  })
})

// ref 或者 reactive 包裹起来的对象才能watch
let message = ref('小满 1')
let message1 = ref('大满 2')
let message2 = reactive({
  foo: {
    bar: {
      name: '小满 2'
    }
  }
})

watch([message, message1], (newval, oldval) => {
  console.log(newval, oldval)
}, {
  deep: true,
  flush: 'pre', // pre sync post
})

// watch ref需要手动开deep reactive默认开启
watch(() => message2.foo.bar.name, (newval, oldval) => {
  console.log(newval, oldval)
})
watchEffect(() => {
  console.log(message, '=====')
  console.log(message1, '------')
  // onInvalidate(() => {
  //   console.log('onInvalidate before')
  // })
})

const stop = watchEffect((onInvalidate) => {
  onInvalidate(() => {})
})

const stopWatch = () => stop()

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
  <div>
    <input type="text" v-model="firstName">
    <input type="text" v-model="lastName">
  </div>
  <div>{{ computName }}</div>
  <button @click="changeName">点我computed</button>
  <div>------*-------</div>
  <div>computed练习</div>
  <div>------*-------</div>
  <div>
    <input type="text" placeholder="搜索" v-model="keyWord">
  </div>
  <div style="margin-top: 20px;">
    <table border width="100%" cellpadding="0" cellspacing="0">
      <thead>
        <tr>
          <th>物品名称</th>
          <th>物品单价</th>
          <th>物品数量</th>
          <th>物品总价</th>
          <th>物品操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in searchData">
          <td align="center">{{ item.name }}</td>
          <td align="center">{{ item.price }}</td>
          <td align="center">
            <button @click="item.num > 1 ? item.num-- : null">-</button>
            {{ item.num }}
            <button @click="item.num < 99 ? item.num++ : null">+</button>
          </td>
          <td align="center">{{ item.num * item.price }}</td>
          <td><button @click="del(index)">删除</button></td>
        </tr>
      </tbody>
      <tfoot>
        <tr>
          <td colspan="5" align="right">
            <!-- 总价: {{ $total }} -->
            总价: {{ total }}
          </td>
        </tr>
      </tfoot>
    </table>
  </div>
  <div>------*-------</div>
  <div>watch练习</div>
  <div>------*-------</div>
  <div>
    <input type="text" v-model="message">
    <input type="text" v-model="message1">
    <div>
      <button @click="stopWatch">停止监听</button>
    </div>
  </div>
  <div>
    <input type="text" v-model="message2.foo.bar.name">
  </div>
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
