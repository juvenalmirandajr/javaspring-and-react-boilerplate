## Configuring the Frontend Maven Plugin

Setting up the codebase. To do so, first, install and configure a Maven plugin called `frontend-maven-plugin`.

Add the following to the `<plugins>` section of the `pom.xml`.

```xml
<plugin>
  <groupId>com.github.eirslett</groupId>
  <artifactId>frontend-maven-plugin</artifactId>
  <version>${frontend-plugin.version}</version>
  <configuration>
    <nodeVersion>${node.version}</nodeVersion>
    <yarnVersion>${yarn.version}</yarnVersion>
    <workingDirectory>src/main/frontend</workingDirectory>
    <installDirectory>target</installDirectory>
  </configuration>
  <executions>
    <execution>
      <id>install node and yarn</id>
      <goals>
        <goal>install-node-and-yarn</goal>
      </goals>
    </execution>
    <execution>
      <id>install dependencies</id>
      <goals>
        <goal>yarn</goal>
      </goals>
    </execution>
    <execution>
      <id>webpack build</id>
      <goals>
        <goal>webpack</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```
Put these in the `<properties>` section.

```xml
<properties>
  <java.version>11</java.version>
  <frontend-plugin.version>1.7.6</frontend-plugin.version>
  <node.version>v10.16.0</node.version>
  <yarn.version>v1.16.0</yarn.version>
</properties>
```

### Filesystem Conventions

Frontend code it will place in `src/main/frontend`.

### Setting Up Our React Application

The `package.json` file will place in `src/main/frontend`. This directory will express as the frontend root.

```json
{
  "name": "java-spring-and-react",
  "version": "1.0.0",
  "description": "",
  "main": "webpack.config.js",
  "scripts": {
    "dev:client": "node_modules/.bin/webpack-dev-server --host 0.0.0.0",
    "webpack": "node_modules/.bin/webpack --config ./webpack.config.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "babel-core": "~6.23.1",
    "babel-loader": "~6.3.2",
    "babel-preset-env": "~1.6.1",
    "babel-preset-react": "~6.22.0",
    "css-loader": "^2.1.1",
    "node-sass": "^4.12",
    "sass-loader": "^6.0.2",
    "style-loader": "^0.13.2",
    "webpack": "~2.3.1",
    "webpack-dev-server": "~2.4.2",
    "write-file-webpack-plugin": "~4.2.0"
  },
  "dependencies": {
    "babel-polyfill": "^6.23.0",
    "react": "~15.4.2",
    "react-dom": "~15.4.2"
  }
}
```

Create `.babelrc` file in frontend root.

```json
{
  "presets": ["env", "react"]
}
```

Build the `webpack.config.js` in the frontend root.

```javascript
let WriteFilePlugin = require('write-file-webpack-plugin');

const webpack = require('webpack');
const path = require('path')

module.exports = {
  entry: {
    path: './app/main.js'
  },
  output: {
    path: path.join(__dirname, '../../../target/classes/static'),
    filename: "bundle.js"
  },
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          "style-loader",
          "css-loader",
          "sass-loader"
        ]
      },
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  },
  plugins: [
    new WriteFilePlugin()
  ],
  devtool: 'eval-source-map'
}
```

When `spring-boot:run` starts, the maven plugin will run webpack. Based on this configuration, spring boot and webpack will work together to install the bundled JavaScript file where all of our resulting Spring assets are located (the `target` directory). This will allow us to use the transpiled `bundle.js` in the context of our Spring templates and static resources.

## Creating a Simple React Application

Create `app/main.js` file in frontend root.

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

const App = (props) => {
  return (<h2>Hello from React</h2>);
}

ReactDOM.render(
  <App />,
  document.getElementById('app')
);
```
### Creating a Static `index.html`

The resulting `bundle.js` file can be used safely in our thymeleaf templates and otherwise. To ensure everything is working, let's just add a static `index.html` in `src/main/resources/static/index.html`.

```html
<!DOCTYPE html>
<html lang="eng">
  <head>
    <title>Spring and React Sitting in a Tree</title>
  </head>
  <body>
    <div id="app">
    </div>
    <script src="/bundle.js"></script>
  </body>
</html>
```

### Running Maven Frontend Commands

Based on the way we configured our `pom.xml`, our installation and transpiling will run when we execute the `spring-boot:run` maven task. We can optionally run the `frontend:install-node-and-yarn`, `frontend:yarn`, and `frontend:webpack` maven commands if we would like.

### Development Flow

You can run a webpack dev server instance in your frontend root. So, with `spring-boot:run` running actively, you'll want to open a new terminal window and execute the following:

```no-highlight
cd src/main/frontend
yarn run dev:client
```

Running the `dev:client` task will watch the frontend application filesystem for changes, and transpile with every modification.
