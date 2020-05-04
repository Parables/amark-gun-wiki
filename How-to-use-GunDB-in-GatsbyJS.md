# How to use GunDB in GatsbyJS

Hi. If you've come up here, welcome.

- GatsbyJS: is a Static Site Renderer that helps you build SEO friendly SPA (Single Page Application) with ease. Gatsby uses Webpack (+Babel) + React + GraphQL.
- GunDb: is a Realtime Decentrialized Isomorphic Graph database written in Vanilla JS that allows you to build Offline-first web app.

Combine them together, you can build an Offline-first Decentrialized SEO-friendly single page web app. Your web app can also be installed to Android home screen or PC Desktop just like native apps. People can use it offline (imagine geologists or miners 1000 meters underground where internet doesn't exist), and there devices automatically sync data when they are back online. And there is no central server, every device is a server-client 2-in-1.

Check my website repo for example:

- Source code: [mimiza website repo](https://github.com/mimiza/mimiza.com)

All the codes below are copied from this repo. I don't have enough time to create a new clean repo just for the sake of this tutorial. So, please expect that you will see some lines that make no sense, because it is related to other things. Anyway, the website is a offline-first multilingual single page web app with a dashboard. You can fork it, edit it for your own good.

Firstly, we create a Global Context Provider. In my repo, it is created in **/src/services/context.js**.

```
import Gun from 'gun'
import 'gun/sea'
import React, { Component } from 'react'
import { access } from './access'

const defaultState = {
    authenticated: false,
    username: null,
    access: [],
}
const GlobalContext = React.createContext(defaultState)

export class GlobalContextProvider extends Component {
    constructor(props) {
        super(props)

        this.state = defaultState

        if (window && !(window.gun || window.user || window.sea)) {
            window.gun = Gun(['https://gunjs.herokuapp.com/gun', 'https://www.raygun.live/gun', 'https://e2eec.herokuapp.com/gun'])
            window.user = window.gun.user()
            window.sea = Gun.SEA
            window.authenticated = new CustomEvent('authenticated', {
                detail: { authenticated: true },
            })
        }
    }

    componentDidMount() {
        if (window) {
            var { user, authenticated } = window

            user.recall({ sessionStorage: true }, ack => {
                if (user.is && user._.sea) window.dispatchEvent(authenticated)
            })

            window.addEventListener('authenticated', event => {
                this.setState({
                    ...event.detail,
                    username: user.is.alias,
                    access: access(user.is.alias),
                })
            })
        }
    }

    componentWillUnmount() {
        if (window) window.removeEventListener('authenticated', null)
    }

    render() {
        return (
            <GlobalContext.Provider value={this.state}>
                {this.props.children}
            </GlobalContext.Provider>
        )
    }
}

export default GlobalContext
```

Gatsby is a SSR so we can't use gun directly. You can import gun and sea, but to use it, you must make sure it is called in browser, by using **if (window) {...}**.

Now let's create a file at **/src/gatsby-browser.js**. We will wrap our GatsbyJS root element with **GlobalContextProvider**:

```
import React from 'react'
import { GlobalContextProvider } from './src/services/context'
import { redirect } from './src/services/i18n'

export const disableCorePrefetching = () => true

export const onInitialClientRender = () => {
    // on runtime first starts, redirect dummy pages
    redirect()
}

export const wrapRootElement = ({ element }) => (
    <GlobalContextProvider>{element}</GlobalContextProvider>
)
```

Now let's create a new React Component at **/src/components/DataTable.js**. This component will take a prop called "path" and will call gun.get(path) then return a table.

```
import React, { Component } from 'react'

export default class DataTable extends Component {
    state = { headers: [] }

    componentDidMount() {
        if (window) {
            var { gun, sea } = window
            gun.get(this.props.path).on(data => {
                Object.keys(data).map(async key => {
                    try {
                        var _data = await sea.verify(JSON.parse(data[key]), key)
                        if (key === _data.pub) {
                            this.setState({ [key]: _data })
                            var headers = this.state.headers
                            for (var _key in _data) {
                                if (!headers[_key]) {
                                    headers[_key] = _key
                                }
                            }
                            this.setState({ headers: headers })
                        }
                    } catch (error) {}
                })
            })
        }
    }

    componentWillUnmount() {
        if (window) {
            window.gun.get(this.props.path).off()
        }
    }

    render() {
        return (
            <div className="table-container">
                <table className="table is-bordered is-striped is-hoverable is-fullwidth">
                    <thead>
                        <tr>
                            {Object.keys(this.state['headers']).map(key => (
                                <td key={key}>{this.state['headers'][key]}</td>
                            ))}
                        </tr>
                    </thead>
                    <tbody>
                        {Object.keys(this.state).map(
                            key =>
                                key !== 'headers' && (
                                    <tr key={key}>
                                        {Object.keys(this.state[key]).map(
                                            _key => (
                                                <td key={_key}>
                                                    {this.state[key][_key]}
                                                </td>
                                            )
                                        )}
                                    </tr>
                                )
                        )}
                    </tbody>
                </table>
            </div>
        )
    }
}
```

Now let's use DataTable in a page, for example **/src/pages/dashboard/user.js**:

```
import React, { Component } from 'react'

import Dashboard from '../../../layouts/Dashboard'
import DataTable from '../../../components/DataTable'

export default class User extends Component {
    state = {}

    render() {
        const { pageContext } = this.props
        return (
            <Dashboard pageContext={pageContext}>
                <section className="section">
                    <DataTable pageContext={pageContext} path="user" />
                </section>
            </Dashboard>
        )
    }
}
```

## Gun 0.2020.421 + Gatsby build error
If you use Gun 0.2020.421, when trying _gatsby build_, you will see this error: **can't import the named export 'Crypto' from non EcmaScript module**. This error is related to webpack and .mjs files. To fix this error, create this file **/src/gatsby-node.js**:

```
exports.onCreateWebpackConfig = ({ actions }) => {
    actions.setWebpackConfig({
        module: {
            rules: [
                {
                    test: /\.mjs$/,
                    include: /node_modules/,
                    type: 'javascript/auto',
                },
            ],
        },
        resolve: {
            extensions: ['.mjs', '.tsx', '.ts', '.js', '.jsx', '.json'],
        },
    })
}
```

Blessings!