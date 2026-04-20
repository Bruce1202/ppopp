# ✅ 1. `async / await`（最推荐）

这是**最干净、最接近同步写法**的方式。

```typescript
const fetchData = async () => {
  try {
    const res = await api.getUserInfo()
    console.log(res)

    // 一定在请求完成后执行
    doSomething(res)
  } catch (e) {
    console.error(e)
  }
}
```

👉 特点：

- 代码像同步逻辑一样
- 自动“阻塞”后续代码（在函数内部）
- 最适合 Vue3 + TS

👉 注意：

- **只能在 async 函数里用**
- 不会阻塞 UI（只是逻辑顺序等待）

------

# ✅ 2. Promise `.then()` 链式写法

老写法，但仍然可用。

```typescript
api.getUserInfo().then(res => {
  console.log(res)
  doSomething(res)
}).catch(err => {
  console.error(err)
})
```

👉 特点：

- 不需要 async
- 适合简单逻辑

👉 缺点：

- 多层调用会变成“回调地狱”

------

# ✅ 3. 在 Vue 生命周期中等待（组合式 API）

常见于 `setup()` 或 `onMounted`

```typescript
import { onMounted } from 'vue'

onMounted(async () => {
  const res = await api.getUserInfo()
  state.user = res
})
```

👉 特点：

- 页面加载时自动等待数据
- 常用于初始化数据

------

# ✅ 4. `watch / watchEffect` 监听结果再继续

适用于**依赖数据变化触发后续逻辑**

```typescript
watch(() => state.user, (newVal) => {
  if (newVal) {
    doSomething(newVal)
  }
})
```

👉 特点：

- 解耦逻辑
- 适合复杂响应式流程

------

# ✅ 5. 使用 `Promise.all`（多个接口必须全部完成）

当你有**多个 OpenAPI 必须全部返回**

```typescript
const [user, order] = await Promise.all([
  api.getUser(),
  api.getOrder()
])

// 两个都完成才执行
doSomething(user, order)
```

👉 特点：

- 并发执行，提高性能
- 统一等待

------

# ✅ 6. 强制“同步感”的封装（业务封装）

比如封一层 service：

```typescript
class UserService {
  async getUser() {
    return await api.getUserInfo()
  }
}

// 使用
const user = await new UserService().getUser()
```

👉 适合：

- 大型项目
- OpenAPI 自动生成代码场景

------

# ⚠️ 7. ❌ 不推荐：阻塞线程（同步请求）

比如：

```
XMLHttpRequest (sync)
```

👉 问题：

- 会卡死浏览器
- 已被淘汰

------

# 🧠 核心总结（重点）

你说的“**必须等到结果返回才继续**”，本质上只有两种思路：

### ✔️ 推荐优先级

1. **async/await（首选）**
2. Promise.then
3. Promise.all（多请求）

### ✔️ 复杂场景

- watch（响应式联动）
- 生命周期控制（onMounted）



---

---

---

# ❗先说结论（很重要）

👉 **最佳实践是：**

- API 请求 → 用 `async/await` 获取结果 → 存到 `ref`
- `computed` → **只做同步计算（基于已有数据加工）**

------

# ✅ 标准推荐写法（最稳）

```typescript
import { ref, computed, onMounted } from 'vue'

const rawData = ref<any>(null)

// 1️⃣ 先拿 API 数据（异步）
const loadData = async () => {
  rawData.value = await api.getXXX()
}

// 2️⃣ computed 只做“同步加工”
const processedData = computed(() => {
  if (!rawData.value) return []

  return rawData.value.map((item: any) => ({
    ...item,
    displayName: item.name + '（已处理）'
  }))
})

// 3️⃣ 生命周期调用
onMounted(() => {
  loadData()
})
```

------

# 🧠 为什么必须这样？

### ❌ 错误示例（很多人会这样写）

```typescript
const data = computed(async () => {
  const res = await api.getXXX()
  return res
})
```

👉 问题：

- `computed` 返回的是 **Promise**
- 模板里拿不到真实值
- Vue 不会等它 resolve
- 会反复触发请求（严重问题）

------

# ✅ 正确模型（你要记住这个结构）

```
API（异步） → ref（存数据） → computed（同步派生）
```

------

# ✅ 进阶写法（更工程化）

### ✔ 带 loading + error

```typescript
const rawData = ref<any>(null)
const loading = ref(false)
const error = ref<Error | null>(null)

const loadData = async () => {
  loading.value = true
  error.value = null

  try {
    rawData.value = await api.getXXX()
  } catch (e: any) {
    error.value = e
  } finally {
    loading.value = false
  }
}

const processedData = computed(() => {
  if (!rawData.value) return []
  return rawData.value.map((x: any) => x.name)
})
```

------

# ✅ 如果你“真的”想在 computed 触发 API（不推荐）

可以用 `watchEffect` 或 `watch` 替代：

```typescript
watchEffect(async () => {
  const res = await api.getXXX()
  rawData.value = res
})
```

👉 但注意：

- 会自动触发（依赖变化就重新请求）
- 不好控制（容易重复调用）

------

# ✅ 更高级方案（推荐你了解）

如果你项目比较大（你之前提到 OpenAPI + 工具开发），可以考虑：

### 1️⃣ 使用缓存请求库

- Vue Query
- SWR

👉 优点：

- 自动缓存
- 自动 loading / error
- computed 直接用数据

------

# 🔥 最后总结（核心一句话）

👉 **computed 永远不要做异步，只做“结果加工”**
