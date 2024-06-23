### License

If this documention has denied, please contact L1uTongwei\<1347277058@qq.com\> to delete.

This documention is for EasyEDA 6.5.40.

### Bind Port

First EasyEDA Auto-router listen port 3579 for address 0.0.0.0

### GET /api/whois (Check if Auto-router online)

If EasyEDA will check local router, it will send a "whois" request first.

EasyEDA will send a GET request like: `/api/whois?version=6.5.40`.

The query argument `version` is the version of EasyEDA.

If the EasyEDA's version is too low, you may refuse the request to make EasyEDA think no auto-router vaild.

```
HTTP GET /api/whois?version=6.5.40 HTTP/1.1 
Full request URI: http://127.0.0.1:3579/api/whois?version=6.5.40
```

Then the server return a string with HTTP data:

```
EasyEDA Auto Router
```

And then the EasyEDA will know this is an available auto-router.

### GET /router (Websocket Connection)

/router is a Websocket connection between EasyEDA & auto-router.

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

This is a standrand handshake of Websocket, to build a vaild Websocket connection.

See https://websockets.spec.whatwg.org//#websocket-opening-handshake.

#### in created Websocket Connection

##### heartbeat

Sometimes the EasyEDA will send message `{"a": "heartbeat"}` to check if connection is alive.

You don't need to reply this.

##### startRoute

EasyEDA use a `startRoute` message to let router work.

A `startRoute` massage set some option in auto-routing progress.

`data` is a standrand .DSN format string, to provide a to-be-routed board design.

EasyEDA doesn't have any settings to change the value of `timeout`, `progressInterval` & `optimizeTime`, So I cannot sure what these are.

In fact, they are never changed for all boards.

Message:

```json
{
    "a": "startRoute",
    "data": "",
    "timeout": 300, 
    "progressInterval": 2,
    "optimizeTime": 3
}
```

##### routingProgress

EasyEDA receive a `routingProgress` message to update routing progress.

`data` here is to let EasyEDA real-time display.

`data` is a array, but in object format (index is field name).

Every element in `data` (mean a net) has three childs: `wires`, `vias` & `net`.

`net` is the net's name.

`vias` is a (x, y) array, to locate every vias' location.

Sure, vias' diameters informations are in .DSN design rules.

`wires` is a wires' array.

A wire has three childs: `layerid` (Layer's ID), `width` (wire width), `points`.

I am not a professional at SVG, but I guess `points` is a path (maybe).

`inCompleteNetNum` is incomplete nets for now.

Message:

```json
{
    "a": "routingProgress",
    "inCompleteNetNum": 13,
    "data": {
        "1": {
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
            "net": "L1_1"
        },
        //...
    }
}
```

##### routingResult

if routing is completed, a `routingResult` message must send to EasyEDA.

The `data` here is the same as `startRoute`.

`complete` is for success(1) or failure(0).

`inCompleteNetNum` is incomplete nets if failure.

Message:

```json
{
    "a": "routingResult",
    "inCompleteNetNum": 0,
    "complete": 1,
    "data": "" //The same as `routingProgress` data
}
```

EasyEDA through `data` to get the route result.

To convient all users, I upload the network activities in this repo.
