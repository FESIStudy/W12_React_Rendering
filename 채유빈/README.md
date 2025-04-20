## 📌 Virtual DOM

### 실제 DOM (Real DOM)

- 브라우저가 렌더링하는 **실제 UI 트리 구조**
- DOM은 트리 구조라 **변경 및 탐색 속도는 빠르지만**, UI 업데이트 시에는 다음을 모두 수행해야 함:
  - CSS 재계산
  - 레이아웃 구성
  - 리페인트
- 결과적으로 UI 구성 요소가 많아질수록 전체 DOM 업데이트는 **속도가 느려질 수 있음**

### Virtual DOM

- 실제 DOM 업데이트를 **최소화**하기 위해 도입된 개념
- 실제 DOM의 **스냅샷을 메모리에 저장**하고 변경 사항을 비교 후 필요한 부분만 반영
- React 내부에서 **리액트 엘리먼트와 실제 DOM을 동기화하는 패턴**을 의미함
- 변경 사항 비교 및 반영 과정은 **Reconciliation(재조정)**

| 특징 | Real DOM | Virtual DOM |
|------|----------|--------------|
| 위치 | 브라우저 내부 | 메모리(JS 객체) |
| 속도 | 느림 (전체 업데이트) | 빠름 (부분 업데이트) |
| 효율성 | 낮음 (직접 조작) | 높음 (간접 처리) |
| 개발 편의성 | 복잡함 | 간편함 |
| 브라우저 의존성 | 존재함 | 없음 (React Native 등에서도 사용) |

> 💡 **Virtual DOM은 실제 DOM 조작을 최소화하기 위한 중간 레이어 역할이다.**  
> 변경을 메모리에서 먼저 처리하고 꼭 필요한 변경만 반영하여 렌더링 성능을 높인다.
> 그러나 Virtual DOM이 정말 실제 DOM보다 빠른지에 대한 다양한 의견이 존재한다. (ex. 직접 DOM을 조작하는 방식을 사용하는 Svelte)

<br/>

## 📌 Reconciliation (재조정)

> Virtual DOM과 실제 DOM을 **비교(diff)** 후, **변경된 부분만 실제 DOM에 반영(patch)** 하는 과정


![image](https://github.com/user-attachments/assets/7dda92d7-a71d-4aec-b53e-9c21f5e6f55d)

### 작동 원리

1. `state` 또는 `props` 변경
2. 새로운 Virtual DOM 생성
3. 이전 Virtual DOM과 비교 (diff)
4. 변경된 부분만 실제 DOM에 반영

### 리액트의 가정 (휴리스틱)

1. **다른 타입의 엘리먼트는 완전히 교체**
2. **`key`를 통해 어떤 요소가 유지되어야 할지 표시 가능**

### 비교 알고리즘 (Diffing)

- 루트 노드가 다르면 전체 서브트리를 교체
- 같은 타입의 노드는 속성과 자식 요소만 비교
- 리스트의 경우, `key`로 요소를 추적  
  → `key`가 없다면 전체 리스트 재렌더링

<br/>

## 📌 React Fiber 구조

> React 16부터 도입된 새로운 Reconciliation 엔진

### 기존 방식: Stack Reconciler (React 15 이하)

- DFS 기반 재귀적 트리 탐색
- 동기적 처리 (렌더링 중단 불가)
- 렌더링이 길어지면 **16ms 초과 → UI 끊김**

> 💡 브라우저는 보통 60FPS로 렌더링 → 1프레임은 약 16ms  
> 이 시간 내에 작업이 끝나지 않으면 **프레임 드랍 발생**

---

### React Fiber의 특징

- 렌더링을 **작은 단위로 분할 (작업 단편)**
- **우선순위 기반으로 스케줄링**
- **비동기 렌더링 가능 (작업 중단 및 재개 가능)**

> 💡Fiber가 기존의 Stack Reconciler와 근본적으로 다른 점은 **동시성**이다.  
> DOM 업데이트, 렌더링 로직을 작업 단위로 구분하고 이를 **비동기**로 실행하여 최대 실행 시간이 16ms가 넘지 않도록 제어한다. 
> 기존에 동기적이었던 렌더링 스택이 자바스크립트의 싱글 스레드 환경으로 인해 야기할 수 있는 비효율성 문제를 해결하기 위한 것이다.

| 구분 | Stack Reconciler | Fiber Architecture |
|------|------------------|---------------------|
| 렌더링 방식 | 재귀 호출, 동기 처리 | 반복문 기반, 비동기 가능 |
| 처리 단위 | 전체 트리 한 번에 처리 | 작업 단편 단위로 분할 |
| 작업 중단 | 불가 | 가능 |
| 우선순위 | 없음 | 있음 |
| 트리 구조 | Virtual DOM 트리 | FiberNode 기반 연결형 구조 |
| 사용자 경험 | UI가 멈출 수 있음 | 상호작용 우선 처리 |
| 성능 | 트리 깊을수록 느림 | 긴 작업도 분산 처리 가능 |
| DevTools | Virtual DOM 중심 | Fiber 구조 시각화 가능 |

<br/>

## 📌 JSX vs React Element vs React Node

| 구분 | 설명 |
|------|------|
| **JSX** | JavaScript의 확장 문법으로 UI를 선언적으로 작성 |
| **ReactElement** | JSX가 변환되는 객체로, 실제 DOM 구조를 정의 |
| **ReactNode** | 렌더링 가능한 모든 요소를 포함하는 상위 개념 |

### JSX 예시

```tsx
const element = <div>Hello</div>;
// 아래와 동일
const element = React.createElement('div', null, 'Hello');
```

### ReactElement 구조

```ts
{
  type: 'div',
  props: { children: 'Hello' },
  key: null
}
```

### ReactNode 타입 정의

```ts
type ReactNode =
  | ReactElement
  | string
  | number
  | boolean
  | null
  | undefined
  | ReactNode[]
```
