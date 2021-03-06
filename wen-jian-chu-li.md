# 文件处理

* 图片处理
* 字体文件
* 第三方JS库

## 图片处理

**使用场景**

* CSS中引入的图片 （file-loader）
* 自动合成雪碧图   (postcss-sprites)
* 压缩图片        (img-loader)
* Base64编码      (url-loader)

**相关配置参数**

```javascript
# webpack.config.js

module: {
    rules: [
        {
            test: /\.less$/,
            use: ExtractTextWebpackPlugin.extract({
                fallback: {
                    loader: 'style-loader',
                    options: { singleton: true }
                },
                use: [
                    {
                        loader: 'css-loader'
                    },
                    {
                        loader: 'postcss-loader',
                        options: {
                            ident: 'postcss',
                            plugins: [
                                // 处理雪碧图
                                require('postcss-sprites')({
                                    // 指定生成的雪碧图放置的位置
                                    spritePath: 'dist/assets/imgs',
                                    retina: true // retina 屏幕处理
                                    // 这里需要注意，文件名需要添加 @2x 来声明是处理 retina屏 的图
                                }),
                                require('postcss-cssnext')()
                            ]
                        }
                    },
                    {
                        loader: 'less-loader'
                    }
                ]
            })
        },
        {
            test: /\.js$/,
            use: [
                {
                    loader: 'babel-loader',
                    options: {
                        presets: ['env'],
                        plugins: ['lodash']
                    }
                }
            ]
        },
        {
            // 对图片文件的处理
            test: /\.(png|jpg|jpeg|gif)$/,
            use: [
                // 引入文件
                // {
                //     loader: 'file-loader',
                //     options: {
                //         publicPath: './assets/imgs/',
                //         // output: 'dist/',
                //         useRelativePath: true
                //     }
                // }
                // url 转换base64，如果设定的 limit 大小偏差太大，就会直接转换成图片，所以使用了 url-loader 可以不需要使用 file-loader了
                {
                    loader: 'url-loader',
                    options: {
                        name: '[name]-[hash:5].[ext]',
                        limit: 100000,  // 文件大小限制
                        publicPath: './assets/imgs/',
                        useRelativePath: true
                    }
                },
                // 图片压缩
                {
                    loader: 'img-loader',
                    options: {
                        pngquant: {
                            quality: 80
                        }
                    }
                }
            ]
        }
    ]
},

```

## 字体文件处理

处理方式和图片处理的方式一样

```js
 {
    test: /\.(et0|woff2?|ttf|svg)$/
    use: [
        {
            loader: 'url-loader'    // file-loader,
            options: {
                name: '[name]-[hash:5].[ext]',
                limit: 5000,
                publicPath: '',
                ouputPath: 'dist',
                useRelativePath: true
            }
        }
    ]
 }
```

## 处理第三方库

### 场景

* 第三方库在远程的 CDN 上
* 下载了第三方库，引用文件的形式

### webpack 怎么处理

* 插件 webpack.providePlugin 自动加载模块，而不必到处 import 或 require 。
* imports-loader
* window

```js
$ npm i jquery -S
# webpack.config.js npm 管理

plugins: [
    new webpack.ProvidePlugin({
        $ : 'jquery',
    })
]

# 不是通过 NPM 管理的依赖, 需要配置 解析处理文件

resolve: {
    alias: {
        jquery$: path.resolve(__dirname, 'src/lib/jquery.min.js')
    }
},
plugins: [
    new webpack.ProvidePlugin({
        $ : 'jquery',
    })
]

# imports-loader

rules: [
    {
        test: path.resolve(__dirname, 'src/app.js')
        use: {
            loader: 'imports-loader',
            options: {
                $ : 'jquery'
            }
        }
    }
],
resolve: {
    alias: {
        jquery$: path.resolve(__dirname, 'src/lib/jquery.min.js')
    }
}

```

## HTML in Webpack

* 自动生成HTML （`HtmlWebpackPlugin`）
* 场景优化

### HtmlWebpackPlugin 配置

* template   指定模版
* filename   指定文件名
* minify     是否压缩
* chunks     指定那个生成此模版的 chunk
* inject     是不是让插件帮你把想要的文件插入到Html页面中

```js
var HtmlWebpackPlugin = require('HtmlWebpackPlugin');

plugins: [
    new HtmlWebpackPlugin ({
        filename: 'index.bundle.html',
        template: './index.html',
        chunks: ['app'],
        minify: {
        }
    })
]
```

[插件文档](https://github.com/jantimon/html-webpack-plugin#configuration)


### 生成的HTML中引入图片

使用 `html-loader` 插件进行处理

```js
rules: [
    {
        test: /\.html$/,
        use: {
            loader: 'html-loader',
            options: {
                attrs: ['img:src'， 'img:data-src'] // 默认配置
            }
        }
    }
]
```

### 配合优化

通过提取公共代码，每个页面加载公共代码，就会是一个请求，所以我们在首页提前载入代码直接嵌入滴代码，从而减少请求。

提前载入 webpack 加载代码:

* inline-manifest-webpack-plugin     (专门将webpack生成的代码载入页面，和html-loader 配合又写bug)
* html-webpack-inline-chunk-plugin （推荐）

```js
var htmlWebpackInlineChunk = require('html-webpack-inline-chunk-plugin');

plugins: [
    // 提取公共代码
    new webpack.optimize.CommonChunkPlugin({
        name: 'manifest'
    }),
    new htmlWebpackInlineChunk ({
        inlineChunk: ['manifest']
    }),
    new HtmlWebpackPlugin ({
        filename: 'index.bundle.html',
        template: './index.html',
        minify: {
        }
    })
]
```
