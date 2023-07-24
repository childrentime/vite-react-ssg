# Vite React SSG

Static-site generation for React on Vite.

[![NPM version](https://img.shields.io/npm/v/vite-react-ssg?color=a1b858&label=)](https://www.npmjs.com/package/vite-react-ssg)

## Install

> **This library requires Node.js version >= 17**
> or `Request` is available
<pre>
<b>npm i -D vite-react-ssg</b> <em>react-router-dom</em>
</pre>

```diff
// package.json
{
  "scripts": {
    "dev": "vite",
-   "build": "vite build"
+   "build": "vite-react-ssg build"

    // OR if you want to use another vite config file
+   "build": "vite-react-ssg build -c another-vite.config.ts"
  }
}
```

```ts
// src/main.ts
import { ViteReactSSG } from 'vite-react-ssg'
import routes from './App.tsx'

export const createRoot = ViteReactSSG(
  // react-router-dom data routes
  { routes },
  // function to have custom setups
  ({ router, routes, isClient, initialState }) => {
    // do something.
  },
)
```

```tsx
// src/App.tsx
import React from 'react'
import type { RouteRecord } from 'vite-react-ssg'
import './App.css'

const pages = import.meta.glob<any>('./pages/**/*.tsx')

const children: RouteRecord[] = Object.entries(pages).map(([filepath, component]) => {
  let path = filepath.split('/pages')[1]
  path = path.split('.')[0].replace('index', '')
  const entry = `src${filepath.slice(1)}`

  if (path.endsWith('/')) {
    return {
      index: true,
      Component: React.lazy(component),
      entry,
    }
  }
  return {
    path,
    Component: React.lazy(component),
    // Used to obtain static resources through manifest
    entry,
  }
})

const Layout = React.lazy(() => import('./Layout'))
export const routes: RouteRecord[] = [
  {
    path: '/',
    element: <Layout />,
    children,
    // Used to obtain static resources through manifest
    entry: 'src/Layout.tsx',
  },
]
```

## Critical CSS

Vite SSG has built-in support for generating [Critical CSS](https://web.dev/extract-critical-css/) inlined in the HTML via the [`critters`](https://github.com/GoogleChromeLabs/critters) package.
Install it with:

```bash
npm i -D critters
```

Critical CSS generation will automatically be enabled for you.

To configure `critters`, pass [its options](https://github.com/GoogleChromeLabs/critters#usage) into `ssgOptions.crittersOptions` in `vite.config.ts`:

```ts
// vite.config.ts
export default defineConfig({
  ssgOptions: {
    crittersOptions: {
      // E.g., change the preload strategy
      preload: 'media',
      // Other options: https://github.com/GoogleChromeLabs/critters#usage
    },
  },
})
```

## Configuration

You can pass options to Vite SSG in the `ssgOptions` field of your `vite.config.js`

```js
// vite.config.js

export default {
  plugins: [],
  ssgOptions: {
    script: 'async',
  },
}
```

See [src/types.ts](./src/types.ts). for more options available.

### Custom Routes to Render

You can use the `includedRoutes` hook to include or exclude route paths to render, or even provide some completely custom ones.

```js
// vite.config.js

export default {
  plugins: [],
  ssgOptions: {
    includedRoutes(paths, routes) {
      // exclude all the route paths that contains 'foo'
      return paths.filter(i => !i.includes('foo'))
    },
  },
}
```
```js
// vite.config.js

export default {
  plugins: [],
  ssgOptions: {
    includedRoutes(paths, routes) {
      // use original route records
      return routes.flatMap((route) => {
        return route.name === 'Blog'
          ? myBlogSlugs.map(slug => `/blog/${slug}`)
          : route.path
      })
    },
  },
}
```

## Roadmap

- [x] Preload assets
- [ ] SSR under dev
- [ ] Initial State
- [ ] Document head
- [ ] More Client components, such as `<ClientOnly />`

## License

[MIT](./LICENSE) License © 2023 [Riri](https://github.com/Daydreamer-riri)