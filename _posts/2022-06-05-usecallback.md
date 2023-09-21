---
layout: single
title: "React Hooks: useMemo, useCallback"
categories: [React, Next.js, Basic]
tag: [React, useCallback, useMemo]
toc: true
author_profile: false
sidebar:
    nav: "docs"

---

# useMemo와 useCallback을 배우기 전에 알아야 하는 것

> 1. 함수형 컴포넌트는 그냥 함수다. **다시 한 번 강조하자면 함수형 컴포넌트는 단지 jsx를 반환하는 함수이다.**
> 2. 컴포넌트가 렌더링 된다는 것은 누군가가 그 함수(컴포넌트)를 호출하여서 실행되는 것을 말한다. **함수가 실행될 때마다 내부에 선언되어 있던 표현식(변수, 또다른 함수 등)도 매번 다시 선언되어 사용된다.**
> 3. 컴포넌트는 **자신의 state가 변경되거나, 부모에게서 받는 props가 변경되었을 때마다** 리렌더링 된다. (심지어 하위 컴포넌트에 최적화 설정을 해주지 않으면 부모에게서 받는 props가 변경되지 않았더라도 리렌더링 되는게 기본이다. )



# useMemo

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

useMemo는 [메모이제이션된](https://ko.wikipedia.org/wiki/메모이제이션) 값을 반환합니다.

> **메모이제이션**(memoization)은 [컴퓨터](https://ko.wikipedia.org/wiki/컴퓨터) [프로그램](https://ko.wikipedia.org/wiki/컴퓨터_프로그램)이 동일한 계산을 반복해야 할 때, 이전에 계산한 값을 메모리에 저장함으로써 동일한 계산의 반복 수행을 제거하여 프로그램 실행 속도를 빠르게 하는 기술이다. [동적 계획법](https://ko.wikipedia.org/wiki/동적_계획법)의 핵심이 되는 기술이다. **메모이제이션**이라고도 한다.

`useMemo`로 전달된 함수는 렌더링 중에 실행된다는 것을 기억하세요. 통상적으로 렌더링 중에는 하지 않는 것을 이 함수 내에서 하지 마세요. 예를 들어, 사이드 이펙트(side effects)는 `useEffect`에서 하는 일이지 `useMemo`에서 하는 일이 아닙니다.

**메모리제이션된 값을 반환한다라는 문장이 핵심이다.** 다음과 같은 상황을 생각해보자. 자식 컴포넌트는 부모 컴포넌트로부터 a와 b라는 두 개의 props를 전달 받는다. 자식 컴포넌트에서는 a와 b를 전달 받으면 서로 다른 함수로 각각의 값을 가공(또는 계산)한 새로운 값을 보여주는 역할을 한다. 자식 컴포넌트는 props로 넘겨 받는 인자가 하나라도 변경될 때마다 렌더링되는데, props.a 만 변경 되었을 때 이전과 같은 값인 props.b도 다시 함수를 호출해서 재계산해야 한다. 여기서 의문점이 든다. 그냥 props.b는 이전에 계산된 값을 쓰면 되는데 props.a가 변경되어서 props.b도 리렌더링이 된다. 즉, 이것은 효율적이지 않습니다. 컴포넌트에서 리 렌더링이 필요한 상황에서만 해주도록 설정을 할 수 있는데 이때 사용하는 함수가 바로 `React.memo` 함수입니다.

**`useMemo`는 성능 최적화를 위해 사용할 수는 있지만 의미상으로 보장이 있다고 생각하지는 마세요.** 가까운 미래에 React에서는, 이전 메모이제이션된 값들의 일부를 “잊어버리고” 다음 렌더링 시에 그것들을 재계산하는 방향을 택할지도 모르겠습니다. 예를 들면, 오프스크린 컴포넌트의 메모리를 해제하는 등이 있을 수 있습니다. `useMemo`를 사용하지 않고도 동작할 수 있도록 코드를 작성하고 그것을 추가하여 성능을 최적화하세요.



## useMemo 임포트 방법

`useMemo` 함수는 React v16.8부터 기본적으로 내장되어 있는 Hooks 중 하나입니다. 따라서, 프로젝트에 `react` 패키지만 설치되어 있다면 named import를 통해 바로 임포트에서 사용할 수 있습니다.

```jsx
import React, { useMemo } from "react";
```

## [예제]

useMemo 를 설명할 수 있는 [간단한 예제](https://codesandbox.io/s/upbeat-margulis-v8k51?file=/src/Info.js)를 만들어 보았다. App 컴포넌트는 Info 컴포넌트에게 사용자로부터 입력 받은 color와 movie값을 props로 넘겨주고, Info 컴포넌트는 전달 받은 color와 movie를 적절한 한글로 바꾸어서 문장으로 보여준다.

```jsx
import Info from "./Info";

const App = () => {
  const [color, setColor] = useState("");
  const [movie, setMovie] = useState("");

  const onChangeHandler = e => {
    if (e.target.id === "color") setColor(e.target.value);
    else setMovie(e.target.value);
  };

  return (
    <div className="App">
      <div>
        <label>
          What is your favorite color of rainbow ?
          <input id="color" value={color} onChange={onChangeHandler} />
        </label>
      </div>
      <div>
        What is your favorite movie among these ?
        <label>
          <input
            type="radio"
            name="movie"
            value="Marriage Story"
            onChange={onChangeHandler}
          />
          Marriage Story
        </label>
        <label>
          <input
            type="radio"
            name="movie"
            value="The Fast And The Furious"
            onChange={onChangeHandler}
          />
          The Fast And The Furious
        </label>
        <label>
          <input
            type="radio"
            name="movie"
            value="Avengers"
            onChange={onChangeHandler}
          />
          Avengers
        </label>
      </div>
      <Info color={color} movie={movie} />
    </div>
  );
};

export default App;
```

```jsx
// Info.js

const getColorKor = color => {
    console.log("getColorKor");
    switch (color) {
      case "red":
        return "빨강";
      case "orange":
        return "주황";
      case "yellow":
        return "노랑";
      case "green":
        return "초록";
      case "blue":
        return "파랑";
      case "navy":
        return "남";
      case "purple":
        return "보라";
      default:
        return "레인보우";
    }
  };

  const getMovieGenreKor = movie => {
    console.log("getMovieGenreKor");
    switch (movie) {
      case "Marriage Story":
        return "드라마";
      case "The Fast And The Furious":
        return "액션";
      case "Avengers":
        return "슈퍼히어로";
      default:
        return "아직 잘 모름";
    }
  };


const Info = ({ color, movie }) => {
  const colorKor = getColorKor(color);
  const movieGenreKor = getMovieGenreKor(movie);

  return (
    <div className="info-wrapper">
      제가 가장 좋아하는 색은 {colorKor} 이고, <br />
      즐겨보는 영화 장르는 {movieGenreKor} 입니다.
    </div>
  );
};

export default Info;
```

![image](https://user-images.githubusercontent.com/18614517/80869400-2bdf4380-8cdb-11ea-8cad-373429ca8962.png)

(예제의 실행 화면)



App 컴포넌트의 입력창에서 color값만 바꾸어도 getColorKor, getMovieGenreKor 두 함수가 모두 실행되고 movie 값만 바꾸어도 마찬가지로 두 함수가 모두 실행된다. **useMemo**를 import 해서 Info 컴포넌트의 코드에서 colorKor과 movieGenroKor을 계산하는 부분을 아래와 같이 바꿔보자.

```jsx
import React, { useMemo } from "react";

const colorKor = useMemo(() => getColorKor(color), [color]);
const movieGenreKor = useMemo(() => getMovieGenreKor(movie), [movie]);
```

useMemo를 사용하면 의존성 배열에 넘겨준 값이 변경되었을 때만 메모리제이션된 값을 다시 계산한다.

예제 코드를 직접 변경하여 color값이 바뀔 때는 getColorKor함수만, movie값이 바뀔 때는 getMovieGenreKor함수만 호출되는 것을 확인할 수 있다. **지금 처럼 재계산하는 함수가 아주 간단하다면 성능상의 차이는 아주 미미하겠지만 만약 재계산하는 로직이 복잡하다면 불필요하게 비싼 계산을 하는 것을 막을 수 있다.** (공식 문서에서도 useMemo에 넘겨주는 콜백 함수의 이름이 computeExpensiveValue() 라고 되어 있다는 것을 주목하자)

지금 나는 부모 컴포넌트에게 props를 전달 받는 상황으로 설명했지만, 하나의 컴포넌트에서 두 개 이상의 state가 있을 때도 활용할 수 있다.





# useCallback

```jsx
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

[메모이제이션된](https://ko.wikipedia.org/wiki/메모이제이션) 콜백을 반환합니다.

> 인라인 콜백과 그것의 의존성 값의 배열을 전달하세요. `useCallback`은 콜백의 메모이제이션된 버전을 반환할 것입니다. 그 메모이제이션된 버전은 콜백의 의존성이 변경되었을 때에만 변경됩니다. 이것은, 불필요한 렌더링을 방지하기 위해 (예로 `shouldComponentUpdate`를 사용하여) 참조의 동일성에 의존적인 최적화된 자식 컴포넌트에 콜백을 전달할 때 유용합니다.

**메모리제이션된 함수를 반환한다라는 문장이 핵심이다.** 서론에서 useMemo와 useCallback을 배우기 전에 알아야 하는 것들 중 두 번째로 *컴포넌트가 렌더링 될 때마다 내부에 선언되어 있던 표현식(변수, 또다른 함수 등)도 매번 다시 선언되어 사용된다.* 라고 이야기했다. useMemo를 설명하고 있는 같은 예제에서 App.js의 onChangeHandler 함수는 내부의 color, movie 상태값이 변경될 때마다 재선언된다는 것을 의미한다. 하지만 onChangeHandler 함수는 파라미터로 전달받은 이벤트 객체(e)의 target.id 값에 따라 setState를 실행해주기만 하면 되기 때문에, 첫 마운트 될 때 한 번만 선언하고 재사용하면 되지 않을까?

## useCallback 임포트 방법

```jsx
import React, { useCallback } from "react";
```

```jsx
// App.js

import React, { useState, useCallback } from "react";

const onChangeHandler = useCallback(e => {
    if (e.target.id === "color") setColor(e.target.value);
    else setMovie(e.target.value);
  }, []);
```

App.js에서 useCallback을 import 하고 onChangeHandler함수의 선언부를 위와 같이 바꿔보자. 첫 마운트 될 때만 메모리에 할당되었는지 아닌지 확인하기는 어렵겠지만 위와 같이 사용한다. 예시와 같은 이벤트 핸들러 함수나 api를 요청하는 함수를 주로 useCallback 으로 선언하는 코드를 꽤 많이 보았다.

하지만, 비싼 계산이 아니라면 useMemo사용을 권장하지 않는 것처럼 이정도 수준의 함수 재선언을 막기 위해 useCallback을 사용하는 것도 크게 의미있어 보이지는 않는다. 다시 공식 문서를 꼼꼼히 읽어보면…

> This is useful when passing callbacks to optimized child components that rely on reference equality to prevent unnecessary renders (e.g. shouldComponentUpdate).

와 같은 말이 나온다. 만약 하위 컴포넌트가 **React.memo()** 같은 것으로 최적화 되어 있고 그 하위 컴포넌트에게 callback 함수를 props로 넘길 때, 상위 컴포넌트에서 useCallback 으로 함수를 선언하는 것이 유용하다라는 의미이다. 함수가 매번 재선언되면 하위 컴포넌트는 넘겨 받은 함수가 달라졌다고 인식하기 때문이다.

- React.memo()로 함수형 컴포넌트 자체를 감싸면 넘겨 받는 props가 변경되지 않았을 때는 상위 컴포넌트가 메모리제이션된 함수형 컴포넌트(이전에 렌더링된 결과)를 사용하게 된다.
- 함수는 오로지 자기 자신만이 동일하기 때문에 상위 컴포넌트에서 callback 함수를 (같은 함수이더라도) 재선언한다면 props로 callback 함수를 넘겨 받는 하위 컴포넌트 입장에서는 props가 변경 되었다고 인식한다.



참조: [(https://leehwarang.github.io/docs/tech/2020-05-02-useMemo&useCallback.html)]()

참조: [(https://ko.reactjs.org/docs/hooks-reference.html)]()