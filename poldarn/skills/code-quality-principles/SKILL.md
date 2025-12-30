---
name: Code Quality Principles
description: This skill should be used when implementing new features, writing new code, creating components, designing APIs, or refactoring existing code. Activates on requests like "implement", "create", "build", "add feature", "write function", "design API", "refactor". Provides architectural guidance for single responsibility, modular design, type-first development, and pragmatic library adoption.
version: 1.0.0
model: opus
---

# Code Quality Implementation Guidelines

Apply these principles during implementation to produce maintainable, well-architected code.

## Principle 1: Single Responsibility & Separation of Concerns

### Functions
- One function, one purpose. Split any function that performs multiple independent operations.
- Name functions after their single responsibility: `validateEmail`, not `validateAndSaveEmail`.
- Keep functions concise. Excessive length often signals SRP violation.

### Hooks (React/Vue)
- One hook per concern. `useAuth` handles authentication; `useFormValidation` handles validation.
- Never combine unrelated state in a single hook.
- Extract reusable logic into custom hooks immediately.

### Components
- Components render UI. Business logic belongs in hooks or services.
- Avoid inline logic in JSX/templates. Extract to named functions or hooks.
- Prefer composition: multiple small components over one large component with conditionals.

### State Encapsulation (Critical)
Flat state is a code smell. When a component accumulates multiple `useState` and `useEffect` calls at the same level, extract them into a cohesive custom hook.

```tsx
function UserProfile({ userId }: Props) {
  const [user, setUser] = useState(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState(null)
  const [editMode, setEditMode] = useState(false)
  const [formData, setFormData] = useState({})

  useEffect(() => { /* fetch user */ }, [userId])
  useEffect(() => { /* sync form */ }, [user])
  useEffect(() => { /* validate */ }, [formData])
}
```

**Refactor to:**
```tsx
function UserProfile({ userId }: Props) {
  const { user, loading, error } = useUser(userId)
  const { formData, updateField, errors } = useProfileForm(user)
  const { editing, startEdit, cancelEdit } = useEditMode()
}
```

**Guiding principle:** When state and effects in a component lack cohesive grouping—appearing as a flat list of loosely related concerns—consolidate them into purpose-named hooks that encapsulate related logic.

### Implementation Checklist
- [ ] Can this function be split into two independent functions?
- [ ] Does this hook manage only one concern?
- [ ] Is business logic separated from render logic?
- [ ] Could this component be broken into smaller, reusable pieces?
- [ ] Does this component have flat, scattered state/effects? → Consolidate into cohesive hooks

## Principle 2: Pragmatic Library Adoption

### Before Implementing Generic Functionality
Stop and search when implementing:
- Date/time manipulation → dayjs, date-fns
- Form validation → zod, yup, valibot
- State management → zustand, jotai, @tanstack/query
- HTTP client wrappers → axios, ky, ofetch
- Data transformation → lodash-es, ramda, remeda
- UI primitives → radix-ui, headlessui, shadcn/ui
- Animation → framer-motion, motion
- Utilities → es-toolkit

### Evaluation Criteria
1. Community adoption and popularity (stars as signal, not threshold)
2. Active maintenance and responsiveness
3. TypeScript support (preferably written in TS)
4. Bundle size appropriate for use case
5. Documentation quality and ecosystem

### When to Build Custom
- Business-specific logic with no generic equivalent
- Performance-critical paths requiring optimization
- Trivial utilities where library overhead exceeds value
- When library constraints conflict with requirements

## Principle 3: Modular Architecture

### Directory Organization
Organize by feature/domain, not by technical layer:

```
src/
├── features/
│   ├── auth/
│   │   ├── index.ts           # Public API
│   │   ├── types.ts           # Domain types
│   │   ├── hooks/
│   │   ├── components/
│   │   └── utils/
│   └── dashboard/
│       └── ...
├── shared/                     # Cross-cutting concerns
│   ├── ui/                     # Design system primitives
│   ├── hooks/                  # Truly shared hooks
│   └── utils/                  # Truly shared utilities
└── lib/                        # Third-party integrations
```

### Module Boundaries
- Export through barrel files (`index.ts`) to define public API
- Internal implementations remain private to the module
- Cross-module communication through explicit contracts
- Forbid circular dependencies

### Cohesion & Coupling
- High cohesion: Everything in a module relates to its core purpose
- Low coupling: Modules depend on interfaces, not implementations
- Avoid "utils" folders that become dumping grounds

## Principle 4: Type-First Development

### Types as Specifications
Define types before implementation. Types document intent and enforce contracts.

```typescript
type UserId = string & { readonly brand: unique symbol }

type AuthState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'authenticated'; user: User }
  | { status: 'error'; error: AuthError }
```

### Type Design Patterns

**Discriminated Unions over Optional Properties:**
```typescript
type Result<T, E> =
  | { success: true; data: T }
  | { success: false; error: E }
```

**Branded Types for Domain Identifiers:**
```typescript
type OrderId = string & { readonly __brand: 'OrderId' }
const createOrderId = (id: string): OrderId => id as OrderId
```

**Readonly for Immutability:**
```typescript
type Config = Readonly<{
  apiUrl: string
  timeout: number
}>
```

### Type Completeness Rules
- No implicit `any` - configure `noImplicitAny: true`
- Explicit return types on exported functions
- Type assertions require documented justification
- Prefer type inference for local variables

## Principle 5: Functional Programming Style

Prefer functional programming paradigms: pure functions, immutability, function composition, and functions as the primary unit of abstraction and reuse.

### Pure Functions

Functions should be deterministic and side-effect free whenever possible:

```typescript
const calculateTotal = (items: CartItem[]): number =>
  items.reduce((sum, item) => sum + item.price * item.quantity, 0)

const applyDiscount = (total: number, discount: number): number =>
  total * (1 - discount)

const finalPrice = pipe(calculateTotal, (total) => applyDiscount(total, 0.1))
```

**Characteristics of pure functions:**
- Same input always produces the same output
- No side effects (no mutation of external state, no I/O)
- Referentially transparent (can be replaced with its return value)

### Immutability

Never mutate data. Create new data structures instead:

```typescript
const addItem = (cart: readonly CartItem[], item: CartItem): CartItem[] =>
  [...cart, item]

const updateQuantity = (
  cart: readonly CartItem[],
  id: string,
  quantity: number
): CartItem[] =>
  cart.map(item => item.id === id ? { ...item, quantity } : item)

const removeItem = (cart: readonly CartItem[], id: string): CartItem[] =>
  cart.filter(item => item.id !== id)
```

**Enforcement strategies:**
- Use `readonly` modifier for arrays and object properties
- Prefer `const` over `let`
- Use spread operators and non-mutating array methods (`map`, `filter`, `reduce`)
- Consider `Object.freeze` for runtime immutability

### Function Composition

Build complex operations by composing simple functions:

```typescript
const pipe = <T>(...fns: Array<(arg: T) => T>) =>
  (value: T): T => fns.reduce((acc, fn) => fn(acc), value)

const processUser = pipe(
  validateEmail,
  normalizeUsername,
  hashPassword,
  createUserRecord
)

const formatPrice = pipe(
  (n: number) => n.toFixed(2),
  (s: string) => `$${s}`,
  (s: string) => s.replace(/\B(?=(\d{3})+(?!\d))/g, ',')
)
```

### Functions as Primary Abstraction

Prefer functions over classes as the unit of abstraction and reuse:

```typescript
const createValidator = <T>(rules: Array<(value: T) => string | null>) =>
  (value: T): string[] =>
    rules.map(rule => rule(value)).filter((error): error is string => error !== null)

const emailValidator = createValidator<string>([
  (v) => v.includes('@') ? null : 'Invalid email',
  (v) => v.length > 5 ? null : 'Email too short'
])

const withRetry = <T>(fn: () => Promise<T>, retries: number): Promise<T> =>
  fn().catch(err => retries > 0 ? withRetry(fn, retries - 1) : Promise.reject(err))

const withCache = <K, V>(fn: (key: K) => V): (key: K) => V => {
  const cache = new Map<K, V>()
  return (key: K) => {
    if (!cache.has(key)) cache.set(key, fn(key))
    return cache.get(key)!
  }
}
```

### Higher-Order Functions over Loops

Use declarative higher-order functions instead of imperative loops:

```typescript
const activeUsers = users
  .filter(user => user.isActive)
  .map(user => ({ name: user.name, email: user.email }))
  .sort((a, b) => a.name.localeCompare(b.name))

const groupByStatus = <T extends { status: string }>(items: T[]): Record<string, T[]> =>
  items.reduce((acc, item) => ({
    ...acc,
    [item.status]: [...(acc[item.status] || []), item]
  }), {} as Record<string, T[]>)
```

### Implementation Checklist
- [ ] Is this function pure? Could side effects be isolated?
- [ ] Am I mutating data? Can I return a new structure instead?
- [ ] Can these operations be composed into a pipeline?
- [ ] Should this class be a set of composable functions instead?
- [ ] Am I using loops where `map`/`filter`/`reduce` would be clearer?

## Quick Reference

| Principle | Key Question | Action |
|-----------|--------------|--------|
| SRP | Can this be split? | Split it |
| Flat State | Scattered state/effects? | Consolidate to hooks |
| Libraries | Is this generic? | Search first |
| Modularity | Does this belong here? | Move to correct module |
| Types | Does the type express intent? | Refine the type |
| FP Style | Is this pure and composable? | Refactor to functions |

## Additional Resources

Consult `references/detailed-patterns.md` for:
- Extended type patterns and examples
- Module boundary design strategies
- Library evaluation framework
- Anti-pattern catalog with fixes
