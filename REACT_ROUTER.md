# React Router Notes

## 1) What is React Router?
- A library for **client-side routing** in React (Single Page Applications).
- Enables navigation **without full page reloads** → faster UX.
- Maps **URL → Component**.

---

## 2) Router Creation 

### Using `createBrowserRouter`
```js
import { createBrowserRouter, RouterProvider } from "react-router-dom";

const router = createBrowserRouter([
  {
    path: "/",
    element: <Layout />,   // wrapper
    children: [
      { path: "", element: <Home /> },
      { path: "about", element: <About /> }
    ]
  }
]);

function App() {
  return <RouterProvider router={router} />;
}
```

### Key points
- Routes are defined as an **array of objects**.
- Each route can include:
  - `path`
  - `element`
  - optional `children` for nesting
- Routing becomes **centralized + scalable**.

---

## 3) Layout + `<Outlet />` (dynamic content)
Use this when some part of page or the page **stays the same** and only the page content changes.

```js
import { Outlet } from "react-router-dom";

function Layout() {
  return (
    <>
      <Navbar />
      <Outlet />   {/* dynamic content */}
      <Footer />
    </>
  );
}
```

### Concept
- `<Outlet />` is a **placeholder**.
- Child route components render **inside** the outlet.

---

## 4) Navigation

### `Link` vs `NavLink` vs `<a>`

| Feature        | Link | NavLink | `<a>` |
|---------------|------|---------|------|
| Reload page   | ❌   | ❌      | ✅   |
| Active styling| ❌   | ✅      | ❌   |
| Best use case | Navigation | Navbar/menus | External links |

**NavLink active styling example:**
```js
<NavLink 
  to="/home"
  className={({ isActive }) => (isActive ? "active" : "")}
>
  Home
</NavLink>
```
NavLink can track if component associated with it is current active route or not.

---

## 5) Important Hooks

### `useNavigate()` (programmatic navigation)
```js
const navigate = useNavigate();
navigate("/home");
```

### `useLocation()` (current URL info)
```js
const location = useLocation();
```

### `useSearchParams()` (query params like `?q=abc`)
```js
const [params] = useSearchParams();
params.get("q");
```

---

## 6) Dynamic Routes
```js
{ path: "user/:id", element: <User /> }
```

Access the param:
```js
const { id } = useParams();
```

---

## 7) Protected Routes (Auth-based access)

### Approach: Wrapper Component + `<Navigate />`
```js
function ProtectedRoute({ children }) {
  const isAuth = false; // replace with real auth logic

  if (!isAuth) {
    return <Navigate to="/login" />;
  }

  return children;
}
```

Usage:
```js
{
  path: "dashboard",
  element: (
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  )
}
```

### Key idea
- If authenticated → allow access
- If not authenticated → redirect to login

---

## 8) Loaders (Data Fetching)

### What is a loader?
- Fetches data **before** rendering the route component.
- Helps reduce data rendering time in the component.
- Start API fetch when cursor hover over the ActionButton (which trigger the call).

### Example
```js
const router = createBrowserRouter([
  {
    path: "/users",
    element: <Users />,
    loader: async () => {
      const res = await fetch("/api/users");
      return res.json();
    }
  }
]);
```

### Access loader data with `useLoaderData()`
```js
import { useLoaderData } from "react-router-dom";

function Users() {
  const data = useLoaderData();
}
```

### Benefits
- Cleaner components
- Better UX (data ready earlier)
- Centralized data logic

---

## 9) Actions (Form Handling)
```js
{
  path: "/form",
  action: async ({ request }) => {
    const data = await request.formData();
    // handle submission
  }
}
```
