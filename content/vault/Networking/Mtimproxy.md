#networking #tools #debugging #utilities  
You can use `mtimproxy` to set up a proxy to inspect all incoming traffic before it is being send to the target

```bash
$ mitmproxy --mode reverse:https://target.trendminer.net --listen-port=9000
$ curl localhost:9000
```