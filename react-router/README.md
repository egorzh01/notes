# React Router v7 ([Источник](https://www.robinwieruch.de/react-router/))

Установка:

```bash
npm install react-router
```

В самом начале нужно импортировать `BrowserRouter` в основной файл, где береться тег с атрибутом `id="root"`, и обернуть им компонент нашего приложения:

```jsx
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router";
import App from "./App.tsx";

createRoot(document.getElementById("root")!).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

Этот компонент использует **\*HTML5 History API** и предоставляет компонентам внутри `App` доступ к функциям маршрутизации (например, useNavigate, useLocation, Route, Link) через контекст React Router, следит за изменениями URL (например, при клике по ссылке или использовании кнопок "вперед/назад" в браузере) и обновляет интерфейс приложения в соответствии с текущим маршрутом.

---

**\*HTML5 History API** — это интерфейс браузера, который позволяет JavaScript управлять историей навигации в веб-приложении. Он предоставляет методы и свойства для работы с историей браузера, что особенно полезно для создания одностраничных приложений (SPA), где изменение URL и содержимого страницы происходит без полной перезагрузки.

`window.history` - Объект, через который осуществляется доступ к истории браузера.

---

## Роутинг(Маршрутизация) и Навигация

**Роутинг** - механизм для определения, какой компонент будет отрисован в зависимости от пути.

**Навигация** - механизм для перемещения между путями.

При нажатии на ссылку для перехода по другому адресу вы используете механизм _навигации_, в момент перехода механизм _роутинга_ ищет соответствующий этому пути компонент и отрисовывает его.

---

Для начала мы реализуем навигацию в нашем компоненте `App`. Для этого мы используем компонент `Link` из _react-router_, при нажатии на который мы можем изменять наш путь.

```jsx
import { Link } from "react-router";

const App = () => {
  return (
    <>
      <h1>React Router</h1>

      <Navigation />
    </>
  );
};

const Navigation = () => {
  return (
    <nav
      style={{
        borderBottom: "solid 1px",
        paddingBottom: "1rem",
      }}
    >
      <Link to="/home">Home</Link> // При нажатии перенаправит нас на /home
      <Link to="/users">Users</Link> // При нажатии перенаправит нас на /users
    </nav>
  );
};

export default App;
```

---

Далее, используя компоненты Routes и Route, мы сопоставляем текущий путь и компонент который хотим отрисовать, когда путь будет ему соответствовать.

```jsx
import { Routes, Route, Link } from "react-router";

const Home = () => {
  return (
    <main style={{ padding: "1rem 0" }}>
      <h2>Home</h2>
    </main>
  );
};

const Users = () => {
  return (
    <main style={{ padding: "1rem 0" }}>
      <h2>Users</h2>
    </main>
  );
};

const App = () => {
  return (
    <>
      <h1>React Router</h1>

      <Navigation />

      <Routes>
        <Route path="home" element={<Home />} /> // Если путь равен /home отрисовываем компонент Home
        <Route path="users" element={<Users />} /> // Если путь равен /users отрисовываем компонент Users
      </Routes>
    </>
  );
};
```

`Routes` — это контейнер, который анализирует текущий URL и выбирает один подходящий `Route` для рендеринга на основе его path. Без `Routes` React Router не сможет сопоставить URL с маршрутами, так как именно `Routes` отвечает за механизм маршрутизации.

- Без `Routes` каждый `Route` будет рендериться независимо, что может привести к отображению нескольких компонентов одновременно или к некорректной работе маршрутизации.
- `Routes` гарантирует, что только один `Route` (или вложенные маршруты, _о них позже_) будет отображен для текущего URL.

## Layout Routes

В прошлом блоке кода, можно заметить что компоненты `Home` и `Users` используют одну и ту жа обертку `main`. Чтобы избежать дублирования, мы могли бы вынести это.

```tsx
const Home = () => {
  return (
    // Убираем main и оставляем пустой тег
    <>
      <h2>Home</h2>
    </>
  );
};

const Users = () => {
  return (
    // Убираем main и оставляем пустой тег
    <>
      <h2>Users</h2>
    </>
  );
};

type LayoutProps = {
  children: React.ReactNode;
};

// Создаем обертку
const Layout = ({ children }) => {
  return <main style={{ padding: "1rem 0" }}>{children}</main>;
};
```

Далее нам неоьбходимо добавить обертку в компоненте `App`.

```tsx
const App = () => {
  return (
    <>
      ...
      <Routes>
        // Добавляем обертку
        <Layout>
          <Route path="home" element={<Home />} />
          <Route path="users" element={<Users />} />
        </Layout>
      </Routes>
    </>
  );
};
```

Но вы можете увидеть что такой подход не разрешен в react-router и вы получите исключение в котором говориться `Uncaught Error: [Layout] is not a <Route> component. All component children of <Routes> must be a <Route> or <React.Fragment>.`(Все дочерние компоненты `<Routes>` должны быть `<Route>` или `<React.Fragment>`).

Обычным выходом из этой ситации будет использование компонента Layout для каждого отдельного компонента:

```tsx
const App = () => {
  return (
    <>
      ...
      <Routes>
        <Route
          path="home"
          element={
            <Layout>
              <Home />
            </Layout>
          }
        />
        <Route
          path="users"
          element={
            <Layout>
              <Users />
            </Layout>
          }
        />
      </Routes>
    </>
  );
};
```

Однако это добавляет нежелательную избыточность в наше приложение. Поэтому вместо дублиролвания компонента `Layout` мы будем использовать так называемый `Layout Route` , который фактически не является путем для маршрутизации, а является всего лишь способом обернуть каждый дочерний компонент в группе Routes одинаковой оберткой:

```tsx
const App = () => {
  return (
    <>
      ...
      <Routes>
        // Оборачиваем наши Route
        <Route element={<Layout />}>
          <Route path="home" element={<Home />} />
          <Route path="users" element={<Users />} />
        </Route>
      </Routes>
    </>
  );
};
```

Как вы видите, можно вкладывать компоненты `Route` в другой компонент `Route` — тогда первые становятся так называемыми `Nested Routes`.

Теперь нам нужно немного изменить наш компонент `Layout` таким способом, используя компонент `Outlet` из _react-router_:

```tsx
import { Routes, Route, Outlet, Link } from 'react-router';

...

const Layout = () => {
  return (
    <main style={{ padding: '1rem 0' }}>
      <Outlet />
    </main>
  );
};
```

По сути компонент `Outlet` в компоненте `Layout` вставляет дочерний элемент соответсвующий текущему пути (`Home` при пути `/home` или `Users` при пути `/users`) вместо себя. В конце концов, использование `Layout Route` помогает вам дать каждому компоненту Route в группе один и тот же макет (например, стиль с CSS, структура с HTML).

## Активные ссылки

Теперь мы можем переместить наш заголовок и компонент с навигацией в компонент `Layout`.

Кроме того давайте обновим наши `Link` на `NavLink`, которые как и Route отслеживают наш текущий путь и позволяют получить нам аргумент `isActive`, который мы можем использовать, передавая в _style_, _className_ или _children_ функцию, которая принимает эти аргументы и работает с ними. Заметьте что обычные компоненты _react_ не могут принимать функции в их _style_, _className_ и _children_ в качестве аргументов.

---

_children_ — это всё, что вложено между JSX-тегами, например <Parent>...</Parent>. Компонент Parent — это обычная функция, которая получает props, включая children, и возвращает JSX. JSX — это синтаксический сахар над React.createElement(...), и React сам вызывает эти компоненты при рендеринге.

---

```tsx
import {
  ...
  NavLink,
} from 'react-router';

const App = () => {
  return (
    <Routes>
      <Route element={<Layout />}>
        <Route path="home" element={<Home />} />
        <Route path="users" element={<Users />} />
      </Route>
    </Routes>
  );
};

const Layout = () => {
  // Функция которая принимает isActive как аргумент и в зависимости от его значения вохвращает обьект со стилями
  const styleFn = ({ isActive }: NavLinkRenderProps) => ({
    fontWeight: isActive ? "bold" : "normal",
  });

  return (
    <>
      <h1>React Router</h1>

      <nav
        style={{
          borderBottom: "solid 1px",
          paddingBottom: "1rem",
        }}
      >
        <NavLink to="/home" style={styleFn}>Home</NavLink>
        <NavLink to="/users" style={styleFn}>Users</NavLink>
      </nav>

      <main style={{ padding: "1rem 0" }}>
        <Outlet />
      </main>
    </>
  );
};
```

## Индексные маршруты

Можно заметить что у нас есть пути `/home` и `/users`. Но у нас нет _индексного маршрута_ `/`.
