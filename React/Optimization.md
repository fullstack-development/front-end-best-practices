# Оптимизация React

Данный топик частично основан на [докладе](https://youtu.be/DDN9himU5PE?si=6syNK_SBnJbYaHO8) Тёмы Синюкова c HolyJS. 

1. [Введение](#1)
2. [throttle/debounce](#2)
3. [Вынос состояния](#3)
4. [useContext](#4)
5. [Render Prop](#5)
6. [Правильный условный рендеринг](#6)

<a name="1"></a>

## Введение

Для начала надо понять, что не каждый рендер в React вызывает отрисовку DOM в браузере. Например, компонент ниже будет рендериться каждую секунду и каждую секунду будет выполняться отрисовка браузером:

```tsx
export const WithChangeView = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setCount((count) => count + 1), 1000);
    return () => clearInterval(interval);
  }, []);

  return <div>{count}</div>;
};
```

А в примере ниже, компонент так же будет рендериться каждую секунду, но отрисовка в браузере произойдёт лишь один раз:

```tsx
const WithoutChangeView = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => setCount((count) => count + 1), 1000);
    return () => clearInterval(interval);
  }, []);

  return <div>Hello</div>;
};
```

Если вы столкнулись с проблемами производительности, то в первую очередь необходимо бороться с лишними рендерами, когда эти рендеры приводят к отрисовке браузером (как в первом примере). А уже потом со всеми остальными. 

<a name="2"></a>

## throttle/debounce для часто срабатываемых событий

В случае, если вы подписываетесь на событие, которое может срабатывать слишком часто (время, скролл, ресайз, движение мыши и т.д.), например несколько раз в секунду и вам это явно не нужно, воспользуйтесь throttle или debounce. В чём разница можно почитать [тут](http://xandeadx.ru/blog/javascript/956?ysclid=lz14uc5emd795136121). 

Один из примеров, это подписка на событие `scroll`, которое будет срабатывать крайне часто. И вряд ли вам вам нужно реагировать на это событие так же часто. Решение - обернуть обработчик в функцию `throttle`:

```tsx
export const ScrollWithThrottle = () => {
  const [scroll, setScroll] = useState(0);

  useEffect(() => {
    const handleScroll = () => {
      setScroll(window.scrollY);
    };
    const throttleHandleScroll = throttle(100, handleScroll);

    window.addEventListener('scroll', throttleHandleScroll);
    return () => window.removeEventListener('scroll', throttleHandleScroll);
  }, []);

  return <div>{scroll}</div>;
};
```
<a name="3"></a>

## Вынос состояния

Допустим у вас есть компонент с собственным состоянием, который при этом подключает какой-то тяжёлый `<SlowComponent />`. При каждом изменении состояния, `<SlowComponent />` будет перерендериваться:


```tsx
const Component = () => {
  const [count, setCount] = useState(0);

  return (
    <div>
      <span>{count}</span>
      <button onClick={() => setCount((count) => count + 1)}>Plus</button>
      <SlowComponent />
    </div>
  );
};
```

Решение - вынести состояние в отдельный компонент:

```tsx
const Component = () => {
  return (
    <div>
      <TriggerComponent />
      <SlowComponent />
    </div>
  );
};

const TriggerComponent = () => {
  const [count, setCount] = useState(0);

  return (
    <>
      <span>{count}</span>
      <button onClick={() => setCount((count) => count + 1)}>Plus</button>
    </>
  );
};
```

Теперь изменение состояние не приводит к перерендеру `<SlowComponent />`.

<a name="4"></a>

## useContext

При работе с `useContext` есть сразу несколько подводных камней.

1. В примере ниже, все элементы внутри `Provider` являются его прямыми потомками. Это приведёт к тому, что любое изменение состояние `theme` приведёт к перерендеру всех дочерних элементов. Даже тех, кто не использует значение из контекста.
    ```tsx
    const Context = React.createContext();

    const Component = () => {
      const [theme, setTheme] = useState("default");

      return (
        <Context.Provider value={{ theme, setTheme }}>
          <div>
            <Child />
            И еще очень много дочерних
          </div>
        </Context.Provider>
      );
    };
    ```
    Что бы этого избежать, всегда выносите `Provider` в отдельный компонент и передавайте потомков через проп `children`. 
    ```tsx
    const Context = React.createContext();
    
    const Component = () => {
      return (
        <ThemeProvider>
          <div>
            <Child />
            И еще очень много дочерних
          </div>
        </ThemeProvider>
      );
    };

    const ThemeProvider = ({ children }) => {
      const [theme, setTheme] = useState("default");

      return (
        <Context.Provider value={{ theme, setTheme }}>
          {children}
        </Context.Provider>
      );
    };
    ```
2. Не менее важная проблема - это хранение объекта в `value`. При каждом изменении состояния и перерендере `ThemeProvider`, объект будет создаваться заново и все компоненты, которые читают контекст, будут перерендериваться. Что бы этого избежать, можно обернуть значение в `useMemo`.
    ```tsx
    const Context = React.createContext();
    
    const ThemeProvider = ({ children }) => {
      const [theme, setTheme] = useState("default");
      const value = useMemo(() => ({ theme, setTheme }), [theme, setTheme]);
      
      return (
        <Context.Provider value={value}>
          {children}
        </Context.Provider>
      );
    };
    ```
3. Но это не решит всех проблем, ведь все компоненты, которые используют только `setTheme` будут перерендериваться при изменении `theme`, хотя сами они `theme` никак не используют. Это решается разнесением контекста. Один контекст для состояния, другой для сеттера.
    ```tsx
    const Context = React.createContext();

    const SetterContext = React.createContext();

    const ThemeProvider = ({ children }) => {
      const [theme, setTheme] = useState("default");

      return (
        <Context.Provider value={theme}>
          <SetterContext.Provider value={setTheme}>
            {children}
          </SetterContext.Provider>
        </Context.Provider>
      );
    };
    ```
    Вот теперь всё хорошо, компоненты использующие `setTheme` не будут перерендериваться при изменении `theme`.

<a name="5"></a>

## Использование Render Prop

У вас есть компонент, подключащий множество других компонентов и содержащий логику, которая заставляет этот компонент часто перерендериваться. Как итог, все его потомки будут тоже часто перерендериваться:

```tsx
const Page = () => {
  const [isHovered, setIsHovered] = useState(false);
  const [scroll, setScroll] = useState();

  // Logic

  return (
    <div
      onScroll={setScroll}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <Header scroll={scroll} />
      <Content />
      <Footer />
    </div>
  );
};
```

Решение - воспользоваться Render Prop. Вы можете в качестве пропсов передать в компонент `Page` другие компоненты, например `Content`. Если вам нужно при этом, что бы компонент получил состояние, как в случае с `Header`, воспользуйтесь функцией: 

```tsx
const Layout = ({ renderTop, center, bottom }) => {
  const [isHovered, setIsHovered] = useState(false);
  const [scroll, setScroll] = useState();

  // Logic
  
  return (
    <div
      onScroll={setScroll}
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      <div>{renderTop(scroll)}</div>
      <div>{center}</div>
      <div>{bottom}</div>
    </div>
  );
};

const Page = () => {
  return (
    <Layout
      renderTop={(scroll) => <Header scroll={scroll} />}
      center={<Content />}
      bottom={<Footer />}
    />
  );
};
```

Теперь компоненты `Header`, `Content` и `Footer` не являются прямыми потомками в `Layout`, и не будут перерендериваться при изменении состояний в `Layout`.

<a name="6"></a>

## Правильный условный рендеринг

Допустим, у вас есть компонент, который должен отображаться только авторизованным пользователям.

```tsx
const AuthorizedUser = () => {
  // здесь много хуков и логики

  const isAuthorized = useIsAuthorized();

  if (!isAuthorized) {
    return null;
  }
  return <>...</>;
};
```

Проблема в том, что все хуки и логика, находящиеся перед `if` всегда выполнятся. Решение - проверку на рендер компонента `AuthorizedUser` делать в родителе. Для переиспользования вы можете вынести проверку в HOC:

```tsx
const withAuthorize = ({ Authorized, UnAuthorized }) => {
  const Component = function WithAuthorizeComponent(authProps, unAuthProps) {
    const isAuthorized = useIsAuthorized();
    return isAuthorized ? (
      <AuthorizedUser {...authProps} />
    ) : (
      <UnAuthorizedUser {...unAuthProps} />
      );
  };

  return Component;
};
```