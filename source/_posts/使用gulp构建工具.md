title: 使用gulp构建工具
date: 2015-12-09 16:40:54
tags: 
 - 项目构建
---
之前一个demo中用的是grunt，照着grunt用到的插件找了下gulp的，总体使用还算顺畅，说实话并不觉得学习成本有降低什么的，差不多。不过也遇到一些问题：

1、gulp.dest()输出目录需要用"./"而不能"/"

2、gulp的jshint有些规则检测不到，而grunt却检测到了，可以用"asi": true测试下。（据说现在大家都用eslint代替jslint，看了下有eslint180条左右的规则，吓cry了好么。）

3、gulp任务有些执行完毕后不会有消息提示，而grunt的插件就友好很多。

 

gulp.task('uglify', ['jshint'], function() {//...}
package.json
```json

{
  "name": "vip.uc108",
  "version": "0.1.0",
  "devDependencies": {
    "del": "^1.2.0",
    "gulp": "^3.9.0",
    "gulp-concat": "^2.6.0",
    "gulp-csscomb": "^3.0.6",
    "gulp-eslint": "^1.0.0",
    "gulp-file-include": "^0.13.7",
    "gulp-imagemin": "^2.3.0",
    "gulp-jshint": "^1.11.2",
    "gulp-livereload": "^3.8.0",
    "gulp-minify-css": "^1.2.0",
    "gulp-notify": "^2.2.0",
    "gulp-rename": "^1.2.2",
    "gulp-ruby-sass": "^1.0.5",
    "gulp-uglify": "^1.2.0"
  }
}

gulpFile.js
```javascript
var gulp = require('gulp');

var sass = require('gulp-ruby-sass');
var csscomb = require('gulp-csscomb');
var fileinclude = require('gulp-file-include');
var imagemin = require('gulp-imagemin');
var jshint = require('gulp-jshint');
var livereload = require('gulp-livereload');
var cssmin = require('gulp-minify-css');
var notify = require('gulp-notify');
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');


gulp.task('sass', function() {
    return sass('./static/introduce/scss', {
            style: "compressed"
        })
        .on('error', function(err) {
            console.error('Error!', err.message);
        }).pipe(gulp.dest('./static/introduce/css'))
        .pipe(livereload({
            start: true
        }));
});

gulp.task('css', function() {
    return gulp.src(['./static/activity/**/*.css', '!./static/activity/**/*min.css'])
        .pipe(csscomb()).
    pipe(cssmin()).
    pipe(rename(function(path) {
            path.basename += ".min";
        }))
        .pipe(gulp.dest('./static/activity'))
        .pipe(livereload({
            start: true
        }));
});

gulp.task('image', function() {
    gulp.src('static/activity/index/img/*')
        .pipe(imagemin())
        .pipe(gulp.dest('./static/activity/index/img'));
});

gulp.task('jshint', function() {
    return gulp.src(['./static/activity/**/*.js', '!./static/activity/**/*.min.js'])
        .pipe(jshint('.jshintrc'))
        .pipe(jshint.reporter('default'));
});

gulp.task('uglify', ['jshint'], function() {
    return gulp.src(['./static/activity/**/*.js', '!./static/activity/**/*.min.js'])
        .pipe(uglify({
            ext: ".min.js"
        }))
        .pipe(rename(function(path) {
            path.basename += ".min";
        }))
        .pipe(gulp.dest('./static/activity/'));
})

gulp.task('fileinclude', function() {
    return gulp.src('src/html/**/*.html').
    pipe(fileinclude({
        prefix: '@@',
        basepath: '@file'
    })).pipe(gulp.dest('./view'));
});

gulp.task('watch', function() {
    gulp.watch(['./static/activity/**/*.js', '!./static/activity/**/*.min.js'], ['uglify']);
});

gulp.task('default', ['image', 'fileinclude'], function() {
    gulp.src('package.json').pipe(notify("default finished"));
});
```
关于gulp插件，有空再试下这些~~

gulp-rev

gulp-concat

gulp-sourcemaps

gulp-connect

 