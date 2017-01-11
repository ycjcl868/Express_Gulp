 > 原来用的`React`+`Webpack`时，体验到那种同步压缩修改的快感。这次使用`Express`时，也想达到那种效果，一番折腾后，于是有了下图，项目地址：[https://github.com/ycjcl868/Express_Gulp](https://github.com/ycjcl868/Express_Gulp)

![](./dist/img/2.gif)

### 目的
我使用`Express`+`Ejs`+`Less`开发，想开发时对所有资源进行`压缩`并`同步`到浏览器端，Google搜索一遍，都不是太符合我的项目要求。于是看着`Gulp文档`结合`npmjs.com`终于折腾完毕。


### 配置
下面说下我的配置方法：

#### 我的目录结构：

```
├── app.js   # Express Server
├── bin
│   └── www  # 启动Server
├── dist     # 编译压缩目录(部署目录)
│   ├── css
│   ├── img
│   ├── js
│   ├── views
│   └── lib  # 第三方库目录(bower安装)
├── .bowerrc # bower前端安装库
├── gulpfile.js  # Gulp配置文件
├── package.json
├── public       # 开发目录 
│   ├── img
│   ├── js
│   └── less     
├── routes
│   ├── index.js
│   └── users.js
└── views      # html
    ├── error.ejs
    └── index.ejs
```

#### package.json文件内容：

```json
{
  "name": "voteapp",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "dependencies": {
    "body-parser": "~1.13.2",
    "cookie-parser": "~1.3.5",
    "debug": "~2.2.0",
    "ejs": "~2.3.3",
    "express": "~4.13.1",
    "morgan": "~1.6.1",
    "serve-favicon": "~2.3.0"
  },
  "devDependencies": {
    "browser-sync": "^2.18.6",
    "del": "^2.2.2",
    "gulp": "^3.9.1",
    "gulp-cache": "^0.4.5",
    "gulp-clean-css": "^2.3.2",
    "gulp-concat": "^2.6.1",
    "gulp-ejs": "^2.3.0",
    "gulp-htmlmin": "^3.0.0",
    "gulp-imagemin": "^3.1.1",
    "gulp-less": "^3.3.0",
    "gulp-livereload": "^3.8.1",
    "gulp-nodemon": "^2.2.1",
    "gulp-uglify": "^2.0.0",
    "gulp-watch": "^4.3.11"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}

```


#### gulpfile.js文件

```js
'use strict';
var gulp = require('gulp');
var minify = require('gulp-clean-css');
var browserSync = require('browser-sync');
var nodemon = require('gulp-nodemon');
var cache = require('gulp-cache');
var uglify = require('gulp-uglify');
var htmlmin = require('gulp-htmlmin');
var imagemin = require('gulp-imagemin');
var less = require('gulp-less');
var path = require('path');
var livereload = require('gulp-livereload');
var concat = require('gulp-concat');
var jshint = require('gulp-jshint');
var cssBase64 = require('gulp-base64');
var del = require('del');

// 删除文件
gulp.task('clean', function(cb) {
    del(['dist/css/*', 'dist/js/*', 'dist/img/*','dist/views/*'], cb)
});

// 压缩ejs
gulp.task('ejs', function() {
  return gulp.src('views/**/*.ejs')
      .pipe(htmlmin({collapseWhitespace: true}))
      .pipe(gulp.dest('dist/views/'));
});

// 压缩less
gulp.task('less', function () {
  return gulp.src('public/less/**/*.less')
      .pipe(less({
          paths: [ path.join(__dirname, 'less', 'includes') ]
      }))
      .pipe(cssBase64())
      .pipe(minify())
      .pipe(gulp.dest('dist/css/'));
});


// 压缩js
gulp.task('js', function () {
    return gulp.src('public/js/**/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'))
        .pipe(uglify({ compress: true }))
        .pipe(gulp.dest('dist/js/'))
});


// 压缩img
gulp.task('img', function() {  
  return gulp.src('public/img/**/*')        //引入所有需处理的Img
    .pipe(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))      //压缩图片
    // 如果想对变动过的文件进行压缩，则使用下面一句代码
    // .pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true }))) 
    .pipe(gulp.dest('dist/img/'))
    // .pipe(notify({ message: '图片处理完成' }));
});


// 浏览器同步，用7000端口去代理Express的3008端口
gulp.task('browser-sync', ['nodemon'], function() {
  browserSync.init(null, {
    proxy: "http://localhost:3008",
        files: ["dist/views/*.*","dist/css/*.*","dist/js/*.*","dist/img/*.*"],
        browser: "google chrome",
        port: 7000
  });
});

// 开启Express服务
gulp.task('nodemon', function (cb) {
  
  var started = false;
  
  return nodemon({
    script: 'bin/www'
  }).on('start', function () {
    // to avoid nodemon being started multiple times
    // thanks @matthisk
    if (!started) {
      cb();
      started = true; 
    } 
  });
}); 

gulp.task('build',['clean','less','ejs','js','img'],function () {
    
});

gulp.task('default',['browser-sync'],function(){
  // 将你的默认的任务代码放这

    // 监听所有css文档
    gulp.watch('public/less/*.less', ['less']);

    // 监听所有.js档
    gulp.watch('public/js/*.js', ['js']);

    // 监听所有图片档
    gulp.watch('public/img/**/*', ['img']);
    // 监听ejs
    gulp.watch('views/**/*.ejs', ['ejs']);

   // 创建实时调整服务器 -- 在项目中未使用注释掉
  var server = livereload();
   // 监听 dist/ 目录下所有文档，有更新时强制浏览器刷新（需要浏览器插件配合或按前文介绍在页面增加JS监听代码）
  gulp.watch(['public/dist/**']).on('change', function(file) {
    server.changed(file.path);
  });
});
```

#### app.js文件

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var routes = require('./routes/index');
var users = require('./routes/users');

var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'dist/views'));
app.set('view engine', 'ejs');

// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'dist')));

app.use('/', routes);
app.use('/users', users);

// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});

// error handlers

// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}

// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});


module.exports = app;
```

然后先在根目录下执行安装：
`npm install`，使用时先运行`gulp build`将文件压缩、打包、编译，然后再执行`gulp`开启自动更新服务器。

![](./dist/img/3.png)