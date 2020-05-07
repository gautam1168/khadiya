---
title: "Vue 3 + Rollup"
date: 2020-05-06T20:56:14+05:30
draft: true
---

I will setup a basic Vue 3 project with Rollup. The official vue-next repo gives a basic setup using [Webpack](https://github.com/vuejs/vue-next-webpack-preview).

## Why
_ES Modules_ are not supported as an output format by Webpack. Rollup supports this format in addition to the standard _umd_ and _commonjs_ formats and seems to create smaller bundles than webpack. 

## TLDR;
Get the setup for running a vue3 project with rollup as bundler from [here](https://github.com/gautam1168/rollup-vue-next.git). Clone it and run `npm run serve`

## Manual Setup
Follow these steps to setup a project named `test`. At the end your directory structure should look like this
```
test
  |-- public
       |-- index.html
  |-- src
       |-- index.ts
       |-- App.vue
  |-- package.json
  |-- rollup.config.js
  |-- serve.js

```

1. Create a directory for your project and run `npm init`.
    ```sh { linenos=false }
      mkdir test
      cd test
      npm init
    ```
2. Install the latest vue 3, rollup and some plugins for rollup from npm
    ```sh { linenos=false }
      npm i --save vue@3.0.0-beta.2 rollup
      npm i --save-dev @rollup/plugin-node-resolve @vue/compiler-sfc rollup-plugin-alias rollup-plugin-commonjs rollup-plugin-vue
    ```
3. Create a index.html file 
    ```sh { linenos=false }
      mkdir public
      cd public 
      touch index.html
    ```
    Add the following content to your `index.html`. Notice we will be using a es module so we put type as `module` in the script tag.
    ```html
      <html>
        <head>
          <title>vue3 test</title>
          <link href="bundle.esm.css" rel="stylesheet">
        </head>
        <body>
          <div id="app"></div>
          <script src="bundle.esm.js" type="module"></script>
        </body>
      </html>
    ```
4. Add a vue component to the project. First come back out of the public folder and create a src folder.
    ```sh { linenos=false }
      cd ..
      mkdir src
      cd src
    ```

    Now, add a Vue component and an index file
    ```sh { linenos=false }
      touch App.vue
      touch index.ts
    ```

    Add the following content to the `App.vue` file
    ```vue 
      <template>
        <div>Click the button to increase the number</div>
        <div> {{ count }} </div>
        <button @click="inc">Increase</button>
      </template>
      <script lang="ts">
      import { ref } from "vue";
      export default {
        setup() {
          const count = ref(0)
          const inc = () => {
            count.value++
          }
          return {
            count,
            inc
          }
        }
      }
      </script>
      <style scoped>
      * {
        font-family: Arial, Helvetica, sans-serif;
      }
      </style>
    ```

    Add the following content to the index.ts file created above
    ```ts
      import { createApp } from 'vue';
      import App from './App.vue';

      createApp(App).mount('#app');
    ```

5. Now that we have the code, lets add the rollup configuration that will compile it. Go back to the root directory and add a rollup.config.js file
    ```sh { linenos=false }
      cd ..
      touch rollup.config.js
    ```

    And add the following configuration to this file
    ```js
      import alias from 'rollup-plugin-alias';
      import commonjs from 'rollup-plugin-commonjs';
      import VuePlugin from 'rollup-plugin-vue';
      import css from 'rollup-plugin-css-only';

      export default {
        input: "src/index.ts",
        output: {
          file: "public/bundle.esm.js",
          format: "es"
        },
        plugins: [
          commonjs(),
          VuePlugin({
            css: false
          }),
          css(),
          alias({
            resolve: [ '.js', '.ts' ],
            entries: [
              { find: 'vue', replacement: 'node_modules/vue/dist/vue.runtime.esm-browser.js' }
            ]
          }),
          
        ],
        watch: {
          include: 'src/**',
          exclude: 'node_modules/**'
        }
      }
    ```
6. At this point, you will be able to compile your code using rollup. But, since rollup doesn't have its own server or hot module replacement (yet), we should also add a server for development.

    ```sh { linenos=false }
      npm i --save-dev live-server
      touch serve.js
    ```

    Add the following content to serve.js
    ```js
      const liveserver = require('live-server');

      const params = {
        port: 8080,
        host: '0.0.0.0',
        root: 'public',
        open: false,
        logLevel: 2,
      };

      liveserver.start(params);
    ```

7. Finally, lets add a script in our package.json so that we can easily start our server and compiler in watch mode. Add the following line to the scripts section of your package.json
    ```json
      {
        "serve": "rollup -c rollup.config.js --watch & node serve.js"
      },
    ```

8. And now just run the serve command and visit your browser on http://localhost:8080 to see the website
    ```sh { linenos=false }
      npm run serve
    ```