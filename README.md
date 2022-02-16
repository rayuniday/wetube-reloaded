### #1.1 Requirements

1. Browser : Brave
2. Visual Studio Code
3. NodeJS : [nodejs.org](nodejs.org) - LTS 버전 다운로드

```
node -v
```

### #1.3 What is NodeJS

브라우저 밖에서 돌아가는 자바스크립트

### #1.4 What is NPM

자바스크립트 언어를 위한 패키지 매니저
yarn이라는 패키지 매니저도 있는데 차의가 거의 없음
[https://www.npmjs.com/package/express](https://www.npmjs.com/package/express)

### #2.0 Your First NodeJS Project

npm init : package.json

- package name : wetube
- version: 1.0.0
- description : The best way to watch videos.
- entry point: index.js
- test command:
- git repository: (https://github.com/rayuniday/wetube-reloaded)
- keywords:
- author:
- license: MIT
- Is this OK? yes

### #2.1 Installing Express

package.json

```json
"scripts": {
  "win": "node index.js"
}
```

npm run win  
npm i express

### #2.2 Understanding Dependencie

npm i
package.json과 index.js만 필요
package-lock.json이 있으면 해당 버전 그대로 다운로드
.gitignore : /node_modules

### #2.3 The Tower of Babel

```javascript
const express = require("express");

const app = express();
```

[babeljs.io](babeljs.io) : 최신 자바스크립트를 NodeJS가 이해하는 자바스크립트로 안전하게 컴파일해서 변환
[https://babeljs.io/setup#installation](https://babeljs.io/setup#installation) - Node

```
npm install --save-dev @babel/core
```

dependencies : 프로젝트를 실행하기 위해 필요한 dependencies
devDependencies : 개발자에게만 필요

babel.config.json

```json
{
  "presets": ["@babel/preset-env"]
}
```

```
npm install @babel/preset-env --save-dev
```

preset을 이용 최신 자바스크립트를 사용

### #2.4 Nodemon

```
npm install @babel/node --save-dev
```

```json
"scripts": {
  "dev": "babel-node index.js"
}
```

babel-node는 @babel/node를 설치했기에 사용 가능
npm run dev

nodemon을 설치하면 수정되는 걸 감시

```
npm i nodemon --save-dev
```

```json
"scripts": {
  "dev": "nodemon --exec npx babel-node index.js"
}
```

### #3.0 Your First Server

src/server.js

```json
"scripts": {
  "dev": "nodemon --exec npx babel-node src/server.js"
}
```

```js
import express from "express";

const PORT = 4000;

const app = express();

const handleListening = () =>
  console.log(`Server listening on port http://localhost:${PORT}`);

app.listen(PORT, handleListening);
```

### #3.1 GET Requests

서버끼리 의사소통 하는 방법이 http
http request는 웹사이트에 접속하고 서버에 정보를 보내는 방법
GET은 http method 중 하나
브라우저는 웹사이트를 request하고, 페이지를 가져다 준다

### #3.2 GET Requests part Two

```js
const handleHome = () => console.log("Somebody is trying to go home.");
app.get("/", handleHome);
```

### #3.3 Responses

req : request object - 브라우저가 뭔가를 요청, 쿠키나 method 같은 정보를 얻을 수 있음
res : response object - 응답

- res.end() : 그냥 끝내버림, request 종료
- res.send() : 메세지를 보냄, 응답

```js
const handleHome = (req, res) => {
  return res.send("I still love you.");
};
const handleLogin = (req, res) => {
  return res.send("Login here.");
};

app.get("/", handleHome);
app.get("/login", handleLogin);
```

### #3.4 Recap

```js
res.send("<h1>I still love you.</h1>");
res.send({ message: "Login here." });
```

### #3.5 Middlewares part One

브라우저가 뭔가를 request하면, 서버는 거기에 응답해준다
middleware : middle software - 중간에 있는 소프트웨어

- middleware는 reqest와 response 사이에 있음
- 브라우저가 request한 다음, 그리고 응답하기 전. 그 사이에 middleware

모든 middleware는 handler, 모든 handler는 middleware
handler는 controller
모든 controller는 middleware

controller에는 두 개의 argument 말고도 하나가 더 있음 : next
next argument : 다음 함수를 호출

```js
//middleware
const gossipMiddleware = (req, res, next) => {
  console.log(`Someone is going to: ${req.url}`);
  next();
};

//controller
const handleHome = (req, res) => {
  return res.send("I love middlewares");
};

app.get("/", gossipMiddleware, handleHome);
```

함수가 next 함수를 호출한다면, 이 함수는 middleware라는 걸 의미
get은 path를 필요, path는 URL
hanler는 다수의 handler를 사용할 수 있음

middleware

- request에 응답하지 않고, request를 지속
- 작업을 다음 함수에게 넘기는 함수
- 필요한 만큼 만들 수 있음
- 사람들이 웹사이트의 어디를 가려는지 알려줌 (req.url)

### #3.6 Middlewares part Two

middleware를 use하는 게 먼저오고, 그다음에 URL의 get이 와야 해
middleware를 위에다 두면, 모든 route에 적용되는 거야

req.method, req.url : 어떤 method가 어느 url로 향하는지 알 수 있어.
middleware를 app 전체에 어떤 url에서도 사용할 수 있도록 할 수도 있고,
middleware 하나의 url에만 사용되게 할 수도 있어
어떤 middleware는 next 함수를 호출하고, 어떤 middleware는 res.send를 사용하지

```js
const logger = (req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
};

const privateMiddleware = (req, res, next) => {
  const url = req.url;
  if (url === "/protected") {
    return res.send("<h1>Not Allowed</h1>");
  }
  console.log("Allowed, you may continue.");
  next();
};

const handleProtected = (req, res) => {
  return res.send("Welcome to the private lounge.");
};

app.use(logger);
app.use(privateMiddleware);
app.get("/protected", handleProtected);
```

### #3.7 Setup Recap

### #3.8 Servers Recap

### #3.9 Controllers Recap

### #3.10 Middleware Recap

### #3.11 External Middlewares
