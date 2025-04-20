## 1. 정의

- 상태(state)나 props 변경 시 발생하는 Virtual DOM 비교 및 변경 감지 과정
- 이전 Virtual DOM과 새 Virtual DOM을 비교하여 변경점만 추출
- 실제 DOM은 Commit Phase에서 변경 사항만 반영됨

## 2. 흐름

1. 상태 또는 props 변경 발생
2. 새로운 React Element 트리 생성
3. 이전 Virtual DOM과 새 Virtual DOM 비교 (Diffing)
4. 변경된 Fiber 노드에 flags 설정 (Placement, Update, Deletion 등)
5. Commit Phase에서 DOM에 반영

## 3. 비교 방식

- 같은 위치의 노드끼리 비교 (depth-first, left-to-right)
- 타입이 다르면 해당 노드 및 하위 노드 전부 제거 후 새로 생성
- 같은 타입이면 props만 비교
- children이 배열이면 key 기준으로 비교


## 4. props 비교

- 타입이 동일한 경우에만 props 비교 진행
- 변경된 props만 업데이트 대상으로 플래그 설정
- 기존 DOM 노드 유지됨 (재사용)

예:
```tsx
<img src="a.png" /> → <img src="b.png" />
// src 속성만 업데이트됨
```


## 5. children 비교 (리스트 렌더링)

### key가 없는 경우
- 순서(index) 기반 비교
- 중간 삽입/삭제 시 이후 모든 노드가 영향을 받아 비효율적

### key가 있는 경우
- key 값 기준으로 매핑 후 재사용 여부 결정
- 삽입/삭제/이동 여부를 정밀하게 판단 가능

```tsx
{items.map(item => <li key={item.id}>{item.name}</li>)}
```

- key는 동일 계층 내에서만 유효
- key 중복 시 예기치 않은 동작 발생


## 6. Fiber 플래그

- Placement: 새 노드 생성 및 삽입 필요
- Update: 기존 노드에 변경사항 존재
- Deletion: 해당 노드 제거 필요
- ChildDeletion: 하위 자식 중 삭제 대상 있음

해당 플래그는 Commit Phase에서 사용됨


## 7. 컴포넌트 비교

### 함수형/클래스형 컴포넌트
- type이 동일하면 이전 인스턴스 재사용
- 새로운 props로 다시 렌더링

### type 변경
- 기존 인스턴스 언마운트, 새 컴포넌트 마운트
- 상태 초기화됨

## 8. 노드 이동 처리

- 같은 노드라도 위치가 바뀌면 React는 삭제 후 새로 생성
- key 기반 비교로 이를 최적화할 수 있음
- key 없을 경우 순서가 기준이므로 효율 떨어짐

예:
```tsx
// 기존
<li key="a">A</li><li key="b">B</li>

// 변경
<li key="b">B</li><li key="a">A</li>
```
- key가 있으므로 A, B 모두 재사용됨. 위치만 변경됨


## 9. 잘못된 key 사용 사례

- index를 key로 사용하는 경우:
  - 항목의 순서가 바뀔 때 불필요한 마운트/언마운트 발생

```tsx
{items.map((item, idx) => <li key={idx}>{item}</li>)}
```

- 정렬, 필터링, 삽입 시 성능 저하 및 상태 손실 가능성 존재


## 10. 비교 범위 제한

- React는 같은 위치, 같은 깊이의 노드끼리만 비교
- 중첩 구조에서 노드 이동 시 깊은 비교 수행하지 않음
- 이동을 삭제 + 생성으로 처리함

이로 인해 deeply nested 구조에서는 구조 변경을 최소화하거나 key 기반 안정성 확보 필요


## 11. 기타 예외 처리

- Fragment도 비교 대상에 포함됨 (key 가능)
- text node도 변경 시 diff 대상
- Suspense, Lazy 등은 내부적으로 Reconciliation 흐름 차이 있음


## 12. 관련 최적화 기술

- React.memo: props 변경 여부로 컴포넌트 리렌더링 방지
- shouldComponentUpdate: 클래스형에서 렌더링 조건 직접 정의
- useMemo, useCallback: 메모이제이션으로 하위 컴포넌트 렌더링 방지


## 13. 요약 테이블

| 조건                        | 처리 방식                      |
|-----------------------------|-------------------------------|
| 타입 변경                   | 기존 제거, 새로 생성           |
| 동일 타입, props 변경       | 기존 유지, props만 변경        |
| children 배열, key 없음     | index 기반 비교                |
| children 배열, key 존재     | key 기준으로 재사용 판단       |
| 컴포넌트 타입 동일          | 기존 인스턴스 재사용           |
| 컴포넌트 타입 변경          | 기존 언마운트 후 재마운트       |
| 노드 이동 (key 없음)        | 삭제 후 새로 생성              |
| 노드 이동 (key 존재)        | 위치만 변경, 재사용 가능       |

