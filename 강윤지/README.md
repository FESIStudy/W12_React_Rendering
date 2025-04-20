## 📌 학습 목표

- React의 렌더링 구조(Fiber, Virtual DOM, Reconciliation)에 대한 이해
- `ReactNode`, `ReactElement`, `JSX`의 차이점 이해
- 리렌더링 최적화 전략(`memo`, `useMemo`, `useCallback`, `key`) 학습

---

## 1. 🧱 React의 렌더링 구조

### ✅ Virtual DOM

- React는 UI를 Virtual DOM 트리 형태로 메모리에 구성한 뒤, 실제 DOM과 비교하여 변경 사항만 반영.
- 실제 DOM보다 가볍고 빠르게 비교 가능
- 변경 감지를 위해 `diffing algorithm` 사용

> 💡 DOM 변경 → 비용 큼 → 최소한으로 바꾸는 게 핵심!
> 

---

### ✅ Fiber Architecture (React 16+)

- React는 재귀 호출(콜스택 기반) 대신 **Fiber 트리** 구조로 렌더링 분할 처리
- Fiber는 각각의 컴포넌트 단위 작업 단위를 의미함
- 작업을 쪼개서 브라우저 프레임 사이에 렌더링을 나눠 수행 → **interruptible rendering**

### 장점

- 높은 우선순위 작업 선처리 가능
- 중단 및 재시도 가능 (예: Suspense)

### DevTools에서 확인 방법

- React DevTools → Component Tree에서 Fiber 구조 확인 가능

---

### ✅ Reconciliation (조정)

- 컴포넌트 트리를 비교하여 변경사항을 반영하는 과정
- 기본적으로는 `key`를 기준으로 동일한 노드인지 판단

### 예제

- key 변경 유무에 따라 리스트 아이템 리렌더링 비교

```tsx
{items.map((item, idx) => (
  <Item key={idx} value={item} />
))}
```

> 🔥 key를 index로 쓰는 것이 항상 안전하진 않다 → 재사용으로 인한 예기치 않은 렌더링 가능성
> 

---

## 2. 🧾 ReactNode, ReactElement, JSX 차이

| 구분 | 설명 |
| --- | --- |
| JSX | React.createElement의 문법적 설탕 |
| ReactElement | JSX 또는 createElement로 만든 실제 객체 |
| ReactNode | React가 렌더링할 수 있는 모든 타입 (Element, string, number, null 등 포함) |

```tsx
const element: ReactElement = <div>Hello</div>;
const node: ReactNode = "Hello"; // string도 포함됨
```

---

## 3. 🧠 리렌더링 최적화 전략

### ✅ `React.memo`

- 컴포넌트의 props가 바뀌지 않으면 리렌더링 방지

```tsx
const MyComponent = React.memo((props) => { ... });
```

### ✅ `useCallback`

- 함수를 메모이제이션하여 컴포넌트 재생성 시 불필요한 함수 재생성을 막음

```tsx
const onClick = useCallback(() => {
  console.log('clicked');
}, []);
```

### ✅ `useMemo`

- 계산 결과 값을 메모이제이션

```tsx
const computedValue = useMemo(() => heavyCalc(input), [input]);
```

### ✅ `key`

- list 렌더링에서 각 항목의 identity로 활용 → 잘못된 key는 성능 저하와 버그 유발
