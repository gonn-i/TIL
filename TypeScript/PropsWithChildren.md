# 🔹 `PropsWithChildren<T>` vs `React.ReactNode`

다른 공통컴포넌트 설계에 있어서 다른 소스코드를 보던 와중 `PropsWithChildren`를 처음 보게 되었다.

자, 그래서 새롭게 알게 된 `PropsWithChildren` 이 그래서 뭘까?

## 📍 `PropsWithChildren` 는 자식 요소를 포함하는 `props`를 나타내는데에 사용되는 타입

> 정확한 형태로는 `PropsWithChildren<T>` 이다
> 영어 뜻 그대로 `children` 포함하는 `Props` 정도이며, 뭐 제네릭 타입으로 T타입의 프로퍼티에 children 까지 받을 수 있는 타입이라고 생각하면 된다.

**제네릭 타입**이다보니, 확장성 면에서 유리하다고 할 수 있다.

`PropsWithChildren` 는 `React`가 제공하는 타입 중에 하나인데, d.ts 파일에서 본 구조는 다음과 같다

```ts
type PropsWithChildren<P = {}> = P & { children?: ReactNode };
```

코드를 보면 제네릭 타입으로 P를 받으며, `ReactNode` 타입의 `children` 을 함께 정의해주고 있는데, 여기에서 주목할 점은 **`children`을 `optional`로 받고 있다는 것이다.**
따라서, `children`이 없어도 에러를 뱉지 않기 때문에 예상치 못한 결과를 낼 수 있기도 하다.

```tsx
import { PropsWithChildren } from 'react';

export const AppLayout = ({ children }: PropsWithChildren) => {
  return <S.AppLayout>{children}</S.AppLayout>;
};
```

하지만, `children`을 별도로 명시할 필요 없이 기본적으로 컴포넌트에 포함시키므로 코드가 간결해진다는 이점이 있어서 요즘 많이 사용한다.

## 📍 `ReactNode`

공통 컴포넌트 설계시 `props`로 `children`을 전달하는 형태를 고려하곤 하는데 그때마다 무의식적으로 `ReactNode`로 타입을 지정해주곤 했다.

- `React.ReactNode` 는 `React` 에서 사용할 수 있는, 거의 모든 요소를 포함하는 타입이다. 개인적으로는 **React 렌더링 가능한 타입의 any** 같은 느낌이 아닐까 싶다.

```tsx
// MobileLayout.tsx
import { ReactNode } from 'react';

interface MobileLayoutProps {
  children: ReactNode; // children을 ReactNode로 받음
  label?: string;
}

const MobileLayout = ({ children, label = 'normal' }: MobileLayoutProps) => {
  return (
    <S.Wrapper role="main" aria-label="모바일 뷰">
      <S.Layout $label={label}>{children}</S.Layout>
    </S.Wrapper>
  );
};
```

위에서는 직접적으로 `children` 의 타입을 `ReactNode` 로 지정해주고 있다.
또 형태에서도 알 수 있듯이 **`children` 를 필수로 설정할 수도 있다.**

## 그래서 `PropsWithChildren<T>` vs `React.ReactNode` 주요 차이점

- `PropsWithChildren<T>`는 `children`을 자동으로 포함 (단, 옵셔널)
- `React.ReactNode`는 `children`을 직접 명시

### 🔹 언제 사용할까?

> `PropsWithChildren<T>`: `children`을 기본으로 포함하고 싶을 때
> 이를테면, 레이아웃 컴포넌트나 텍스트 컴포넌트처럼 `children`을 전제로 하는 컴포넌트에서 유용

> `React.ReactNode`: `children`을 필수로 만들거나 특정 타입으로 지정할 때

### 🧐 깨달은 점

`PropsWithChildren<T>`는 편리하지만 `children` 전달을 필수화하지 않기 때문에 옵셔널로 사용할 경우에만 사용하고,
유연성을 위해 `children`을 꼭 넘겨줘야하는 경우에는, `React.ReactNode`를 직접 선언하는 것이 좋을 것 같다.

아님 `PropsWithChildren<T>`의 형태를 참고해서, children을 필수로 받는 타입을 만들어 사용해도 좋을 것 같다!

```ts
type StrictPropsWithChildren<P = {}> = P & { children: React.ReactNode };
```
