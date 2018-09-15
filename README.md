English | [简体中文](./README.CN.md) 

# Omi <a href="https://www.npmjs.com/package/omi"><img src="https://img.shields.io/npm/v/omi.svg" alt="Version"></a> <a href="https://travis-ci.org/Tencent/omi"><img src="https://api.travis-ci.org/Tencent/omi.svg?branch=master"></a> 

> Omi === Preact + Scoped CSS + Store System + Native Support in 3kb javascript.

<p align="center">
  <a href="##Omi"><img src="./assets/omi.png" alt="Omi"></a>
</p>

Omi has prosperous ecology and strong features through the second-hand preact developments:

- Super tiny size and Super fast
- Extensive React/Preact/Omi compatibility, compatibility with IE9+
- Scoped CSS, made css selector simple, lazy evaluation and package with it's component
- Each component has a update method, you can continue to use setState
- Store system, data and logic of components management for omi without other frameworks dependencies 
- Server side render, Native Suppport, ES6+, JSX, VDOM, React DevTools, HMR ...
- Create website with no build configuration if you use omi-cli, the same as multi-page support of create-react-app

---

- [Getting Started](#getting-started)
	- [Hello Omi](#hello-omi)
    - [Scoped CSS](#scoped-css)
	- [Store](#store)
	- [Lifecycle](#lifecycle)
- [CLI](#cli)
- [Install](#install)
- [Official Plugins](#official-plugins)
- [Links](#links)
- [License](#license)


## Getting Started

### Hello Omi


``` js
import { render, Component } from 'omi';

class Hello extends Component {
    render() {
        return <div> {this.props.name}</div>
    }
}

class App extends Component {
    install() {
        this.name = 'Omi'
    }

    handleClick = (e) => {
        this.name = 'Hello Omi !' 
        this.update()
    }

    style() {
        return `h3{
                    cursor:pointer;
                    color: ${Math.random() > 0.5 ? 'red' :'green'};
                }`
    }

    staticStyle() {
        return `div{
                    font-size:20px;
                }`
    }
	
    render() {
        return (
		<div>
			<Hello name={this.name}></Hello>
			<h3 onclick={this.handleClick}>Scoped css and event test! click me!</h3>
		</div>
		)
    }
}

render(<App />, 'body')
```

Different to preact, you need not to `import { h } from 'omi'`.

Tell Babel to transform JSX into Omi.h() calls:

``` json
{
    "presets": ["env", "omi"]
}
```

You need install env and omi preset for the config:

``` bash
"babel-preset-env": "^1.6.0",
"babel-preset-omi": "^0.1.1",
```

### Scoped CSS

What's the different between `style` and `staticStyle` method? For example：

``` js
render() {
        return (
		<div>
			<Hello name={this.name}></Hello>
			<Hello name={this.name}></Hello>
			<Hello name={this.name}></Hello>
		</div>
		)
    }
```

The `style` method will render three times to html head element, the `staticStyle` will render one times only !
When you update the component `style` method will rerender, but the `staticStyle` will not rerender.


If you want to use the scoped css but you did not want write it in your javascript, you may need [to-string-loader](https://www.npmjs.com/package/to-string-loader), see the [omi-cli config](https://github.com/AlloyTeam/omi-cli/blob/master/template/app/config/webpack.config.dev.js#L156-L162)：

``` js
{
    test: /[\\|\/]_[\S]*\.css$/,
    use: [
        'to-string-loader',
        'css-loader'
    ]
}
```

If your css file name is begin with `_`, the css content will use to-string-loader. For example：

``` js
import Omi from 'omi'
//typeof style is string
import style from './_index.css' 

class App extends Omi.Component {

  staticStyle() {
    return style
  }

  style() {
    return `
      code{
        color: ${Math.random() > 0.5 ? 'red' : 'blue'}
      }
      .app-logo{
        cursor: pointer; 
      }
    `
  }

  render() {
    return (
      <div class="app">
        <header class="app-header">
          <h1 class="app-title">Welcome to Omi</h1>
        </header>
      </div>
    )
  }
}
```

You can use the feature in the project created by omi-cli with no configuration.


### Store

``` js
import { render, Component } from 'omi';

class Hello extends Component {
  render() {
    return <div>{this.$store.name}</div>
  }
}

class App extends Component {

  handleClick = () => {
    this.$store.rename('Hello Omi !')
  }

  render() {
    return (
      <div>
        <Hello ref={c => { this.hello = c }} ></Hello>
        <button onclick={this.handleClick}>Click me to call this.$store.rename('Hello Omi !') </button>
      </div>
    )
  }
}

class AppStore {
  constructor(data, callbacks) {
    this.name = data.name || ''
    this.onRename = callbacks.onRename || function () { }
  }

  rename(name) {
    this.name = name
    this.onRename()
  }
}

const app = new App()
const store = new AppStore({ name: 'Omi' }, {
  onRename: () => {
    app.update()
    //or update part of app
    //app.hello.update()
  }
})

render(app, document.body, { store })
```

You can use `this.$store` in all components to access the data or data logic.

### Lifecycle

| Lifecycle method            | When it gets called                              |
|-------------------------------|--------------------------------------------------|
| `componentWillMount / install`        | before the component gets mounted to the DOM     |
| `componentDidMount / installed`         | after the component gets mounted to the DOM      |
| `componentWillUnmount / uninstall`      | prior to removal from the DOM                    |
| `componentWillReceiveProps` | before new props get accepted                    |
| `shouldComponentUpdate`     | before `render()`. Return `false` to skip render |
| `componentWillUpdate / beforeUpdate`       | before `render()`                                |
| `componentDidUpdate / afterUpdate`        | after `render()`                                 |

## CLI

```bash
$ npm i omi-cli -g               # install cli
$ omi init your_project_name     # init project, you can also exec 'omi init' in an empty folder
$ cd your_project_name           # please ignore this command if you executed 'omi init' in an empty folder
$ npm start                      # develop
$ npm run build                  # release
```

Server Side Render:

```bash
$ omi init-ssr your_project_name     # init ssr project, you can also exec 'omi init-ssr' in an empty folder
$ cd your_project_name               # please ignore this command if you executed 'omi init' in an empty folder
$ npm run dev                        # develop
$ npm run build                      # release
$ npm start                          # release 
```

## Install

``` bash
npm i omi
```

or get it from CDN:

* [https://unpkg.com/omi@3.0.7/dist/omi.min.js](https://unpkg.com/omi@3.0.7/dist/omi.min.js)
* [https://unpkg.com/omi@3.0.7/dist/omi.js](https://unpkg.com/omi@3.0.7/dist/omi.js)

## Official Plugins

- [omi-tap](https://github.com/AlloyTeam/omi/tree/master/plugins/omi-tap): Support tap event in your Omi project.
- [omi-router](https://github.com/AlloyTeam/omi/tree/master/plugins/omi-router): Router for Omi.
- [omi-finger](https://github.com/AlloyTeam/omi/tree/master/plugins/omi-finger): Omi /[AlloyFinger](https://github.com/AlloyTeam/AlloyFinger) integration.
- [omi-transform](https://github.com/AlloyTeam/omi/tree/master/plugins/omi-transform): Omi /[transformjs](https://alloyteam.github.io/AlloyTouch/transformjs/) integration.
- [omi-touch](https://github.com/AlloyTeam/omi/tree/master/plugins/omi-touch): Omi /[AlloyTouch](https://github.com/AlloyTeam/AlloyTouch) integration.

## Links

- [preact github](https://github.com/developit/preact)
- [preactjs.com](https://preactjs.com/)
- [omijs.org](http://omijs.org/)
- [differences to react](https://preactjs.com/guide/differences-to-react)
- [native support](https://github.com/AlloyTeam/omi/tree/master/src/native)

## License

MIT © [dntzhang](https://github.com/dntzhang)
