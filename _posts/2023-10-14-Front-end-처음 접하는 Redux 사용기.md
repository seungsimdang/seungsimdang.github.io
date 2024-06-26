---
published: true
title: '2023-10-14-Front-End-처음 접하는 Redux 사용기'
categories:
  - Front-End
tags:
toc: true
toc_sticky: true
toc_label: 'Front-End'
---

### Redux란?

Redux는 Javascript 애플리케이션 상태 관리 라이브러리 중 하나로, 주로 React 애플리케이션과 함께 사용된다. Redux는 애플리케이션 상태를 중앙 집중식 저장소(Centralized Store)에 보관하고 관리하는 방법을 제공하여, 애플리케이션의 상태 관리를 단순하고 예측 가능하게 만들어준다.

### Redux Data Flow

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/redux%20data%20flow.gif?raw=true)

Redux의 Data Flow는 위 사진과 같이 동작한다. React Component에서 어떠한 이벤트가 발생하면, Reducer가 Action을 Dispatch하여 Store의 내부 상태를 업데이트하고, 새롭게 업데이트된 상태를 이용하여 다시 컴포넌트를 렌더링 시키는 Flow이다.

### Redux 핵심 개념

위에서 간단하게 Flow를 소개했는데 핵심이 되는 개념들을 좀 더 자세하게 설명하자면 다음과 같다.
<br />
<br />
**1. Action**  
Action은 간단한 **JavaScript 객체**이다. 여기에는 우리가 수행하는 작업의 유형을 지정하는 **'type' 속성**이 있으며 선택적으로 redux 저장소에 일부 데이터를 보내는 데 사용하는 **'payload' 속성**을 가질 수 있다.

```json
{ type: 'LIKE_ARTICLE', articleId: 42 }
{ type: 'FETCH_USER_SUCCESS', response: { id: 3, name: 'Mary' } }
{ type: 'ADD_TODO', text: 'Read the Redux docs.' }
```

<br />
<br />

**2. Reducer**  
Reducer는 애플리케이션 상태의 변경 사항을 결정하고 **업데이트된 상태를 반환**하는 함수이다.

```javascript
(previousState, action) => nextState;
```

위와 같이 이전 State와 action object를 받은 후에 다음 상태를 반환한다.

<br />
<br />

**3. Store**  
객체 저장소인 Store는 애플리케이션의 **전체 상태 트리를 보유**한다. 내부 상태를 변경하는 유일한 방법은 **해당 상태에 대한 Action을 전달**하는 것이다.

<br />
<br />

**4. Dispatch**  
Dispatch는 Store의 내장 함수 중 하나로 Reducer에게 Action을 보내라고 명령을 내린다.

<br />
<br />

**5. Subscribe**  
Store의 상태 변화를 감시하고, 상태가 변경될 때 콜백 함수를 호출하여 UI를 업데이트한다.

### Redux Thunk란?

Redux Thunk는 Redux와 함께 사용되는 미들웨어(middleware)중 하나로, 비동기 작업을 처리하고 Redux 액션을 dispatch하는 것을 도와준다. Redux Thunk를 사용하면 store에서 비동기 작업을 처리하고 상태를 업데이트할 때 유용하다.

```typescript
useEffect(() => {
  dispatch(fetchPosts());
}, [dispatch]);

const fetchPosts = (): any => {
  return async function fetchPostsThunk(dispatch: any, getState: any) {
    const response = await axios.get(
      'https://jsonplaceholder.typicode.com/posts'
    );
    dispatch({ type: 'FETCH_POSTS', payload: response.data });
  };
};
```

위 코드에서는 `useEffect` 내에서 `dispatch`를 통해 thunk action creator에서 return하는 함수에서 `dispatch`와 `getState`를 매개변수로 받아와 사용할 수 있도록 해주었다.

![image](https://github.com/seungsimdang/seungsimdang.github.io/blob/master/_images/redux%20thunk%20error.png?raw=true)

위 코드를 실행하면 다음과 같은 에러가 발생하는데, 그 이유는 `dispatch`시에는 인자로 action 객체가 와야 하는데, 함수를 dispatch하려 하기 때문이다. 따라서, Redux Thunk 미들웨어를 설치하고, 함수를 dispatch할 수 있게 해주는 `thunk action creator`를 이용하면 된다.

### Redux Thunk를 사용하는 이유

위에서 `dispatch`시에 함수를 dispatch하기 위해서 Redux Thunk를 사용한다고 알아보았다. 하지만 우리는 기존처럼 Redux Thunk 없이 다음과 같은 방법으로 충분히 데이터를 가져올 수 있다.

```javascript
useEffect(() => {
  fetchPosts();
}, []);

async function fetchPostsThunk(dispatch: any, getState: any) {
  const response = await axios.get(
    'https://jsonplaceholder.typicode.com/posts'
  );
  dispatch({ type: 'FETCH_POSTS', payload: response.data });
}
```

하지만, 굳이 Redux Thunk를 사용하는 이유는, `useDispatch`, `useSelector`와 같은 함수들은 react component 내에서만 사용할 수 있다. 앱이 커졌을 때, Action Creator 함수들은 react component 밖에 따로 폴더를 분리하여 위치시키는것이 일반적인데, component 밖에서도 `useDispatch`, `useSelector`와 같은 함수들을 사용할 수 있도록 하여 한 곳에서 데이터 흐름을 제어하기 위해서 중앙 집중식 접근 방식 미들웨어인 Redux Thunk를 사용하는 것이다.

### TypeScript의 `ReturnType`

해당 어플리케이션을 만들면서 다음과 같이 `rootReducer`를 생성하고 `rootReducer`의 type을 export하여 state의 type을 명시하는 데에 사용하였다.

```typescript
const rootReducer = combineReducers({
  todos,
  counter,
  posts,
});

export default rootReducer;

export type RootState = ReturnType<typeof rootReducer>;
```

```typescript
import { RootState } from './reducers';

...

const counter = useSelector((state: RootState) => state.counter);
const todos = useSelector((state: RootState) => state.todos);
const posts = useSelector((state: RootState) => state.posts);
```

이때 사용한 `ReturnType`은 함수 타입에서 사용되는 유틸리티 타입 중 하나로, 특히 함수의 return type을 추출하는 데에 사용된다. `ReturnType`을 사용하면 함수의 시그니처(signature)에서 반환하는 값의 타입을 추출할 수 있으며, 이를 다른 타입 정의나 변수에 할당할 수 있다.
<br />
<br />
`ReturnType`을 사용했을 때의 장점은, 함수 시그니처가 변경되더라도 반환 유형을 자동으로 업데이트할 수 있다는 장점이 이따.

### 마무리

오늘은 Redux의 개념과 Redux Thunk에 대하여 알아보았다. 솔직히 말하면 하루 사이에 배운 내용들이라 아직 완벽히 이해가 안되는 느낌이다... 😢 다음 시간에 Redux를 사용한 실습을 해보면서 Redux랑 좀 더 친해져봐야겠다 ㅎㅎ
