![React Keycloak](/art/react-keycloak-logo.png?raw=true 'React Keycloak Logo')

# React Keycloak <!-- omit in toc -->

> [Razzle](https://github.com/jaredpalmer/razzle) bindings for [Keycloak](https://www.keycloak.org/)

[![NPM (scoped)](https://img.shields.io/npm/v/@react-keycloak/razzle?label=npm%20%7C%20razzle)](https://www.npmjs.com/package/@react-keycloak/razzle)

[![License](https://img.shields.io/github/license/react-keycloak/react-keycloak.svg)](https://github.com/react-keycloak/react-keycloak/blob/master/LICENSE.md)
[![lerna](https://img.shields.io/badge/maintained%20with-lerna-cc00ff.svg)](https://lerna.js.org/)
[![GitHub contributors](https://img.shields.io/github/contributors/react-keycloak/react-keycloak)](https://github.com/react-keycloak/react-keycloak/graphs/contributors)
[![Github Issues](https://img.shields.io/github/issues/react-keycloak/react-keycloak.svg)](https://github.com/react-keycloak/react-keycloak/issues)

[![Gitter](https://img.shields.io/gitter/room/react-keycloak/community)](https://gitter.im/react-keycloak/community)

---

## Table of Contents <!-- omit in toc -->

- [Install](#install)
- [Support](#support)
- [Getting Started](#getting-started)
  - [Setup Razzle App](#setup-razzle-app)
  - [HOC Usage](#hoc-usage)
  - [Hook Usage](#hook-usage)
- [Examples](#examples)
- [Contributing](#contributing)
- [License](#license)

---

## Install

React Keycloak requires:

- React **16.0** or later
- razzle **3** or later
- `keycloak-js` **9.0.2** or later

```shell
yarn add @react-keycloak/razzle
```

or

```shell
npm install --save @react-keycloak/razzle
```

## Support

| version | keycloak-js version |
| ------- | ------------------- |
| v2.0.0+ | 9.0.2+              |
| v1.x    | >=8.0.2 <9.0.2      |

## Getting Started

### Setup Razzle App

> **N.B:** This setup requires you to install [`cookie-parser`](https://github.com/expressjs/cookie-parser) middleware.

Edit your app `server.js` as follow

```js
...

import { ServerPersistors, SSRKeycloakProvider } from '@react-keycloak/razzle'

// Create a function to retrieve Keycloak configuration parameters -- 'see examples/razzle-app'
import { getKeycloakConfig } from './utils'

const assets = require(process.env.RAZZLE_ASSETS_MANIFEST)

const server = express()
server
  .disable('x-powered-by')
  .use(express.static(process.env.RAZZLE_PUBLIC_DIR))
  .use(cookieParser()) // 1. Add cookieParser Express middleware
  .get('/*', (req, res) => {
    const context = {}

    // 2. Create an instance of ServerPersistors.ExpressCookies passing the current request
    const cookiePersistor = ServerPersistors.ExpressCookies(req)

    // 3. Wrap the App inside SSRKeycloakProvider
    const markup = renderToString(
      <SSRKeycloakProvider
        keycloakConfig={getKeycloakConfig()}
        persistor={cookiePersistor}
      >
        <StaticRouter context={context} location={req.url}>
          <App />
        </StaticRouter>
      </SSRKeycloakProvider>
    )
    ...
  })

...
```

Edit your `client.js` as follow

```js
import { ClientPersistors, SSRKeycloakProvider } from '@react-keycloak/razzle'

// Create a function to retrieve Keycloak configuration parameters -- 'see examples/razzle-app'
import { getKeycloakConfig } from './utils'

// 1. Wrap the App inside SSRKeycloakProvider
hydrate(
  <SSRKeycloakProvider
    keycloakConfig={getKeycloakConfig()}
    persistor={ClientPersistors.Cookies}
  >
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </SSRKeycloakProvider>,
  document.getElementById('root')
)
...
```

### HOC Usage

When a page requires access to `Keycloak`, wrap it inside the `withKeycloak` HOC.

```jsx
...

import { withKeycloak } from '@react-keycloak/razzle'

const Home = ({ keycloak, keycloakInitialized: initialized, isServer }) => {
  console.log('Is running on server:', isServer)
  console.log('Keycloak is initialized:', initialized)

  return (
    <div className="Home">
      <div className="Home-header">
        <img src={logo} className="Home-logo" alt="logo" />
        <h2>Welcome to Razzle</h2>
      </div>
      <p className="Home-intro">
        To get started, edit <code>src/App.js</code> or <code>src/Home.js</code>
        and save to reload.
      </p>

      <div>
        {!keycloak.authenticated ? (
          <button onClick={() => keycloak.login()}>Login</button>
        ) : (
          <button onClick={() => keycloak.logout()}>Logout</button>
        )}
      </div>

      <ul className="Home-resources">
        <li>
          <a href="https://github.com/jaredpalmer/razzle">Docs</a>
        </li>
        <li>
          <a href="https://github.com/jaredpalmer/razzle/issues">Issues</a>
        </li>
        <li>
          <a href="https://palmer.chat">Community Slack</a>
        </li>
      </ul>
    </div>
  )
}

export default withKeycloak(Home)
```

### Hook Usage

Alternately, when a component requires access to `Keycloak`, you can also use the `useKeycloak` Hook.

```jsx
...

import { useKeycloak } from '@react-keycloak/razzle'

const Home = () => {
  const [keycloak, initialized, isServer] = useKeycloak()
  console.log('Is running on server:', isServer)
  console.log('Keycloak is initialized:', initialized)

  return (
    <div className="Home">
      <div className="Home-header">
        <img src={logo} className="Home-logo" alt="logo" />
        <h2>Welcome to Razzle</h2>
      </div>
      <p className="Home-intro">
        To get started, edit <code>src/App.js</code> or <code>src/Home.js</code>
        and save to reload.
      </p>

      <div>
        {!keycloak.authenticated ? (
          <button onClick={() => keycloak.login()}>Login</button>
        ) : (
          <button onClick={() => keycloak.logout()}>Logout</button>
        )}
      </div>

      <ul className="Home-resources">
        <li>
          <a href="https://github.com/jaredpalmer/razzle">Docs</a>
        </li>
        <li>
          <a href="https://github.com/jaredpalmer/razzle/issues">Issues</a>
        </li>
        <li>
          <a href="https://palmer.chat">Community Slack</a>
        </li>
      </ul>
    </div>
  )
}

export default Home
```

## Examples

See inside `examples/razzle-app` folder of [`@react-keycloak/react-keycloak-examples`](https://github.com/react-keycloak/react-keycloak-examples) repository for a sample implementation.

## Contributing

See the [contributing guide](https://github.com/react-keycloak/react-keycloak/blob/master/CONTRIBUTING.md) to learn how to contribute to the repository and the development workflow.

## License

MIT

---

If you found this project to be helpful, please consider buying me a coffee.

[![buy me a coffee](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://buymeacoff.ee/4f18nT0Nk)
