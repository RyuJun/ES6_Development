# webpack과 babel을 이용한 ES6 개발환경 설치

- webpack : client-side ㅗㅁ듈 빌더이자 모듈 로더
- npm : 패키지 매니저
- babel : ES6를 ES5로 변환시켜주는 트랜스파일러

## 1. webpack 설명

webpack의 특성은 은 다음과 같다.
- 지원하는 모듈 포맷: AMD, CommonJS
- 지원하는 패키지 매니저: Bower, npm
- non-code를 위한 로더: CSS, templates, ...
- 온디멘드 로딩 (chunked transfer)
- 자동으로 브라우저를 새로고침 해주는 hot module reloading을 지원하는 자체 개발 서버지원
- Dead code elimination


## 2. 데모 프로젝트 설치
- GitHub Repository로 이동 후 clone [ES6_Development](https://github.com/RyuJun/ES6_Development)
- `cd ES6_Development/`
- `npm install`

## 3. 데모 프로젝트의 구조
```bash
build/
    bundle.js
    bundle.js.map
    index.html
html/
    index.html
js/
    main.js
    world.js
node_modules/
package.json
webpack.config.js
```
설정 파일과 라이브러리:
- `packge.json` npm을 설정하는 역할을 한다. `package.json`내용을 참고하여 npm은 모든 dependencies들을 설치 한다.
- `node_modules/` npm이 package들을 설치하는 디렉토리 폴더
- `webpack.config.js` webpack을 설정한다. 

Input 디렉토리:
- `js/` ES6 코드를 지니는 디렉토리, webpack이 ES5로 트랜스파일할 파일이 위치한다.
- `html` 수정되지 않은 채 output으로 내보낼 파일들을 지닌 디렉토리

Output 디렉토리:
- `build/` webpack의 결과물이 위치할 디렉토리. 결과물은 `webpack, babel, npm`에 기대지 않는 `HTML,ES5 코드` 이다. 아래 리스트는 빌드 순서, 스텝에 따라 만들어진다.
- `bundle.js` 모든 자바스크립트의 묶음
- `bundle.js.map` 브라우저가 ES5 코드를 작동시킬 수 있도록 가능하게 해주는 source map이다. 디버그는 ES6로 한다.
- `index.html` 는 `html/` 에서 복사해 가져온 것이다.

## 4. 파일 패키지 package.json
package.json은 npm 설정을 관리한다.
```js
{
  "devDependencies": {
    "babel-loader": "^6.1.0",
    "babel-preset-es2015": "^6.1.18",
    "babel-polyfill": "^6.3.14",
    "webpack": "^1.12.6",
    "webpack-dev-server": "^1.12.1",
    "copy-webpack-plugin": "^0.2.0"
  },
  "scripts": {
    "build": "webpack",
    "watch": "webpack --watch",
    "start": "webpack-dev-server --hot --inline"
  },
  "babel": {
    "presets": [
      "es2015"
    ]
  },
  "author": "Axel Rauschmayer",
  "license": "MIT",
  "dependencies": {
    "babel-core": "^6.26.0"
  }
}
```
`devDependencies`는 개발에 필요한 npm package를 의미한다.
- `webpack`은 webpack을 로컬에 설치한다.
- `webpack-dev-server`는 webpack에 hot-reloading 개발 웹서버를 추가한다.
- `copy-webpack-plugin`은 build 디렉토리에 파일을 복사하는 webpack 플러그인이다.
- `babel-loader`는 webpack이 Babel을 이용하여 자바스크립트를 트랜스파일 하도록 하는 기능을 제공한다. 이패키지는 내부적으로 몇가지 핵심적인 babel 패키지를 가지고 온다.
- `babel-preset-es2015`은 ES6를 ES5로 컴파일하기 위해 쓰이는 Babel 프리셋 이다.

`scripts`에는 webpack을 작동하는 다양한 방법을 적어놓는다.
- 한번만 Build
	- `npm run build`
	- 웹 브라우저에서 `build/index.html`을 연다
- 파일(들)을 감시~watch~한다. 그 중 하나가 바뀌면 rebuild한다.
	- `npm run watch`
	- 웹 브라우저에서 `build/index.html`을 연다. 변경사항이 있으면 웹페이지를 새로고침한다.
- webpack 개발 서버를 이용한 hot reloading
	- `npm start` (`npm run start` 의 줄임 )
	- `http://loacalhost:8080/` 을 연다. 변경사항이 있으면 자동으로 새로고침된다.

`babel`은 Babel이 프리셋 중에서`es2015`를 사용하도록 알려준다. 

## 5. Directory js/
두 개의 작은 파일이 모든 자바스크립트 코드를 지니고 있다.
먼저, `js/main.js`
```
import 'babel-polyfill';
import world from './world';

document.getElementById('output').innerHTML = `Hello ${world}!`;
```
`babel-polyfill`을 가져오는 첫번째 줄을 통하여 ES6 기본 library를 global 변수로 가지고 온다. (Object.assing(), 새로운 ES6 string 메소드 등)

둘째로는 `js/world.js`
```
exoprt defualt 'WORLD';
```

build 과정 중에 webpack은 이 파일들을 모두 모아 하나의 파일, `build/bundle.js`로 묶는다. 그리고 그파일을 위한 source map을 `build.js.map`이라는 파일로 만든다.

## 6. File html/index.html
HTML파일은 `<script>` 엘리먼트를 통하여 bundel을 실행한다.
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>webpack Babel demo</title>
</head>
<body>

<h1>webpack Babel demo</h1>

<div id="output"></div>

<script src="bundle.js"></script>
</body>
</html>
```
## 7. webpack.config.js
다음은 webpack을 위한 설정 파일에서 발췌한 것이다.(몇가지 import와 세부사항을 제외하였다.)
```
var dir_js = path.resolve(__dirname, 'js');
var dir_html = path.resolve(__dirname, 'html');
var dir_build = path.resolve(__dirname, 'build');

module.exports = {
    entry: path.resolve(dir_js, 'main.js'),
    output: {
        path: dir_build,
        filename: 'bundle.js'
    },
    module: {
        loaders: [
            {
                loader: 'babel-loader',
                test: dir_js,
            }
        ]
    },
    plugins: [
        // Simply copies the files over
        new CopyWebpackPlugin([
            { from: dir_html } // to: output.path
        ]),
    ],
    // Create Sourcemaps for the bundle
    devtool: 'source-map',
    devServer: {
        contentBase: dir_build,
    },
};
```
해당 파일은 native Node.js 모듈로서 설정에 맞춰서 object를 export하는 역할을 맡는다. 파일은 `__dirname`이라는 독특한 Node.js변수를 사용하는데,
이는 현재 작동되는 module을 담고있는 parent 디렉토리 주소를 지니고 있다. 설정 파일은 다음과 같은 properties를 갖고 있다.
- `entry` : 자바스크립트의 코드가 시작하는 부분을 의미한다. webpack은 이곳을 시작으로 dependency를 컴파일 해 나간다.
- `output` : webpack은 모든 entry file과 그 dependency를 묶은다음에 `bundle.js`라는 output파일로 출력한다.
- `module.loaders` : 는 preprocessor(진처리기), 즉 webpack 전에 미리 하는 예비 프로세서 이다. ES6를 위한 준비는 `bundle-loader`라 하는 모듈로더를 통해 제공된다.
    - `test` property는 loader가 어떤 파일을 트랜스파일해야 하는가를 지정해준다. 한가지 또는 여러개의 test를 지정할 수 있다.
    - Multiple tests : 개별 test의 배열 ("and")
- `plugins` : webpack의 기능을 연장해준다. 본인은 `CopyWebpackPlugin`을 이용하여 `html/`하에 있는 모든 파일을 `build/`로 복사한다. 이것은 기존에 `grunt`나 `gulp`가 수행하던 일을 webpack이 여기서 (플러그인을 통해) 수행한다는 뜻이기도 하다.
- `devtool` 옵션은 'source-map' 옵션을 키고, 이를 설정한다.
- `devServer`는 webpack이 개발서버가 어떤 파일을 맡아야 하는지 알려준다.

