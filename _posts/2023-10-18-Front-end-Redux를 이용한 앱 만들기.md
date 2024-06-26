---
published: true
title: '2023-10-18-Front-End-Redux를 이용한 앱 만들기'
categories:
  - Front-End
tags:
toc: true
toc_sticky: true
toc_label: 'Front-End'
---

### 시작하며...

저번시간에 Redux에 대한 개념을 공부했었다. 이번 시간에는 Redux를 이용해 앱을 만들면서 이론적인 내용들이 어떻게 쓰이는지 실습을 해보았다. 이번 글에서는 실습을 하면서 강사님이 강조하셨던 부분이나 이해가 안됐던 부분을 복습해보도록 하겠다.

### React Functional Component(React.FC)의 type 지정

React.FC는 저번 토이프로젝트를 하면서 처음 접해본 개념이다. 그때는 TypeScript를 배운지 얼마 되지 않았기 때문에 Component 구현 시 검색한 결과를 참고할 일이 많았다. 많은 레퍼런스들에서 타입 지정 시에 React.FC를 사용하였기 때문에 React에서 Functional Component의 타입 지정을 할 때는 React.FC를 대중적으로 사용하는줄 알았다.
<br />
<br />
하지만 나는 의문이 들어 검색을 해본 결과, 요즘은 React.FC의 사용을 지양하는 추세라고 한다. 그 이유는 몇가지가 있었다.

**1. 타입 추론이 어렵다.**  
 React.FC를 사용하면 TypeScript가 컴포넌트의 props 및 상태를 추론하는 데 어려움을 겪을 수 있다. React.FC를 사용하면 defaultProps 및 propTypes와 같은 추가적인 컴포넌트 속성을 적용하기 어렵다.
<br />
<br />

**2. 더 일반적인 함수 컴포넌트 사용**  
React.FC를 사용하지 않고도 TypeScript에서 함수형 컴포넌트를 정의할 수 있다. 실제로, 실습에서 사용한 코드를 통해 예시를 들어보면 다음과 같다.

```typescript
const Form: FC<FormProps> = ({
  title,
  getDataForm,
  firebaseError,
}: FormProps) => {

...
```

위 코드는 React.FC를 사용하여 컴포넌트의 타입을 지정하였다.

```typescript
const Form = ({ title, getDataForm, firebaseError }: FormProps) => ({
  title,
  getDataForm,
  firebaseError,
}: FormProps) => {

...
```

하지만, React.FC를 사용하지 않고도 위 코드와 같이 함수형 컴포넌트를 정의할 수 있다.
<br />
<br />
이밖에도, "children prop이 옵셔널로 포함되어 props 타입이 명확하지 않다"라는 단점과, "타입스크립트의 제네릭 문법을 지원하지 않는다." 등의 단점이 존재하기 때문에 사용을 지양한다고 한다.

### `mode: 'onBlur'`란?, 제너릭으로 타입을 지정해준 이유는?

```typescript
const {
  register,
  handleSubmit,
  formState: { errors },
  reset,
} = useForm<Inputs>({ mode: 'onBlur' });
```

위 코드에서 사용된 `onBlur` 이벤트에 관해서 이론 강의 시간에 다루셨다고 강사님이 말씀하셨는데, 필수 강의에 없었던건지 내가 까먹은건지 모르겠지만 처음 들어보는 생소한 문법이어서 어떤 역할을 하는 친구인지 찾아보았다.
<br />
<br />
`mode: 'onBlur'`를 설정하는 것은 React Hook Form의 동작 방식을 결정하는 중요한 옵션 중 하나라고 한다. 이 옵션은 입력 필드의 값이 변경될 때가 아니라 해당 필드에서 포커스가 벗어났을 때 유효성 검사를 실행하도록 설정한다. 이는 사용자가 입력 필드를 편집 중인 동안 모든 입력 필드의 유효성을 검사하는 것이 아니라 입력이 완료된 후에 검사하는 방식을 나타낸다. 이를 통해 사용자 경험이 더 좋아질 수 있고 성능을 향상시키는 효과가 있다고 한다.
<br />
<br />
요약하자면, 폼 상태 및 유효성 검사를 처리하는 데 쓰이는 Hook인 `useForm` 사용 시, `mode: 'onBlur'` 설정은 입력 필드의 유효성을 포커스가 벗어난 후에 검사하도록 하는 mode라고 한다.
<br />
<br />

```typescript
type Inputs = {
  email: string;
  password: string;
};
```

`Inputs`은 위와 같이 선언해주었을 때, `useForm`의 타입 지정을 제너릭을 사용해서 해주었는데, 그 이유에 대해서도 궁금해졌다. 그 이유는 크게 2가지가 있었다.

**1. 유형 안정성 및 타입 추론**  
`useForm`과 같은 React Hook Form을 사용할 때 해당 타입을 제너릭으로 지정하면 TypeScript가 코드에서 사용된 필드에 관련된 정보에 대한 타입 검사 및 추론을 수행할 수 있다는 장점이 있다.
예를 들어, 위 코드와 같이 `Inputs`를 지정하면, TypeScript는 `register` 및 `errors`와 관련된 필드에 대한 타입 정보를 제공하고 코드에서 해당 필드를 안전하게 사용할 수 있다. 만약 필드가 추가되거나 변경되면 `Inputs` 타입을 업데이트함으로써 변경 사항을 감지하고 오류를 방지할 수 있다.
<br />
<br />

**2. 가독성 및 유지 보수성**  
제너릭을 사용하여 타입 지정을 하면 코드의 가독성과 유지 보수성을 향상시킬 수 있다는 장점이 있다. 코드 리뷰 시, 어떤 필드가 Form에 속하는지 명확하게 이해할 수 있다. 또한, `Inputs`에서 필드 유효성 검사 규칙을 정의할 수 있어 추후 유지 보수에 용이하다.

### `SubmitHandler`란?

form작성이 끝나면 호출되는 `onSubmit`함수를 다음과 같이 작성하였다.

```typescript
const onSubmit: SubmitHandler<FieldValues> = ({ email, password }) => {
  getDataForm(email, password);
  reset();
};
```

이때, 함수의 타입 지정을 React Hook Form 라이브러리에서 제공하는 타입`SubmitHandler`를 이용하여 해주었는데, 이것이 무엇인지 자세히 살표보도록 하겠다.
<br />
<br />
앞에서도 언급했듯이, `SubmitHandler`는 React Hook Form에서 제공하는 TypeScript 타입으로, 폼 제출 처리 함수의 시그니처를 정의하는 데 사용된다. 해당 타입은 다음과 같이 정의되는데,

```typescript
type SubmitHandler<TFieldValues extends FieldValues = FieldValues> = (
  data: UnpackNestedValue<TFieldValues>,
  event?: React.BaseSyntheticEvent
) => void | Promise<void>;
```

위의 타입 정의에 대한 설명은 다음과 같다.

1. `TFieldValues`  
   폼 필드의 데이터 모델을 나타낸다. 이것은 보통 Inputs 또는 다른 사용자 정의 데이터 모델 타입일 수 있다.
2. `data`  
   제출 처리 함수가 받는 인자로, 폼에서 수집한 데이터가 포함된 객체이다.
3. `event`  
   선택적으로 제공할 수 있는 React 합성 이벤트 객체이다.

위와 같이 간단히 `SubmitHandler` 타입에 대해서 알아보았다. 그래서 "`onSubmit` 함수에서 `SubmitHandler`를 사용함으로써 어떤 이점이 있냐?" 는 위에서 설명한 제너릭을 사용한 타입 지정 시의 장점과 동일하다.

### `TypedUseSelectorHook`이란?

```typescript
export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

해당 앱을 만들때, `useSelector` 사용 시, 일일히 `RootState`라는 타입을 지정해주지 않기 위해 `useAppSelector`를 export하여 사용하였다. 이때 타입 지정에 사용된 `TypedUseSelectorHook`에 대해서 알아보겠다.
<br />
<br />
Redux Store는 상태를 중앙에서 관리하고 Action을 Dispatch하여 상태를 변경한다.  
TypeScript 사용 시, `useSelector`를 사용할 때 Redux에서 Reducer로 부터 반환되는 상태의 타입을 정확하기 추론하기 어려운 경우가 있는데 이때 사용되는 것이 `TypedUseSelectorHook`이다.
<br />
<br />
`TypedUseSelectorHook`은 `react-redux`라이브러리에서 제공되며, 이를 사용하면, `useSelector`훅을 호출할 때 상태의 타입을 추론할 수 있다.

```typescript
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

const cartState = useAppSelector((state) => state.cart);
```

위와 같이 `TypedUseSelectorHook`를 사용하여 `useSelector`를 감싸면, TypeScript가 `state` 객체의 상태를 `RootState`로 추론하게 되고, `cartState` 변수에 대한 타입이 정확하게 추론되어 타입 안정성을 확보할 수 있다.

### `useParams`란?

저번 앱 제작과정에서도 `useParams` 훅을 사용했던 것 같은데, 개념정리를 하지 않었던 것 같아 이번 기회에 해보려고 한다.
<br />
<br />
`useParams`는 URL의 동적 세그먼트에서 추출한 매개변수를 얻는 데 사용된다. 예를 들어, `/users/:id`와 같은 URL을 정의하면 `id`와 같은 동적 세그먼트를 user의 고유 식별자로 사용할 수 있다. `useParams`는 이러한 동적 세그먼트에서 `id`와 같은 매개변수 값을 추출한다.
<br />
<br />
해당 앱에서는 다음과 같이 `useParams`를 사용하였다.

```typescript
import { useParams } from 'react-router-dom';

...

const { id } = useParams();
const productId = Number(id);

...

const productMatching = products.some((item) => item.id === productId);
```

**⭐참고 ⭐**  
위 코드에서 사용된 `Array.prototype.some()` 함수는 JavaScript 배열에서 사용할 수 있는 배열 메소드 중 하나로, 배열 요소 중에서 적어도 하나의 요소가 특정 요소를 만족하는지를 확인하는 데 사용된다. `Array.prototype.some()` 함수는 boolean값을 반환하며, 조건을 만족하는 요소가 하나 이상 있는 경우 `true`를, 그렇지 않은 경우 `false`를 반환한다.

### enum과 객체의 차이점

이번 앱에서 카테고리의 이름을 지정할 때 다음과 같이 `enum` 타입을 사용하였는데, 강사님께서 `enum`과 객체의 차이점에 대해 복습하라고 하셔서 다시 한번 둘의 차이점을 정리해보았다.

```typescript
export enum CategoriesName {
  All = '',
  Electronics = 'Electronics',
  Jewelry = 'Jewelery',
  MensClothing = `Men's clothing`,
  WomensClothing = `Women's clothing`,
}
```

- `enum`은 명명된 상수 값의 집합을 정의하는 데 사용되고, 주로 정적인 상수 값에 대한 집합을 나타낸다. TypeScript에서는 위와 같이 일반적으로 숫자 또는 문자열 값을 가진다.
- **객체(Object)** 는 데이터를 저장하고 조직화하는 데 사용되며, 속성과 해당 값으로 구성된다. 주로, 아래와 같이 동적 데이터 구조를 나타내는 데 사용된다.

```javascript
const person = {
  name: 'John',
  age: 30,
  city: 'New York',
};

console.log(person.name); // 'John'
```

### `typePrefix`란?

```typescript
export const fetchProduct = createAsyncThunk(
  'product/fetchProduct',
  async (id: number, thunkAPI) => {
    try {
      const response = await axios.get<IProduct>(
        `https://fakestoreapi.com/products/${id}`
      );
      console.log(response.data);
      return response.data;
    } catch (e) {
      return thunkAPI.rejectWithValue('Error loading product');
    }
  }
);
```

해당 앱에서 product를 fetch하는 함수를 구현한 부분이다. 위 코드에서 `'product/fetchProduct'`에 해당하는 부분을 `typePrefix`라고 부른다고 한다. `typePrefix`가 무엇인지 살펴보자.
<br />
<br />
위 코드에서 사용된 `createAsyncThunk` 함수는 Redux Toolkit의 일부로 제공되며 비동기 작업을 처리하는 Redux Thunk 액션 생성자를 생성하는 데 사용된다. `createAsyncThunk`를 호출할 때, 첫 번째 매겨변수로 전달하는 문자열을 `typePrefix`라 한다.
<br />
<br />

- **`typePrefix`의 역할**
  - `typePrefix`는 Redux 액션 유형의 일부를 나타낸다.
  - Redux 액션은 "type" 속성을 가지고 있으며, 일반적으로 문자열 값을 가진다. 이 "type"속성은 액션의 유형을 식별하는 데 사용된다.
  - `createAsyncThunk`로 생성된 Redux Thunk 액션은 성공, 실패 및 보류 단계에 대한 각각의 액션 유형을 생성한다. `typePrefix`는 이러한 각각의 유형을 일관적으로 생성하는 데 사용된다.

### 마무리

Redux를 사용한 첫 앱 제작을 마무리했다. 내 기준에서는 상당한 난이도를 가진 앱을 불과 몇 시간만에 만들다 보니까 차근차근 기능을 되짚어가며 따라가기 어려웠다. 그래서 다양한 기능들을 정리하려다보니 분량 조절에 실패한 것 같다 ㅎㅎ... 하지만, 이런 경험이 모여 나중에 큰 도움이 될거라 믿는다 😊
