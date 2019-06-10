---
title: "React, jest, babel and yarn - notes to self"
date: 2018-11-24T12:00:00+01:00
draft: false
---
This blog post is a set of notes for my future self on how to set up a basic React project using Yarn, with babel for compilation and Jest as the test framework. I’ve included a simple 'Hello World' unit test to make sure everything is correctly hooked up.

Use asdf to install NodeJS:

```bash
asdf plugin-list-all
asdf plugin-add nodejs
asdf list-all nodejs
asdf install nodejs 11.1.0
asdf global nodejs 11.1.0
```

To create a React app, first install the create-react-app package:

```bash
yarn add global create-react-app
```

Yarn errors when creating a React app, so use npx instead:

```bash
npx create-react-app learn-react
```

Move into the newly created project directory:

```bash
cd learn-react
```

Add Babel as a project dependency:

```bash
yarn add --dev babel-core@⁷.0.0-bridge.0 @babel/core regenerator-runtime
```

Add a dependency on the babel-jest package.

```bash
yarn add --dev babel-jest
```

Edit package.json and add:

```json
"scripts": {
  "test": "jest react-scripts test",
},
"jest": {
  "transform": {
    "^.+\\.jsx?$": "babel-jest"
  }
}
```

Create a `.babelrc` file:

```bash
touch .babelrc
```

Add necessary package dependencies for Babel:

```bash
yarn add --dev @babel/preset-env @babel/preset-react
```

Edit `.babelrc` and add:

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

Remove the existing source files:

```bash
rm -rf src/*
```

Add the React test renderer as a dependency:

```bash
yarn add --dev react-test-renderer
```

Add `src/hello-world.test.js` and add:

```javascript
import React from 'react';
import ShallowRenderer from 'react-test-renderer/shallow';
import HelloWorld from './hello-world'

test('Hello World', () => {
  const renderer = new ShallowRenderer();
  renderer.render(<HelloWorld />);
  const result = renderer.getRenderOutput();

  expect(result.type).toBe('div')
  expect(result.props.children).toEqual('Hello World')
});
```

Run `yarn test` and make sure the test fails.

Add `src/hello-world.js` and add:

```javascript
    import React from 'react';

    export default function HelloWorld() {
      return (
        <div>Hello World</div>
      );
    }
```

Run `yarn test` and make sure the test passes.

I’ve created a [simple project template repo in GitHub](https://github.com/ambye85/react-babel-jest). The readme explains how to set it up on your machine. Feel free to use it if it helps.
