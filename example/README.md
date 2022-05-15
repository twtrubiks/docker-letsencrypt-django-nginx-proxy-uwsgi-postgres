# 說明

測試 domain

```cmd
❯ curl -H "Host: whoami.A.local" localhost
I'm b5ffefea7f65
❯ curl -H "Host: whoami.B.local" localhost
I'm b5ffefea7f65
❯ curl -H "Host: whoami.C.local" localhost
I'm b5ffefea7f65
```

測試 `foo.com` 中的 `/nginx_status`

```cmd
❯ curl -H "Host: foo.com" localhost/nginx_status
Active connections: 1
server accepts handled requests
 10 10 10
Reading: 0 Writing: 1 Waiting: 0
```

測試 domain 中 block http_user_agent,

```cmd
❯ curl -H "Host: whoami.A.local" -I -A 'mj12bot' localhost
curl: (52) Empty reply from server

❯ curl -H "Host: whoami.B.local" -I -A 'mj12bot' localhost
HTTP/1.1 410 Gone
Server: nginx/1.21.6
Date: Sun, 15 May 2022 03:22:28 GMT
Content-Type: text/html
Content-Length: 143
Connection: keep-alive

❯ curl -H "Host: whoami.C.local" -I -A 'mj12bot' localhost
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Sun, 15 May 2022 03:25:04 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 17
Connection: keep-alive
```