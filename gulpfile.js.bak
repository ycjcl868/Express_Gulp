var gulp = require('gulp');
var del = require('del');

var sourcemaps = require('gulp-sourcemaps');
var minify = require('gulp-clean-css');
var less = require('gulp-less');
var rename = require('gulp-rename');
var LessAutoprefix = require('less-plugin-autoprefix');
var autoprefix = new LessAutoprefix({ browsers: ['last 2 versions'] });

var uglify = require('gulp-uglify');
var pump = require('pump');

var babel = require('gulp-babel');

var path = require('path');


// 打包压缩 less
gulp.task('less', function() {
  return gulp.src('static/css/main.less', { base: '.' })
        .pipe(sourcemaps.init())
        .pipe(less({
          paths: [ path.join(__dirname, 'less', 'includes') ],
          plugins: [autoprefix]
        }))
        .pipe(sourcemaps.write('.'))
        .pipe(minify())
        .pipe(rename({
          suffix: '.min'
        }))
        .pipe(gulp.dest('./'));
})

gulp.task('js', function(cb) {
  pump([
    gulp.src(['static/js/partials/*.js', '!static/js/partials/*.min.js'], { base: '.' }),
    sourcemaps.init(),
    babel({
      presets: ['env']
    }),
    uglify(),
    sourcemaps.write('.'),
    rename({
      suffix: '.min'
    }),
    gulp.dest('./')
  ], cb)
})

gulp.task('commonjs', function(cb) {
  pump([
    gulp.src(['static/js/common.js', '!static/js/common.min.js'], { base: '.' }),
    sourcemaps.init(),
    babel({
      presets: ['env']
    }),
    uglify(),
    sourcemaps.write('.'),
    rename({
      suffix: '.min'
    }),
    gulp.dest('./')
  ], cb)
})


gulp.task('build', ['less', 'js', 'commonjs'], function() {
  return del([
    'static/css/**/*.map',
    'static/js/**/*.map'
  ]);
})

gulp.task('watch', ['less', 'js', 'commonjs'], function() {
  gulp.watch(['static/css/main.less', 'static/css/partials/*.less'], ['less']);
  gulp.watch(['static/common.js', 'static/partials/*.js'], ['js', 'commonjs']);
})

gulp.task('default', ['less', 'js', 'commonjs'], function() {
  gulp.watch(['static/css/main.less', 'static/css/partials/*.less'], ['less']);
  gulp.watch(['static/common.js', 'static/partials/*.js', '!static/partials/*.min.js'], ['js', 'commonjs']);
})
