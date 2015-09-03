# ES2015 新的起航

通过 [babel](https://babeljs.io/) 编译, ES2015 终于可以上生产环境, 而今年的 React 生态大热, 社区开始全面拥抱 ES2015.

> https://babeljs.io/docs/learn-es2015/

> 目前兼容主流浏览器(ie9+), 如果要支持 IE8, 需要注意 [Object.defineProperty 只能用于 DOM](http://babeljs.io/docs/advanced/caveats/)

babel 同时提供了 ES2015 的浏览器 runtime, 允许不经转换直接在浏览执行时处理:

```html
<script src="node_modules/babel-core/browser.js"></script>
<script type="text/babel">
class Test {
  test() {
    return "test";
  }
}

var test = new Test;
test.test(); // "test"
</script>
```

## 代码风格

### 规则制定

@TODO

### 工具选型

[jscs]() vs [eslint]()?

@TODO

- with [JSCS-Formatter]() tool
- Friendly with es6 / jsx
- less config

### 环境配置

1. 如何引入 ESLint

    > 最佳实践是本地安装 `eslint`, ([SublimeLinter-eslint](https://github.com/roadhump/SublimeLinter-eslint) 默认支持), 这样做的好处是不需要配置环境变量, 同时可以避免团队成员全局 `eslint` 版本不一致, 也不需要告诉团队成员还有其他依赖需要安装

    ```sh
    $ npm install --save-dev eslint@latest
    ```

2. 使 ESLint 支持 ES2015

    同样为 ESLint 本地安装 babel 解析器 `babel-eslint`:

    ```sh
    $ npm install --save-dev eslint@latest
    ```

    在 `.eslintrc` 中配置解析器:

    ```json
    {
      "parser": "babel-eslint"
    }
    ```

    > 稍后可以在 `.eslintrc` 中添加更多的检查规则

3. `gulpfile.js` - gulp 配置

    ```js
    var gulp = require('gulp')
    var eslint = require('gulp-eslint')

    gulp.task('lint', function () {
      return gulp.src(['src/**/*.js', 'webpack.config.js', 'gulpfile.js'])
        .pipe(eslint())
        .pipe(eslint.format())
        .pipe(eslint.failOnError())
    })

    gulp.task('default', ['lint'], function () {

    })
    ```

    我们创建了 `lint` 任务, 在执行 `gulp` 时会被触发

    ![image](https://cloud.githubusercontent.com/assets/533360/9658228/79b6f6ac-527a-11e5-8c64-ef9498a4e318.png)

4. `package.json` - npm scripts 支持

    ```json
    "scripts": {
      "lint": "gulp lint"
    }
    ```

    也可以通过执行 `npm run lint` 触发

5. 如何保证所有团队成员提交的代码都是通过检查的

    > 最佳实践是在 `git commit` 时进行检查, 如果不通过拒绝提交(如果是 "1 warning" 不强制拒绝), 要完成这样的需求, 我们需要基于 git 配置 `pre-commit` hook, 基于 ESLint 执行检查, 为了将 `pre-commit` 抽离为项目配置, 在项目初始化时进行软链
    
    在项目根目录下创建 `.pre-commit`:

    > 当待提交的 cached 文件命中 `.js$` 时触发 `npm run lint`

    ```sh
    #!/bin/sh
    if git diff --cached --name-only --diff-filter=ACM | grep '.js$' >/dev/null 2>&1
    then
        ./node_modules/.bin/gulp lint
    fi

    exit $?
    ```

    然后更改文件权限为可执行:

    ```sh
    $ chmod +x .pre-commit
    ```
    
    添加 `package.json` 的安装时的软链行为:

    ```json
    "scripts": {
      "postinstall": "ln -s .pre-commit .git/hooks/pre-commit"
    }
    ```

    这样任何团队成员在通过 `npm install` 完成项目初始化后, 都自动添加了 `git commit` 的检查 hook

    ![image](https://cloud.githubusercontent.com/assets/533360/9658260/b8b75fc2-527a-11e5-9599-0092f3b22b4b.png)

6. 编辑器支持(以 [Sublime Text 3](http://www.sublimetext.com/3) 为例, 对于其他编辑器 ESLint 都有广泛的插件支持)

    > 最佳实践是基于强大的 [SublimeLinter3](https://github.com/SublimeLinter/SublimeLinter3) 插件, 后续也可以一致性的进行 `html` 和 `css` 文件的检查

    通过 [Package Control](https://packagecontrol.io/installation) 安装 [SublimeLinter3](https://github.com/SublimeLinter/SublimeLinter3), [SublimeLinter-contrib-eslint](https://github.com/roadhump/SublimeLinter-eslint)

    配置 SublimeLinter, 注册 `javascript (babel)` 类型为 `javascript`:

    ```json
    "syntax_map": {
        "html (django)": "html",
        "html (rails)": "html",
        "html 5": "html",
        "javascript (babel)": "javascript",
        "php": "html",
        "python django": "python"
    }
    ```

    推荐启用 SublimeLinter 的 "show error on save", 这样在保存时能自动检查并快速定位到不规范的代码

    ![image](https://cloud.githubusercontent.com/assets/533360/9658353/484f8a6a-527b-11e5-8a6e-49d6b436779c.png)

    > 为了使编辑器更好的支持 ES2015 和 JSX 语法, 推荐安装 [Babel](https://github.com/babel/babel-sublime), 并且替换默认的 Javascript 语法包
    
    修改 "Settings - User":

    ```json
    "ignored_packages":
    [
        "JavaScript"
    ]
    ```

    > 如果你是 React 的开发者, 推荐集成 React 的 [lint 规则](https://github.com/yannickcr/eslint-plugin-react)

    ```sh
    $ npm install eslint-plugin-react --save-dev
    ```

    在 `.eslintrc` 中配置使用插件:
    
    ```json
    {
      "plugins": [
        "react"
      ],
      "ecmaFeatures": {
        "jsx": true
      },
      "rules": {
        "react/jsx-boolean-value": 1
      }
    }
    ```

    > 推荐兼容 ES2005 和 JSX 的 color scheme [Oceanic Next Color Scheme](https://github.com/voronianski/oceanic-next-color-scheme)

    ![image](https://cloud.githubusercontent.com/assets/533360/9658412/af3f1d6c-527b-11e5-8950-8ac29edd1fcd.png)
