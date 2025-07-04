# DnD Mobile

## 설명

디앤디모바일 홈페이지 퍼블리싱 파일

## 시작하기

### 설치

1. Clone the repo
```
git clone https://github.com/dndmobilePub/dndmobile.git
```
2. Install NPM packages
```
npm install
```
```
npm install gulp
```
```
npm install -g gulp
```
```
npm install --save-dev gulp
```
```
npm link gulp
```
```
npm link gulp-file-include gulp-sourcemaps gulp-concat gulp-uglify gulp-rename gulp-sass sass gulp-postcss cssnano autoprefixer browser-sync
```

3. Local start
```
gulp
```

### Node.js v22.16.0 기준 Gulp 재설치 전체 절차 (Git Bash용)

① 기존 환경 정리
```
rm -rf node_modules
rm -f package-lock.json
npm cache clean --force

```

② 호환 가능한 패키지 설치
```
npm install --save-dev \
  gulp \
  gulp-file-include \
  gulp-sourcemaps \
  gulp-concat \
  gulp-uglify \
  gulp-rename \
  gulp-dart-sass \
  sass \
  gulp-postcss \
  cssnano \
  autoprefixer \
  browser-sync

```

③ gulpfile.js 수정 (최신 구조 반영)
```
const gulp = require('gulp');
const fileinclude = require('gulp-file-include');
const sourcemaps = require('gulp-sourcemaps');
const concat = require('gulp-concat');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');
const scss = require('gulp-dart-sass'); // ✅ Node 22 호환
const postcss = require('gulp-postcss');
const cssnano = require('cssnano');
const autoprefixer = require('autoprefixer');
const browserSync = require('browser-sync').create();

const src = './src';
const dist = './dist';
const paths = {
  html: src + '/html/**/*.html',
  image: src + '/assets/images/*',
  font: src + '/assets/font/*',
  js: src + '/assets/js/**/*.js',
  scss: src + '/assets/scss/**/*.scss'
};

// HTML include
gulp.task('fileinclude', function () {
  return gulp.src([
    './src/html/**/*.html',
    '!./src/html/include*'
  ])
    .pipe(fileinclude({
      prefix: '@@',
      basepath: '@file'
    }))
    .pipe(gulp.dest('./dist/html'))
    .pipe(browserSync.reload({ stream: true }));
});

// 이미지 복사
gulp.task('images', function () {
  return gulp.src(paths.image)
    .pipe(gulp.dest('dist/assets/images'))
    .pipe(browserSync.reload({ stream: true }));
});

// JS 복사 및 minify
gulp.task('js', function () {
  return gulp.src(paths.js)
    .pipe(gulp.dest('dist/assets/js'))
    .pipe(uglify())
    .pipe(rename({ suffix: '.min' }))
    .pipe(browserSync.reload({ stream: true }));
});

// SCSS 컴파일
gulp.task('scss', function () {
  return gulp.src(paths.scss)
    .pipe(sourcemaps.init())
    .pipe(scss({ outputStyle: 'expanded' }).on('error', scss.logError))
    .pipe(postcss([autoprefixer(), cssnano()]))
    .pipe(sourcemaps.write('./maps'))
    .pipe(gulp.dest('dist/assets/css'))
    .pipe(browserSync.reload({ stream: true }));
});

// 브라우저 동기화
gulp.task('browserSync', function (done) {
  browserSync.init({
    server: {
      baseDir: './',
      directory: true
    },
    startPath: 'dist/html/index.html',
    browser: 'chrome'
  });

  gulp.watch('./sass/**/*.scss', gulp.series('scss'));
  gulp.watch('./*.html').on('change', browserSync.reload);
  gulp.watch('./js/**/*.js').on('change', browserSync.reload);

  done();
});

// 파일 변경 감지
gulp.task('watch', function () {
  gulp.watch(paths.html, gulp.series('fileinclude'));
  gulp.watch(paths.js, gulp.series('js'));
  gulp.watch(paths.scss, gulp.series('scss'));
});

// 기본 작업
gulp.task('default', gulp.parallel('fileinclude', 'scss', 'js', 'images', 'watch', 'browserSync'));
```

④ 실행
```
npx gulp

```

### 배포
1. src 폴더에서 수정 후 local 확인 필수 (현행화 필수)
2. ftp_upload 폴더에서 한번 더 수정 후 ftp upload 진행 (ftp_upload 파일에서 진행하는 이유 : 폴더 구조 변경으로 인한 경로 수정)
3. project.html 변경시 data-content 경로 수정 필수! (data 호출하는 레이어팝업) 
4. /(루트 폴더)에 html 파일 upload
5. dndmobile 폴더에 assets 파일 업로드
