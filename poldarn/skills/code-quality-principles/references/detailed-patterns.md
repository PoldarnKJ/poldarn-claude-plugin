# Detailed Patterns & Anti-Patterns

## Single Responsibility Violations & Fixes

### Anti-Pattern: Mixed Concerns in Function

```typescript
async function handleUserRegistration(email: string, password: string) {
  const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
  if (!isValid) throw new Error('Invalid email')
  if (password.length < 8) throw new Error('Password too short')

  const hashedPassword = await bcrypt.hash(password, 10)
  const user = await db.users.create({ email, password: hashedPassword })

  await sendEmail(email, 'Welcome!', welcomeTemplate(user.name))
  await analytics.track('user_registered', { userId: user.id })

  return user
}
```

**Issues:** Validation, persistence, notification, and analytics in one function.

### Fix: Separated Concerns

```typescript
const validateRegistration = (email: string, password: string): Result<void, ValidationError> => {
  if (!isValidEmail(email)) return { success: false, error: 'INVALID_EMAIL' }
  if (!isValidPassword(password)) return { success: false, error: 'WEAK_PASSWORD' }
  return { success: true, data: undefined }
}

const createUser = async (email: string, password: string): Promise<User> => {
  const hashedPassword = await bcrypt.hash(password, 10)
  return db.users.create({ email, password: hashedPassword })
}

const notifyUserRegistered = async (user: User): Promise<void> => {
  await Promise.all([
    sendWelcomeEmail(user),
    trackRegistration(user)
  ])
}
```

### Anti-Pattern: Component with Embedded Logic

```tsx
function UserProfile({ userId }: Props) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [editMode, setEditMode] = useState(false)
  const [formData, setFormData] = useState({ name: '', bio: '' })
  const [errors, setErrors] = useState<Record<string, string>>({})

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data)
        setFormData({ name: data.name, bio: data.bio })
      })
      .finally(() => setLoading(false))
  }, [userId])

  const validate = () => {
    const newErrors: Record<string, string> = {}
    if (formData.name.length < 2) newErrors.name = 'Name too short'
    if (formData.bio.length > 500) newErrors.bio = 'Bio too long'
    setErrors(newErrors)
    return Object.keys(newErrors).length === 0
  }

  const handleSubmit = async () => {
    if (!validate()) return
    await fetch(`/api/users/${userId}`, {
      method: 'PATCH',
      body: JSON.stringify(formData)
    })
    setEditMode(false)
  }

  // ... 100+ lines of JSX
}
```

### Fix: Separated Hooks

```tsx
function UserProfile({ userId }: Props) {
  const { user, loading } = useUser(userId)
  const { formData, errors, updateField, validate } = useProfileForm(user)
  const { save, saving } = useProfileMutation(userId)

  if (loading) return <Skeleton />
  if (!user) return <NotFound />

  return <ProfileView user={user} form={{ formData, errors, updateField }} onSave={save} />
}
```

## Type Design Patterns

### Discriminated Unions for State Machines

```typescript
type AsyncState<T, E = Error> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T; updatedAt: Date }
  | { status: 'error'; error: E; retriedCount: number }

const renderState = <T,>(state: AsyncState<T>, render: (data: T) => ReactNode) => {
  switch (state.status) {
    case 'idle': return null
    case 'loading': return <Spinner />
    case 'success': return render(state.data)
    case 'error': return <ErrorBanner error={state.error} />
  }
}
```

### Branded Types for Type Safety

```typescript
declare const __brand: unique symbol
type Brand<T, B> = T & { [__brand]: B }

type UserId = Brand<string, 'UserId'>
type OrderId = Brand<string, 'OrderId'>
type Email = Brand<string, 'Email'>

const UserId = {
  create: (id: string): UserId => id as UserId,
  validate: (id: string): id is UserId => /^usr_[a-z0-9]+$/.test(id)
}

function getUser(id: UserId): Promise<User>
function getOrder(id: OrderId): Promise<Order>

getUser(orderId)
getUser(UserId.create('usr_123'))
```

### Builder Pattern for Complex Objects

```typescript
type EmailConfig = {
  to: Email[]
  cc?: Email[]
  subject: string
  body: string
  attachments?: Attachment[]
}

class EmailBuilder {
  private config: Partial<EmailConfig> = {}

  to(...recipients: Email[]): this { this.config.to = recipients; return this }
  cc(...recipients: Email[]): this { this.config.cc = recipients; return this }
  subject(s: string): this { this.config.subject = s; return this }
  body(b: string): this { this.config.body = b; return this }
  attach(...files: Attachment[]): this { this.config.attachments = files; return this }

  build(): Result<EmailConfig, 'MISSING_REQUIRED_FIELDS'> {
    if (!this.config.to || !this.config.subject || !this.config.body) {
      return { success: false, error: 'MISSING_REQUIRED_FIELDS' }
    }
    return { success: true, data: this.config as EmailConfig }
  }
}
```

## Module Boundary Patterns

### Feature Module Structure

```
features/checkout/
├── index.ts                 # Public exports only
├── types.ts                 # Domain types
├── api.ts                   # API calls (internal)
├── hooks/
│   ├── useCart.ts
│   ├── useCheckout.ts
│   └── index.ts            # Hook barrel
├── components/
│   ├── CartSummary.tsx
│   ├── CheckoutForm.tsx
│   ├── PaymentMethod.tsx
│   └── index.ts            # Component barrel
├── utils/
│   ├── price.ts
│   └── validation.ts
└── __tests__/
```

### Public API Design (index.ts)

```typescript
export type { CartItem, CheckoutState, PaymentMethod } from './types'

export { useCart, useCheckout } from './hooks'

export { CartSummary, CheckoutButton } from './components'
```

### Internal Dependencies Stay Internal

```typescript
import { calculateSubtotal } from '../utils/price'

export const useCart = () => {
}
```

## Library Evaluation Framework

### Decision Matrix

| Factor | Weight | Score (1-5) | Notes |
|--------|--------|-------------|-------|
| GitHub Stars | 15% | | >1k=3, >10k=4, >50k=5 |
| Last Commit | 20% | | <1mo=5, <6mo=4, <1yr=3 |
| TypeScript | 20% | | Native=5, @types=3, None=1 |
| Bundle Size | 15% | | Proportional to functionality |
| Documentation | 15% | | Examples, API docs, guides |
| Community | 15% | | Issues response, ecosystem |

### Red Flags

- No updates in 12+ months
- Issue backlog > 500 with no triage
- Breaking changes without migration path
- No TypeScript support (for TS projects)
- Excessive dependencies
- No test coverage

### When Custom Implementation Wins

1. **Trivial utilities** (< 20 lines)
2. **Performance-critical** paths where library overhead matters
3. **Domain-specific** logic with no generic equivalent
4. **Security-sensitive** code requiring full control
5. **Build-time** vs runtime constraints

## Functional Programming Patterns

### Pure Functions vs Impure Functions

**Anti-Pattern: Impure Function with Side Effects**

```typescript
let totalOrders = 0

function processOrder(order: Order) {
  totalOrders++
  order.status = 'processed'
  console.log(`Processing order ${order.id}`)
  saveToDatabase(order)
  return order
}
```

**Issues:** Mutates global state, mutates input parameter, performs I/O, non-deterministic.

**Fix: Pure Core with Isolated Side Effects**

```typescript
const processOrderPure = (order: Order): Order => ({
  ...order,
  status: 'processed',
  processedAt: new Date().toISOString()
})

const executeOrderProcessing = async (order: Order): Promise<void> => {
  const processed = processOrderPure(order)
  await saveToDatabase(processed)
  logger.info(`Processing order ${order.id}`)
}
```

### Immutability Patterns

**Anti-Pattern: Direct Mutation**

```typescript
function addTag(user: User, tag: string) {
  user.tags.push(tag)
  return user
}

function updateSettings(config: Config, updates: Partial<Config>) {
  Object.assign(config, updates)
  return config
}
```

**Fix: Immutable Updates**

```typescript
const addTag = (user: User, tag: string): User => ({
  ...user,
  tags: [...user.tags, tag]
})

const updateSettings = (config: Config, updates: Partial<Config>): Config => ({
  ...config,
  ...updates
})

const updateNestedField = <T extends object, K extends keyof T>(
  obj: T,
  key: K,
  updater: (value: T[K]) => T[K]
): T => ({
  ...obj,
  [key]: updater(obj[key])
})
```

### Function Composition Patterns

**Anti-Pattern: Nested Function Calls**

```typescript
const result = formatCurrency(
  applyTax(
    applyDiscount(
      calculateSubtotal(items),
      discount
    ),
    taxRate
  )
)
```

**Fix: Pipe/Compose Pattern**

```typescript
const pipe = <T>(...fns: Array<(arg: T) => T>) =>
  (value: T): T => fns.reduce((acc, fn) => fn(acc), value)

const flow = <T, R>(...fns: Function[]) =>
  (value: T): R => fns.reduce((acc, fn) => fn(acc), value as unknown) as R

const calculateFinalPrice = flow<CartItem[], string>(
  calculateSubtotal,
  (subtotal: number) => applyDiscount(subtotal, discount),
  (discounted: number) => applyTax(discounted, taxRate),
  formatCurrency
)

const result = calculateFinalPrice(items)
```

### Higher-Order Functions as Abstractions

**Anti-Pattern: Repeated Logic with Minor Variations**

```typescript
async function fetchUsers() {
  try {
    const response = await api.get('/users')
    return response.data
  } catch (error) {
    console.error('Failed to fetch users:', error)
    throw error
  }
}

async function fetchProducts() {
  try {
    const response = await api.get('/products')
    return response.data
  } catch (error) {
    console.error('Failed to fetch products:', error)
    throw error
  }
}
```

**Fix: Higher-Order Function Abstraction**

```typescript
const createFetcher = <T>(endpoint: string, resourceName: string) =>
  async (): Promise<T> => {
    try {
      const response = await api.get<T>(endpoint)
      return response.data
    } catch (error) {
      console.error(`Failed to fetch ${resourceName}:`, error)
      throw error
    }
  }

const fetchUsers = createFetcher<User[]>('/users', 'users')
const fetchProducts = createFetcher<Product[]>('/products', 'products')
```

### Functional Wrappers (Decorators via Functions)

**Anti-Pattern: Class-Based Decorators for Cross-Cutting Concerns**

```typescript
class UserService {
  @Retry(3)
  @Cache(60)
  @Log()
  async getUser(id: string): Promise<User> {
    return api.get(`/users/${id}`)
  }
}
```

**Fix: Composable Function Wrappers**

```typescript
const withRetry = <T extends (...args: any[]) => Promise<any>>(
  fn: T,
  retries: number
): T => {
  return (async (...args: Parameters<T>) => {
    let lastError: Error
    for (let i = 0; i <= retries; i++) {
      try {
        return await fn(...args)
      } catch (err) {
        lastError = err as Error
      }
    }
    throw lastError!
  }) as T
}

const withCache = <K, V>(fn: (key: K) => Promise<V>, ttlMs: number) => {
  const cache = new Map<K, { value: V; expiry: number }>()
  return async (key: K): Promise<V> => {
    const cached = cache.get(key)
    if (cached && cached.expiry > Date.now()) return cached.value
    const value = await fn(key)
    cache.set(key, { value, expiry: Date.now() + ttlMs })
    return value
  }
}

const withLogging = <T extends (...args: any[]) => any>(
  fn: T,
  name: string
): T => {
  return ((...args: Parameters<T>) => {
    console.log(`[${name}] Called with:`, args)
    const result = fn(...args)
    if (result instanceof Promise) {
      return result.then(r => { console.log(`[${name}] Returned:`, r); return r })
    }
    console.log(`[${name}] Returned:`, result)
    return result
  }) as T
}

const getUser = pipe(
  (id: string) => api.get<User>(`/users/${id}`),
  withRetry(3),
  withCache(60000),
  withLogging('getUser')
)
```

### Replacing Classes with Functions

**Anti-Pattern: Stateful Class for Simple Operations**

```typescript
class PriceCalculator {
  private items: CartItem[] = []
  private discount: number = 0

  addItem(item: CartItem) {
    this.items.push(item)
  }

  setDiscount(discount: number) {
    this.discount = discount
  }

  calculate(): number {
    const subtotal = this.items.reduce((sum, i) => sum + i.price * i.qty, 0)
    return subtotal * (1 - this.discount)
  }
}
```

**Fix: Pure Functions with Data**

```typescript
type Cart = {
  readonly items: readonly CartItem[]
  readonly discount: number
}

const createCart = (): Cart => ({ items: [], discount: 0 })

const addItem = (cart: Cart, item: CartItem): Cart => ({
  ...cart,
  items: [...cart.items, item]
})

const setDiscount = (cart: Cart, discount: number): Cart => ({
  ...cart,
  discount
})

const calculateTotal = (cart: Cart): number => {
  const subtotal = cart.items.reduce((sum, i) => sum + i.price * i.qty, 0)
  return subtotal * (1 - cart.discount)
}

const cart = pipe(
  createCart,
  c => addItem(c, item1),
  c => addItem(c, item2),
  c => setDiscount(c, 0.1),
  calculateTotal
)()
```

### Option/Maybe Pattern for Null Safety

```typescript
type Option<T> = { tag: 'some'; value: T } | { tag: 'none' }

const Some = <T>(value: T): Option<T> => ({ tag: 'some', value })
const None = <T>(): Option<T> => ({ tag: 'none' })

const map = <T, U>(opt: Option<T>, fn: (value: T) => U): Option<U> =>
  opt.tag === 'some' ? Some(fn(opt.value)) : None()

const flatMap = <T, U>(opt: Option<T>, fn: (value: T) => Option<U>): Option<U> =>
  opt.tag === 'some' ? fn(opt.value) : None()

const getOrElse = <T>(opt: Option<T>, defaultValue: T): T =>
  opt.tag === 'some' ? opt.value : defaultValue

const findUser = (id: string): Option<User> =>
  users.find(u => u.id === id)
    ? Some(users.find(u => u.id === id)!)
    : None()

const getUserEmail = (id: string): string =>
  pipe(
    findUser(id),
    opt => map(opt, user => user.email),
    opt => getOrElse(opt, 'unknown@example.com')
  )
```

## Anti-Pattern Catalog

| Anti-Pattern | Symptom | Principle Violated | Fix |
|--------------|---------|-------------------|-----|
| God Component | 500+ LOC, 10+ hooks | SRP | Split into feature components |
| Utility Dumping Ground | utils/ with unrelated functions | Modularity | Move to relevant modules |
| Primitive Obsession | `userId: string` everywhere | Type-First | Use branded types |
| Reinventing Lodash | Custom deepMerge, debounce | Library Adoption | Use established library |
| Prop Drilling | Props passed 4+ levels | Modularity | Context or composition |
| Implicit Any | Missing types on APIs | Type-First | Add explicit types |
| Circular Dependencies | Module A imports B imports A | Modularity | Extract shared to third module |
| Mixed Abstraction Levels | High and low level code mixed | SRP | Separate by abstraction |
| Direct Mutation | `array.push()`, `obj.prop = x` | FP Style | Return new structures |
| Impure Functions | Side effects mixed with logic | FP Style | Isolate side effects |
| Nested Calls | `f(g(h(x)))` deeply nested | FP Style | Use pipe/compose |
| Stateful Classes | Class with mutable state | FP Style | Pure functions + data |
| Imperative Loops | `for`, `while` for transforms | FP Style | `map`, `filter`, `reduce` |
