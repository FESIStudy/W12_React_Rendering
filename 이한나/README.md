## React의 렌더링 프로세스

React의 렌더링은 Render 단계와 Commit 단계로 이루어짐

### 1. **초기 렌더링**

- **Render Phase**
    - JSX를 **React Element**로 변환
    - React Element들을 모아 **Virtual DOM** 생성
- **Commit Phase**
    - 생성된 Virtual DOM을 기반으로 **실제 DOM을 구성**하고 브라우저에 반영

### 2. **재렌더링**

상태(state)나 props가 변경되면 렌더링이 다시 시작됨

- **Render Phase**
    - 변경된 내용을 바탕으로 **새로운 Virtual DOM** 생성
    - 이전 Virtual DOM과 비교(diffing)하여 변경된 부분 식별 ⇒ **재조정 (Reconciliation)**
- **Commit Phase**
    - **변경된 부분만 실제 DOM에 반영**하여 효율적인 업데이트 수행

### 굳이 이러한 복잡한 렌더링 과정을 거치는 이유는?

⇒ 대부분의 상황에 충분히 빠르게 업데이트를 보장하고, DOM 수정을 최소화 하기 위해서



## React의 재조정 (**Reconciliation)**

### 16 이전 버전의 재조정 과정: Stack Reconciliation

React 16 이전에는 재조정(Reconciliation) 과정에서 **재귀 호출 기반의 콜스택(Stack)** 구조를 사용

1. **동기적으로 처리됨** – 한 번 렌더링이 시작되면 중단 불가
2. **트리의 모든 노드를 깊이 우선으로 순회**하여, 상태 변화에 따라 DOM 업데이트 수행

⚠️ 단점: 재귀 호출 기반이라 **렌더링 도중에 브라우저 제어권을 넘길 수 없고**, **사용자 이벤트나 애니메이션이 끊길 수 있음**

⇒ 이 문제를 해결하기 위해 **Fiber 아키텍처**가 도입

### 16 이후 버전의 재조정 과정: Fiber Reconciliation

React 16 이후에는 **Fiber Tree 구조**로 렌더링 시스템이 변경

1. **비동기적으로 처리 가능**
2. **우선순위 기반의 업데이트 가능**

📌 주요 변화
- 함수 호출 대신, 작업 단위를 객체(FiberNode)로 분리함
- Fiber는 **연결 리스트 형태로 구성**되어, 순서를 명시적으로 제어할 수 있음
- 각 FiberNode는 `expirationTime`, `lanes`, `alternate` 등의 **메타데이터**를 가지며, 이를 통해
    - 렌더링 **우선순위 판단**
    - **렌더링 일시 중단 및 재개**
    - **부분적인 업데이트**
    
    가능해짐.
    

⇒ React는 `requestIdleCallback`, `MessageChannel` 등의 API를 활용해 **남는 시간에 렌더링을 조금씩 처리하며**, **더 부드럽고 빠른 사용자 경험**을 제공할 수 있음

⚠️ 단점: 구현이 복잡하고, 작업단위로 분리해야하다 보니 초기렌더링 속도가 느려질 수 있음

## React에서 key가 중요한 이유

React는 리스트를 렌더링할 때, **각 항목을 고유하게 식별**하기 위해 key를 사용함

### 왜 필요할까?

- React는 **Virtual DOM 비교(differentiation)** 시, 이전 Virtual DOM과 새 Virtual DOM을 비교해 **최소한의 DOM 변경**을 시도
- 이때 key가 없거나 잘못 설정되어 있으면 React는 **기존 DOM을 재사용하지 않고 모두 새로 그릴 수 있음**
- 항목을 고유하게 식별해, **불필요한 재렌더링 방지**
- DOM 노드의 **정확한 재사용 및 위치 추적**
- 리스트 내 항목의 **삽입, 삭제, 순서 변경 시** 효율적인 업데이트 가능

### 주의할 점

- 배열의 **index를 key로 사용하는 것은 비추천**
    - 항목의 순서가 바뀌면 index도 바뀌어, 잘못된 DOM 재사용이 발생할 수 있음
    - 대신, 항목이 고유하게 식별될 수 있는 **id 값** 등을 key로 사용하는 것이 좋음


## ReactNode vs React Element vs JSX

| 개념 | 설명 | 예시 |
| --- | --- | --- |
| **JSX** | JavaScript XML. React Element를 만들기 위한 문법 | <div>Hello</div> |
| **ReactElement** | React.createElement()로 생성된 **React 객체** (JSX를 변환한 결과) | React.createElement('div', null, 'Hello') |
| **ReactNode** | **렌더링 가능한 모든 값**을 포함하는 넓은 타입 | ReactElement, string, number, JSX, null, false, ReactElement, 배열 등 |