---
title: '发布一个自己的npm包之wxh-tools'
date: 2019-08-01 14:52:15
tags: ['node','javascript']
---

工作中经常重复写一些方法, 整理起来托管到[ GitHub仓库中 ](https://github.com/Wxh16144/wxh-tools/tree/master/src/tools),
搭建一个`npm`包, 每次使用时, 按需下载打包成一个文件, 方便使用;
<!--more-->

### 项目地址
> NPM地址: [https://www.npmjs.com/package/wxh-tools](https://www.npmjs.com/package/wxh-tools);

> GITHUB: [https://github.com/Wxh16144/wxh-tools](https://github.com/Wxh16144/wxh-tools);

工具截图:
<img style="width:100%" src="https://raw.githubusercontent.com/Wxh16144/wxh-tools/master/use.gif" alt="wxh-tools工具截图">


### 项目目录
+ bin
  - www   <span style="font-size:.8em;color:green">//  执行包命令
+ config
  - methods_list.json   <span style="font-size:.8em;color:green">//  所有/src/tools/下方法列表, 由create_methods_list.js 创建
  - webpack.config.js   <span style="font-size:.8em;color:green">//  打包时使用的webpack配置
+ dist  <span style="font-size:.8em;color:green">//  打包目录
+ reoisutiry  <span style="font-size:.8em;color:green">//  临时目录，用于存放从git下载到本地的资源
+ src
  + tools   <span style="font-size:.8em;color:green">//  存放平常使用的方法(全部)
  + utils
    - create_file_by_user_select.js   <span style="font-size:.8em;color:green">//  根据用户选择的方法，创建文件夹
    - get_json_file.js  <span style="font-size:.8em;color:green">//  读取json文件内容并序列化
    - read_dir_file.js  <span style="font-size:.8em;color:green">//  读取某一个目录下的所有文件
    - webpack_build.js  <span style="font-size:.8em;color:green">//  webpack配置
  - get.js  <span style="font-size:.8em;color:green">//  从git上下载文件
  - index.js  <span style="font-size:.8em;color:green">//  获取用户选择config
  - main.js   <span style="font-size:.8em;color:green">//  入口文件
- .babelrc
- create_methods_list.js  <span style="font-size:.8em;color:green">//  读取src/tools/所有方法并生成文件

### 安装依赖

```bash
  npm install @babel/cli@7.5.5  @babel/core@7.5.5  @babel/preset-env@7.5.5  babel-loader@8.0.0-beta.0  core-js@3.1.4  download@7.1.0  inquirer@6.5.0  ora@3.4.0  rimraf@2.6.3  shelljs@0.8.3  webpack@4.37.0  webpack-cli@3.3.6 --save
```
> 列举出来的`npm` 语句太长, 为了偷懒我也使用了 `src/utils/get_json_file.js` 方法;

**Demo:**
```javascript
  const method = require('./src/utils/get_json_file');
  const json_file = method('./package.json');
  const dep = json_file.dependencies;
  const res = Object.keys(dep).reduce((str, key) => {
    return `${str}  ${key}@${(dep[key]).slice(1)}`
  }, '')
  console.log(res);
  // @babel/cli@7.5.5  @babel/core@7.5.5  @babel/preset-env@7.5.5  babel-loader@8.0.0-beta.0  core-js@3.1.4  download@7.1.0  inquirer@6.5.0  ora@3.4.0  rimraf@2.6.3  shelljs@0.8.3  webpack@4.37.0  webpack-cli@3.3.6
```

### 工具封装

**get_json_file.js**
> 读取`json`文件, 并且序列化

```javascript
  const fs = require('fs');
  /**
  *读取*.JSON文件,并序列化返回
  *
  * @param {*} filepath 文件地址
  * @returns 文件内容
  */
  module.exports = (filepath) => {
    if (!filepath) return '';
    const _Json = fs.readFileSync(filepath);
    return JSON.parse(_Json);
  };
```

**read_dir_file.js**
> 读取某一个目录下的所有文件(不包括文件夹, 不进行递归)

```javascript
  const path = require('path');
  const path_join = path.join;
  const fs = require('fs');

  /**
  *获取目录下的所有文件(不包括文件夹)
  *
  * @param {*} [dir=path_join(__dirname)] 目录
  * @param {*} [reg=/./] 正则表达死
  * @returns 文件数组
  */
  module.exports = (dir = path_join(__dirname), reg = /./) => {
    // 读取所有的文件
    let files = fs.readdirSync(dir);
    return files.reduce((arr, item) => {
      // 获取文件路径
      let fPath = path_join(dir, item);
      // 读取
      let stat = fs.statSync(fPath);
      // 判断是否为文件
      let isFile = stat.isFile();
      // 过滤正则表达式条件
      return isFile && reg.test(item) ? [...arr, item] : arr
    }, []);
  };
```
**get.js**
> 从远程github仓库下载文件
> 使用`download`下载文件包; [download npm 地址](https://www.npmjs.com/package/download);

```javascript
const path = require('path');
const path_join = path.join;
const fs = require('fs');
const download = require('download');

/**
 *从远程github仓库下载文件
 *
 * @param {array} files 文件名
 * @param {string} [savePath=path_join(__dirname, '../repository')] 文件下载保存地址
 * @returns Promise
 */
module.exports = (files, savePath = path_join(__dirname, '../repository')) => {
  const URL = 'https://raw.githubusercontent.com/Wxh16144/wxh-tools/master/';
  return Promise.all((Array.isArray(files) ? files : [])
    .map(file_name => download(URL + file_name, savePath)));
};
```


### 工具准备完成, 开始构建CONFIG（index.js）
> 使用`inquirer`控制台交互包; [inquirer npm 地址](https://www.npmjs.com/package/inquirer);
> 使用`ora`加载动画包; [ora npm 地址](https://www.npmjs.com/package/ora);

引入所需模块

```javascript
// 路径处理
const path = require('path');
const path_join = path.join;

// 控制台交互
const inquirer = require('inquirer');
const prompt = inquirer.createPromptModule();
// 加载动画
const ora = require('ora');

// 获取下载文件
const download_methods_list = require('./get');
```

> 控制台交互, 一个Promise数组; 

```javascript
// 询问需要添加的方法
const Ask_the_task = [
  // 询问用户需要使用那些方法
  async () => {
    // 添加动画
    const spinner = ora('正在获取所有方法列表...').start();
    // 下载所有方法列表
    const res = await download_methods_list(['config/methods_list.json']);
    // 读取方法列表
    const methods_list = require('./utils/get_json_file')(path_join(__dirname, '../repository/methods_list.json'));
    // 停止动画
    spinner.stop();
    const ALL_TOOLS_SERVER = (methods_list.methods_list || []).map(filename => {
      return filename.replace(/\.js$/, '');
    });
    // 创建问题
    return prompt({
      type: 'checkbox',
      name: 'tools_key',
      message: '请选择您所需要的方法',
      default: [],
      choices: ALL_TOOLS_SERVER
    })
  },
  // 询问用户保存名字
  () => prompt({
    type: 'input',
    name: 'save_file_name',
    message: '保存文件名',
    default: 'utils.js',
  }),
  // 询问用户是否需要打包
  () => prompt({
    type: 'confirm',
    name: 'is_build',
    message: '是否需要使用webpack打包',
    default: true
  }),
];

```
> 从上往下询问问题
> 并合并用户选择的CONFIG, 导出执行方法

```javascript
/**
 *循环提出询问
 *
 * @returns 用户配置
 */
const task = async () => {
  const config = {};
  for (const fn of Ask_the_task) {
    const P = fn();
    const res = await P;
    Object.assign(config, res)
  }
  return config;
};

// 导出提问方法
module.exports = task;
```

### 根据用户选择下载对应方法 (main.js)
> 使用`rimraf`删除文件包; [rimraf npm 地址](https://www.npmjs.com/package/rimraf);

加载模块
```javascript
const path = require('path');
const path_join = path.join;
const fs = require('fs');

// 删除文件
const rimraf = require('rimraf');

// 获取下载文件
const download_methods_list = require('./get');

// 加载动画
const ora = require('ora');

// webpack 打包
const webpack_build_function = require('./utils/webpack_build');

// 交互问题
const task = require('./index');
```

> 询问用户, 下载方法到本地, 合并方法文件, 使用webpack 打包

```javascript
// 所有问题
const task = require('./index');
(async function () {
  const CONFIG = await task();
  // 获取用户配置
  const { tools_key = [], is_build = true } = CONFIG;
  // 临时目录
  const DIRECTORY = path_join(__dirname, '../temporary/');
  rimraf.sync(DIRECTORY);
  // 添加下载动画
  const download_spinner = ora('正在下载所选方法数据...').start();
  // 根据所选方法收集
  // 下载所有方法列表
  const load_select_methods = await download_methods_list(
    tools_key.map(key => `src/tools/${key}.js`),
    path_join(__dirname, '../temporary')
  );
  // 下载完成
  download_spinner.succeed('所需方法下载完成');
  // 添加创建文件动画
  const create_file_spinner = ora('创建并生成文件中...').start();
  const create_file_by_user_select = require('./utils/create_file_by_user_select');
  const create_file = await create_file_by_user_select(CONFIG);
  create_file_spinner.succeed('创建完成');

  // 添加打包动画
  const build_spinner = ora('处理文件中...').start();
  const build_res = await webpack_build_function(CONFIG);
  build_spinner.succeed(`处理成功,地址:${build_res}`);
}());
```

### 创建合并文件(create_file_by_user_select.js)
> 处理 `github` 下载到本地 `temporary` 目录的方法;
> 打包好的文件会在 **window** 有一个全局的方法集为 **WT**

```javascript
const path = require('path');
const path_join = path.join;
const fs = require('fs');

/**
 *创建合并文件
 *
 * @param {*} CONFIG 用户输入配置
 */
module.exports = async CONFIG => {
  // 获取临时目录路径
  const DIRECTORY = path_join(__dirname, '../../temporary/');
  const { tools_key = [] } = CONFIG;
  let javascript = '';
  tools_key.forEach(fileN => {
    // 逐个导入方法
    javascript += `import ${fileN}  from './${fileN}.js';
    `
  });
  const myexport = tools_key.map(String).join(',');
  // 绑到window上
  // 导出方法
  javascript += `
  window.WT = { ${myexport} };
  export { ${myexport} };`
  // 写到临时目录
  fs.writeFileSync(path_join(DIRECTORY, 'utils.js'), javascript);
}
```

### 使用webpack打包(webpack_build.js)
> 传入用户CONFIG配置; 通过 `--build` 标识是否使用`@babel/preset-env`打包;
> 使用`shell`下载文件包; [shell npm 地址](https://www.npmjs.com/package/shell);
> 这里有个方法 `process.cwd()` 是获取用户执行方法的目录; 使用这个作为webpack打包出口;

执行`webpack` 打包, 使用配置**config/webpack.config.js**文件

```javascript
const path = require('path');
const path_join = path.join;
const fs = require('fs');
const shell = require('shelljs');


/**
 *检查用户输入的文件名是否合法
 *
 * @param {*} filename 文件名
 * @returns 文件名
 */
const get_File_Name = filename => {
  if (/\.js$/.test(filename)) {
    return filename
  } else {
    return filename + '.js'
  }
}

/**
 *使用webpack打包文件
 *
 * @param {*} CONFIG 用户配置
 * @returns 结果
 */
module.exports = async CONFIG => {
  // 配置参数
  const { tools_key = [], is_build = true, save_file_name } = CONFIG;
  // 获取到webpack配置地址
  const CONFIG_PATH = path_join(__dirname, '../../config/');
  // 打包输出路径地址 // 用户打开控制台的地址
  const OUTPUT_PATH = path_join(process.cwd(), get_File_Name(save_file_name));
  // 拼接 webpack 命令行
  const CMD = `npx webpack ${is_build ? '--build' : ''}  --output ${OUTPUT_PATH}`;
  // 返回异步结果(shells是同步的)
  return new Promise((resovle, reject) => {
    shell.cd(CONFIG_PATH);
    const EXEC_RES = shell.exec(CMD);
    if (EXEC_RES.code === 0) {
      resovle(OUTPUT_PATH); //返回成功地址
    } else {
      reject(EXEC_RES); //失败
    }
  });
};
```


### 配置文件

**.babellrc**

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "node": "10"
        },
        "modules": "umd",
        "useBuiltIns": "usage",
        "corejs": 3,
      }
    ]
  ],
  "plugins": [],
  "ignore": [
    "src/tools"
  ]
}
```

**config/webpack.config.js**
```javascript
const path = require('path');
// 判断是否有 --build 标识
const IS_BUILD = (Array.isArray(process.argv) ?process.argv : []).includes('--build');
// babel
const BABEL_BUILD = {
  test: /\.js$/,
  exclude: /(node_modules|bower_components)/,//排除掉node_module目录
  use: {
    loader: 'babel-loader',
    options: {
      presets: [
        ["@babel/preset-env", {
          modules:false,
          useBuiltIns: "usage",
          corejs: 3,
        }]
      ],
      plugins: [],
    }
  }
};
module.exports = {
  entry: path.resolve(__dirname, '../temporary/utils.js'),
  mode: 'production',
  output: {
    filename: 'utils.js',
    path: path.resolve(__dirname, '../temporary/dist')
  },
  module: {
    rules: [
      (IS_BUILD ? BABEL_BUILD : {}) // --build
    ]
  }
};
```
**bin/www**
```bash
#! /usr/bin/env node
require('../src/main.js');
```

## 发布到NPM
> 先修改`package.json` 文件, 添加 `bin` 属性
> 代表全局方法名为: **wxh**

```json
"bin": {
  "wxh": "./bin/www"
},
```
#### 登录自己的npm账号
```bash
npm adduser
```
#### 推送包到npm仓库中
```bash
npm publish
```

#### 推送npm包注意事项
+ 检查自己的包在`npm`库中有没有重命名
+ 每一次推送包都要修改`package.json` 的 `version` 字段


### 写在最后

到这里我们整个包都已经写好了, 只需要安装到全局, 我们就可以使用了;
之后的日常维护, 我们只需要在  **src/tools** 目录下添加一个个方法就好了;
偷懒写一个方法 **create_methods_list.js**;
方法根据 **/src/tools**目录下的方法文件, 生成 `methods_list.json` 文件到**config**目录;

```javascript
const fs = require('fs');
const path = require('path');
const path_join = path.join;
const read_dir_file = require('./src/utils/read_dir_file');
// 读取
const fileArr = read_dir_file(path_join(__dirname, './src/tools'));
fs.writeFileSync(path_join(__dirname, './config/methods_list.json'),
  JSON.stringify({ methods_list: fileArr })
);
```


