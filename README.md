### License

If this documention has denied, please contact L1uTongwei\<1347277058@qq.com\> to delete.

For easy, I will just write important information, no whole HTTP request.

### Bind Port

First EasyEDA listen http://127.0.0.1:3579 for address 0.0.0.0

### /api/whois

I use EasyEDA 6.5.40.

If EasyEDA will check local router, it will send a "whois" request first.

```
HTTP GET /api/whois?version=6.5.40 HTTP/1.1 
Full request URI: http://127.0.0.1:3579/api/whois?version=6.5.40
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) LCEDA/6.5.40 Chrome/102.0.5005.167 Electron/19.1.9 Safari/537.36 EasyEDA-Editor/6.5.40 (Online Mode)
Origin: https://lceda.cn
```

Then the server return:

```
HTTP/1.1 200 OK
With Data: 45617379454441204175746f20526f75746572 ("EasyEDA Auto Router")
```

And then the EasyEDA will know this is an available router.

### /router

/router API will let server switch protocol to Websocket.

So this is a Websocket URI: ws://127.0.0.1:3579/router

First EasyEDA will send a /router request with:

```
HTTP GET /router HTTP/1.1 
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: (Hide for different values)
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

Then Router will return a 101.

```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Sec-WebSocket-Accept: (Hide for different values)
```

### Websocket Connection

#### heartbeat

Sometimes the EasyEDA will send message `{"a": "heartbeat"}` to check if connection is alive.

You don't need to reply this.

#### startRoute

EasyEDA use a `startRoute` message to let router work.

Message:

```json
{
    "a": "startRoute",
    "data": "", //This is a standrand .DSN file
    "timeout": 300, 
    "progressInterval": 2,
    "optimizeTime": 3
}
```

#### routingProgress

EasyEDA receive a `routingProgress` message to update routing progress.

Message:

```json
{
    "a": "routingProgress",
    "inCompleteNetNum": 13, //Incomplete Net Number
    "data": {
        "1": { // Just a index (Why not array?)
            "wires": [
                {
                    "layerid": 1,
                    "width": 0.788,
                    "points": [
                        56.204,
                        -175.245,
                        55.329,
                        -176.12
                    ]
                },
                //...
            ],
            "vias": [
                {
                    "x": 56.204,
                    "y": -175.245
                }
            ],
            "net": "L1_1" // Net name
        },
        //...
    }
}
```

Every `progressInterval` times route you must report it with `routingProgress` message.

If a progress's deadline `timeout` reached, EasyEDA will end it.

#### routingResult

if routing is completed, a `routingResult` message must send to EasyEDA.

Message:

```json
{
    "a": "routingResult",
    "inCompleteNetNum": 0,
    "complete": 1,
    "data": "" //The same as `routingProgress` data
}
```
