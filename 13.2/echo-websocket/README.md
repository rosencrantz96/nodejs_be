# 메아리 애플리케이션 만들기: 웹소켓 
: 클라이언트에서 메시지를 보내면 서버에서 같은 메시지를 반환하는 프로그램 

> 1. `ws` 패키지 설치 
> 2. 서버 측 구축하기 (`server.js` 파일 작성 및 구동)
> 3. 클라이언트 측 구현하기 (`index.html` 파일 작성 )

## ws 패키지 설치 
`npm install ws`

→ web-socket 디렉토리 아래에 `package.json`과 `package-lock.json`, [node_modules] 디렉토리가 생긴다 

## 서버 구축하기: `server.js` 파일 작성 및 서버 구동 

```javascript 
const WebSocket = require('ws');    // ws 패키지 임포트
const server = new WebSocket.Server({ port: 3000 }); // 서버 인스턴스 생성 

server.on('connection', ws => {     // 서버 접속 이벤트 핸들러
    ws.send('[서버 접속 완료!]');   // 클라이언트 접속 시 클라이언트로 메시지를 보낸다 

    // 클라이언트에서 메시지가 수신된 경우의 이벤트 핸들러 
    ws.on('message', message => {
        ws.send(`서버로부터 응답: ${message}`);
    });

    ws.on('close', () => {      // 클라이언트 접속 종료 이벤트 
        console.log(' 클라이언트 접속 해제');
    });
});
```
1. ws의 `Server()` 함수를 사용하여 서버 인스턴스를 생성한 후 `server` 변수에 저장한다 

2. 웹소켓 서버(`server` 변수)의 `on()` 함수: 이벤트를 받는 함수 

> 첫 번째 인수 - 이벤트 유형 <br>
> 두 번째 인수 - 이벤트 발생 시 실행할 콜백 함수를 인수로 설정 

- 'connection' : 클라이언트가 접속 시 발생하는 이벤트 
- 콜백 함수의 인수로 `ws` → WebSocket 클래스의 인스턴스이다 

3. `ws.on()` 함수: 클라이언트에서 이벤트가 발생할 때 실행하는 함수 
> WebSocketServer와 마찬가지로 첫 번째 인수에 이벤트 유형, 두 번째 인수에 콜백 함수를 받는다 

4. `ws.on('close', 콜백함수)`: 클라이언트가 접속을 종료했을 때 실행 

### WebSocketServer의 이벤트
|이벤트|설명|
|---|---|
|`close`|서버가 close될 때 발생|
|`connection`|핸드셰이크가 완료되면 발생|
|`error`|서버에서 에러가 발생하면 발생|
|`headers`|응답의 헤더가 핸드쉐이크 소켓에 기록되기 전에 발생하는 이벤트<br>헤더를 보내기 전에 검사, 수정 가능|
|`listening`|서버가 바인딩되었을 때 발생|
|`wsClientError`|WebSocket이 연결되기 전에 에러가 나면 발생|

### WebSocket의 이벤트
|이벤트|설명|
|---|---|
|`close`|연결을 닫을 때 발생|
|`error`|에러가 나면 발생|
|`message`|메시지를 수신할 때 발생|
|`open`|서버와 연결이 되면 발생|
|`ping`|서버에서 ping을 수신하면 발생|
|`pong`|서버에서 pong을 수신하면 발생|
|`redirect`|리다이렉션을 하기 전에 발생|
|`unexpected-response`|서버 응답이 예상한 응답이 아닐 때(예:401 응답) 발생|
|`upgrade`|핸드셰이크의 일부로 서버로부터 응답 헤더가 수신될 때 발생<br>(Emitted when response headers are received from the server as part of the handshake)<br>이를 통해 서버에서 헤더(예: `set-cookie` 헤더)를 읽을 수 있다|

[ws api 문서](https://github.com/websockets/ws/blob/HEAD/doc/ws.md#event-upgrade)

## 클라이언트 측 구현하기: client.html
```html
<body>
    <!-- 메시지를 적을 텍스트 영역 -->
    <textarea id="message" cols="50" rows="5"></textarea>
    <br />

    <!-- 버튼 -->
    <button onclick="sendMessage()">전송</button>
    <button onclick="webSocketClose()">종료</button>
    <div id="messages"></div>
</body>
<script>
    // 웹소켓 연결
    const ws = new WebSocket('ws://localhost:3000');

    // send 함수로 메시지 발송
    function sendMessage() {
        ws.send(document.getElementById('message').value);
    }

    // 웹소켓 연결 종료
    function webSocketClose() {
        console.log("종료 누름");
        ws.close();
    }

    // WebSocket의 open 이벤트 핸들러
    ws.onopen = function() {
        console.log(" 클라이언트 접속 완료!");
    }

    // WebSocket의 message 이벤트 핸드러. 서버에서 메시지 수신 시 실행 
    ws.onmessage = function(event) {
        // 엔터 키를 <br /> 태그로 변경
        let message = event.data.replace(/(\r\n|\n|\r)/g, "<br />");
        let el = document.createElement('div');     // div 태그 생성
        el.innerHTML = message;     // <div>{메시지}</div> 값이 됨. HTML로 파싱
        el.className = 'message';   // <div class='message'>{메시지}</div> 값이 됨
        document.getElementById('messages').append(el) // messages 요소에 추가 
    }

    // 접속 종료 시 실행 
    ws.onclose = function(e) {
        console.log('종료');
        document.getElementById('messages').append("서버 접속 종료");
    }
</script>
```
1. 메시지를 넣는 영역은 textarea 태그 사용, `id`는 `getElementById`를 사용하는데 필요하다 

> $*$ `getElementById`: 주어진 id 값으로 해당 요소를 찾아서 해당 엘리먼트 객체를 가져오는데 사용 

2. 전송과 종료 버튼에는 각각 `sendMessage()`, `webSocketClose()` 함수를 바인딩 

3. 웹소켓 연결은 `new WebSocket(서버주소)`를 하면 맺어진다 (웹 브라우저에는 웹 소켓 기능이 이미 있기 때문에 별도로 라이브러리를 추가하지 않아도 된다)<br>
반환값으로 WebSocket의 인스턴스가 돌아오는데 해당 값을 ws 변수에 저장

4. 메시지 발송: 웹소켓 인스턴스 `ws`의 `send()` 함수 사용 
> 텍스트 영역의 값을 서버로 송신하고, 서버에서는 웹소켓의 message 이벤트가 발생한다 

5. 웹소켓 연결 종료: `ws.close()`

6. `ws.onopen` **open 이벤트 핸들러**: 서버와 연결되면 발생한다 → 서버와 연동되면 실행되며 ' 클라이언트 접속 완료' 문구가 브라우저의 콘솔에 찍히게 된다 

7. `ws.onmessage` **서버에서 메시지 수신 시 발생하는 이벤트 핸들러**
    - `replace(/(\r\n|\n|\r)/g, "<br />");` : 텍스트 영역에서 엔터를 입력하면 `\r\n`을 새 줄을 뜻하는 태그인 `<br />`로 변경하는 코드
    - 서버에서 입력받은 값을 화면에 나타내기 위해서는 HTML 태그로 표시해야 한다 → `document.createElement('div')` 함수를 사용하여 `div` 태그를 생성해 `el`에 저장
    - `el`은 `<div></div>` 영역이다 → `innerHTML` 함수로 `<div></div>` 사이에 태그를 넣을 수 있다 (값은 텍스트 영역에서 받은 메시지)
    - `el.className = 'message';`: 클래스명을 붙여준다 (`<div class='message'>{메시지}</div>`)
> `document.getElementById('messages').append(el)`: 만든 메시지를 HTML의 messages라는 id를 가진 요소에 추가해준다! 

8. 접속 종료: `ws.close` (콘솔에 '종료'가 찍히고 화면에는 서버 접속 종료가 찍힌다)

### 접속을 종료한 후 다시 전송 버튼을 눌러도 웹소켓은 다시 실행되지 않는다 
`WebSocket is already in CLOSING or CLOSED state.`