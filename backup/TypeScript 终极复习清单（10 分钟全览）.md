<html><head></head><body><h1>📝 TypeScript 终极复习清单（10 分钟全览）</h1>
<hr>
<h2>1️⃣ 基础语法</h2>
<ul>
<li>
<p><strong>变量声明</strong>：<code inline="">let</code> / <code inline="">const</code>，类型可省略（TS 自动推断）</p>
</li>
<li>
<p><strong>基本类型</strong>：<code inline="">number</code>, <code inline="">string</code>, <code inline="">boolean</code>, <code inline="">null</code>, <code inline="">undefined</code>, <code inline="">any</code>, <code inline="">unknown</code>, <code inline="">never</code></p>
</li>
<li>
<p><strong>数组/元组</strong></p>
</li>
</ul>
<pre><code class="language-ts">const arr: number[] = [1,2,3]
const tuple: [string, number] = ['a', 1]
</code></pre>
<ul>
<li>
<p><strong>枚举/字面量类型</strong></p>
</li>
</ul>
<pre><code class="language-ts">type Role = 'admin' | 'user'
enum RoleEnum { Admin, User }
</code></pre>
<hr>
<h2>2️⃣ 函数与泛型</h2>
<ul>
<li>
<p><strong>函数类型</strong></p>
</li>
</ul>
<pre><code class="language-ts">function add(a: number, b: number): number { return a + b }
</code></pre>
<ul>
<li>
<p><strong>可选参数 &amp; 默认参数</strong></p>
</li>
</ul>
<pre><code class="language-ts">function greet(name?: string) {}
</code></pre>
<ul>
<li>
<p><strong>泛型函数</strong></p>
</li>
</ul>
<pre><code class="language-ts">function identity&lt;T&gt;(arg: T): T { return arg }
</code></pre>
<ul>
<li>
<p><strong>泛型约束</strong></p>
</li>
</ul>
<pre><code class="language-ts">function merge&lt;T extends object, U extends object&gt;(a: T, b: U): T &amp; U { return {...a,...b} }
</code></pre>
<hr>
<h2>3️⃣ 对象与类型别名</h2>
<ul>
<li>
<p><strong>interface / type</strong></p>
</li>
</ul>
<pre><code class="language-ts">interface User { id: number; name: string }
type PartialUser = Partial&lt;User&gt;
</code></pre>
<ul>
<li>
<p><strong>Mapped Types</strong></p>
</li>
</ul>
<pre><code class="language-ts">type Nullable&lt;T&gt; = { [K in keyof T]: T[K] | null }
type RemoveNullable&lt;T&gt; = { [K in keyof T as Exclude&lt;K,'id'&gt;]: T[K] }
</code></pre>
<ul>
<li>
<p><strong>工具类型</strong>：<code inline="">Partial&lt;T&gt;</code>, <code inline="">Required&lt;T&gt;</code>, <code inline="">Pick&lt;T,K&gt;</code>, <code inline="">Omit&lt;T,K&gt;</code>, <code inline="">Record&lt;K,T&gt;</code>, <code inline="">ReturnType&lt;T&gt;</code>, <code inline="">Parameters&lt;T&gt;</code></p>
</li>
</ul>
<hr>
<h2>4️⃣ 高阶类型模板</h2>
<ul>
<li>
<p><strong>深度可选/深度只读</strong></p>
</li>
</ul>
<pre><code class="language-ts">type DeepPartial&lt;T&gt; = { [K in keyof T]?: T[K] extends object ? DeepPartial&lt;T[K]&gt; : T[K] }
type DeepReadonly&lt;T&gt; = { readonly [K in keyof T]: T[K] extends object ? DeepReadonly&lt;T[K]&gt; : T[K] }
</code></pre>
<ul>
<li>
<p><strong>按值类型筛选</strong></p>
</li>
</ul>
<pre><code class="language-ts">type PickByType&lt;T,V&gt; = { [K in keyof T as T[K] extends V ? K : never]: T[K] }
type OmitByType&lt;T,V&gt; = { [K in keyof T as T[K] extends V ? never : K]: T[K] }
</code></pre>
<ul>
<li>
<p><strong>联合转交叉</strong></p>
</li>
</ul>
<pre><code class="language-ts">type UnionToIntersection&lt;U&gt; = (U extends any ? (k: U)=&gt;void : never) extends ((k: infer I)=&gt;void) ? I : never
</code></pre>
<ul>
<li>
<p><strong>元组工具</strong></p>
</li>
</ul>
<pre><code class="language-ts">type First&lt;T extends any[]&gt; = T extends [infer F, ...any[]] ? F : never
type Last&lt;T extends any[]&gt; = T extends [...any[], infer L] ? L : never
</code></pre>
<ul>
<li>
<p><strong>异步返回类型</strong></p>
</li>
</ul>
<pre><code class="language-ts">type ReturnTypeAsync&lt;T&gt; = T extends (...args:any)=&gt;Promise&lt;infer R&gt; ? R : never
</code></pre>
<hr>
<h2>5️⃣ React / Vue 高阶</h2>
<ul>
<li>
<p><strong>React Hooks</strong></p>
</li>
</ul>
<pre><code class="language-ts">const [state, setState] = useState&lt;number&gt;(0)
const ref = useRef&lt;HTMLInputElement&gt;(null)
const [count, dispatch] = useReducer(reducer, 0)
</code></pre>
<ul>
<li>
<p><strong>泛型组件</strong></p>
</li>
</ul>
<pre><code class="language-ts">type ListProps&lt;T&gt; = { items: T[], render: (item:T)=&gt;React.ReactNode }
function List&lt;T&gt;({items, render}:ListProps&lt;T&gt;) { return &lt;&gt;{items.map(render)}&lt;/&gt; }
</code></pre>
<ul>
<li>
<p><strong>Vue Composition API</strong></p>
</li>
</ul>
<pre><code class="language-ts">const count = ref&lt;number&gt;(0)
const double = computed(() =&gt; count.value * 2)
</code></pre>
<hr>
<h2>6️⃣ Node / API 类型</h2>
<ul>
<li>
<p><strong>Express</strong></p>
</li>
</ul>
<pre><code class="language-ts">app.get('/user/:id', (req: Request&lt;{id:string}&gt;, res: Response) =&gt; {})
</code></pre>
<ul>
<li>
<p><strong>统一接口类型</strong></p>
</li>
</ul>
<pre><code class="language-ts">interface ApiResponse&lt;T&gt; { code: number; message: string; data: T }
async function fetchUser(): Promise&lt;ApiResponse&lt;User&gt;&gt; { ... }
</code></pre>
<hr>
<h2>7️⃣ 模块与全局声明</h2>
<ul>
<li>
<p><strong>global.d.ts</strong>：全局变量 / 接口</p>
</li>
</ul>
<pre><code class="language-ts">declare global {
  interface Window { __APP_VERSION__: string }
  const __BUILD_TIME__: string
}
</code></pre>
<ul>
<li>
<p><strong>module.d.ts</strong>：模块或资源类型声明</p>
</li>
</ul>
<pre><code class="language-ts">declare module '*.png'
declare module '*.scss' { const classes: {[key:string]:string}; export default classes }
declare module 'legacy-lib' { export function init(): void }
</code></pre>
<hr>
<h2>8️⃣ 类型守卫 &amp; 类型安全技巧</h2>
<ul>
<li>
<p><strong>自定义类型守卫</strong></p>
</li>
</ul>
<pre><code class="language-ts">function isString(value: unknown): value is string { return typeof value === 'string' }
</code></pre>
<ul>
<li>
<p><strong>事件系统类型安全</strong></p>
</li>
</ul>
<pre><code class="language-ts">type Events = { login: {user:string} }
class EventBus&lt;E&gt; { on&lt;K extends keyof E&gt;(event:K, cb:(p:E[K])=&gt;void); emit&lt;K extends keyof E&gt;(event:K, p:E[K]) }
</code></pre>
<ul>
<li>
<p><strong>Template Literal + Capitalize</strong></p>
</li>
</ul>
<pre><code class="language-ts">type EventHandler = `on${Capitalize&lt;'click'|'hover'&gt;}` // onClick | onHover
</code></pre>
<hr>
<h2>9️⃣ 团队 &amp; 项目实践</h2>
<ol>
<li>
<p>开启 <code inline="">"strict": true</code></p>
</li>
<li>
<p>接口统一放 <code inline="">types/</code></p>
</li>
<li>
<p>工具类型集中出口 <code inline="">types/index.ts</code></p>
</li>
<li>
<p>推断优先，避免冗余类型</p>
</li>
<li>
<p><code inline="">global.d.ts</code> 管理全局变量</p>
</li>
<li>
<p><code inline="">module.d.ts</code> 管理资源导入/无类型库</p>
</li>
</ol>
<hr>
<h2>🔟 小结口诀</h2>

场景 | 技巧
-- | --
基础类型 | 自动推断 + 明确可选/必选
对象类型 | interface / type + Mapped Types
泛型 | 泛型约束 + 工具类型
高阶类型 | DeepPartial / DeepReadonly / PickByType
React/Vue | Hooks + 泛型组件 + Ref
Node/API | Request/Response + ApiResponse
全局/模块 | global.d.ts / module.d.ts
类型守卫 | value is Type
事件系统 | 泛型 + 索引类型
模板字面量 | on${Capitalize<Event>}

</body></html>