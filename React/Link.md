# 🔹 Link

목록 페이지에서 카드 컴포넌트를 눌렀을때 세부 페이지로 이동 시키는 경우,
`nav` 에서 로그인이나 회원가입 등 특정 페이지로 이동 시키는 경우 등 그동안
한 페이지에서 -> 다음 페이지로 이동시킬때 `Link` 태그와 `useNavigate` 를 이용해왔다.

하지만 내부 동작 및 속성에 대해 자세히 알고 쓰지 못했기에 더 공부해보고자 한다.

## 📍 `Link` 태그에 대해서

`react-router-dom` 에서 제공하는 태그로, 웹 내에서 페이지간에 네비게이션을 처리하는 컴포넌트이다. 기본적으로 `<a>` 태그와 유사한 역할을 한다. 정확히는 `<a>` 태그의 성질을 확장한 구조인데 이는 후술하도록 하겠다. (HTMLAnchorElement 와 관련하여)

**그럼 `<a>` 를 쓰지 왜 `Link`를 따로 만들었나?**
`<a>` 는 전체 페이지를 리랜더 하는 특성이 있는 반면, `Link`태그는 SPA 방식으로 페이지를 전환하는데 사용한다. 특히, 이때 페이지 새로고침없이 원하는 content 를 동적으로 변경할 수 있다는 점이 특징이다.

## 📍 `Link` 그 내부를 살펴보자

```ts
declare const Link: React.ForwardRefExoticComponent<LinkProps & React.RefAttributes<HTMLAnchorElement>>;
```

타입 선언을 뜯어보면, `Link` 는 `LinkProps`와 `React.RefAttributes<HTMLAnchorElement>` 타입을 받고 있는 걸 알 수 있다.

#### 가장 먼저 `LinkProps` 먼저부터 보자

```ts
interface LinkProps extends Omit<React.AnchorHTMLAttributes<HTMLAnchorElement>, 'href'> {
  // ... 일부 내용 생략
  state?: any;

  preventScrollReset?: boolean;

  relative?: RelativeRoutingType;

  to: To;

  viewTransition?: boolean;
}
```

`LinkProps` 은 `React.AnchorHTMLAttributes<HTMLAnchorElement>` 인터페이스를 상속받고 있다. 그래서 위에서 `<a>` 태그의 속성을 가지고 있다고 말한 이유이다. 근데 `Omit` 으로 `href` 속성을 제외하고 있는데, 그 이유는 `Link` 태그의 실제 사용을 보면 알 수 있다.

`<Link to="/" />` 다음과 같은 방식으로 Link는 페이지 라우팅을 하는데, 이떄 주목할 점은 a 태그처러 `href` 로 이동할 주소를 전달하는 것이 라니라는 점이다. `Link`는 `to` 속성으로 링크 처리를 위해 `href`를 제외하고 가져오는 것이다.

**다음으로 속성 하나씩 보기**

1️⃣ `discover` : 링크 경로를 언제 발견할지 설정하는 속성

```js
// (default) 링크가 렌더링될 때 경로를 발견
// 미리 경로를 탐지해서 필요한 리소스를 로드함
<Link discover="render" />

// 링크가 클릭되었을 때 경로를 발견
<Link discover="none" />
```

2️⃣ `prefetch` : 링크와 관련된 데이터를 미리 불러오는 방법

```js
// (기본값): 미리 불러오지 않음
<Link prefetch="none" />

// 사용자가 링크에 hover하거나 focus할 때 미리 불러옴
<Link prefetch="intent" />

// 링크가 렌더링될 때 미리 불러옴
<Link prefetch="render" />

// 링크가 화면에 보일 때 미리 불러옴
<Link prefetch="viewport" />
```

3️⃣ `reloadDocument`: `boolean` 값으로, 링크 클릭 시 문서를 새로 고침하도록 지정
-> 기본으로는 리로드없이 처리되지만, true 로 설정하면 페이지가 깜빡인다.

4️⃣ `replace`: 히스토리 스택에서 현재 항목을 교체하도록 설정

5️⃣ `state` : 링크로 이동할 때 추가적인 상태를 전달할 수 있다. 사실 전역 상태 관리보다는 `useLocation` 로 `state` 값을 많이 사용한 나로서는 굉장히 반가운 속성이다.

```js
<DropDown.Item onClick={() => navigate(`/community/modify/${boardId}`, { state: { post } })}>수정하기</DropDown.Item>
```

6️⃣ `preventScrollReset`: `boolean` 타입으로, 링크 클릭 후 스크롤 위치를 초기화하는 것을 방지한다. 원래 기본값으로 링크 클릭시 페이지 최상단으로 스크롤되지만, 해당 속성이 `true` 이면 이전 스크롤 위치가 유지된다.

7️⃣ `relative` : 상대 경로를 처리하는 방식

```js
// (기본값) 라우트 패턴에 대해 상대 경로를 처리
<Link to=".." relative="route" />

// 실제 URL 경로에 대해서 상대경로 처리
<Link to=".." relative="path" />
```

8️⃣ `to`: 링크가 이동할 경로를 지정하는 필수 속성! 문자열로만 경로를 지정했는데, 객체 형태로도 가능하다고 한다.

```js
<Link to={`/detail/${id}`} />
<Link to={{ pathname: "/some/path", search: "?query=string", hash: "#hash" }} />

```

9️⃣ `viewTransition`: `View Transition`을 활성화하여 페이지 이동 시 전환 애니메이션을 적용할 수 있다.

#### React.RefAttributes<HTMLAnchorElement>> 는 뭘까

`HTMLAnchorElement` 는 `<a>` 를 의미하기 때문에, `Link` 컴포넌트가 `ref`를 받아서 내부 `<a>` 태그에 접근할 수 있다는 것이다.

그래서 다음과 같은 코드 작성이 가능하다
(v6 이전에는 `innerRef` 로 받았어야 했는데 수정되었다고 한다.)

```tsx
const App = () => {
  const linkRef = useRef<HTMLAnchorElement>(null);

  return <Link to="/about" ref={linkRef} />;
};
```

사실 개발을 하면서 `<Link>`에 `ref`를 전달할 일이 많지는 않다고 생각한다. 하지만 이를 활용할 수 있다는 걸 알면, 특정 상황에서 자동 포커스, 클릭 이벤트 트리거, 스타일 조작 등 다양한 방식으로 응용할 수 있을 것이라고 생각된다!
