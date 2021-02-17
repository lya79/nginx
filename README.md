# 安裝
http://nginx.org/en/download.html

# 啟動
```shell
C:\Users\USER\workspace\nginx\nginx-1.18.0> .\nginx.exe
```

# 指令
```shell
nginx.exe -s signal
signal 值可以是：

stop： fast shutdown
quit： graceful shutdown
reload：reloading the configuration file
reopen：reopening the log files
```

## 查詢 nginx.conf設定檔案所在位置
```shell
.\nginx.exe -t  
```

# Nginx Load Balancer
* round-robin：標準(預設)輪詢方式
* least-connected：當連線進來時會把 Request 導向連線數較少的 Server
* ip-hash：依據 Client IP 來分配到不同台 Server

## least_conn
將請求發送到有效連接數量最少的伺服器。

```shell
upstream backend {
    least_conn;
    server backend1.example.com;
    server backend2.example.com;
}
```

## ip_hash
根據客戶端 IP 地址決定請求發送到哪台伺服器。在這種情況下，使用 IPv4 地址的前三個八位字節或整個 IPv6 地址來計算散列值。該方法保證來自同一地址的請求到達同一個伺服器，除非它不可用。

```shell
upstream backend {
    ip_hash;
    server backend1.example.com;
    server backend2.example.com;
}
```

## 依據權重(搭配 round-robin)
使用 round-robin 策略時，默認情況下，Nginx 會根據權重在伺服器組中分發請求。server 指令的 weight 參數用於設置一台伺服器的權重，默認是 1。
這個例子中，backend1.example.com 的權重是 3，另外兩台伺服器使用默認權重 1，但是 IP 地址是 192.0.0.1 的伺服器被標記為備份伺服器，只有在其他伺服器都不可訪問時才會接受請求。在這個權重配置下，每六個請求會有五個發到 backend1.example.com。

```shell
upstream backend {
    server backend1.example.com weight=3; # 每5個 request會有 3個執行這個伺服器
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

# 臨時移除某個伺服器
如果其中一台伺服器需要臨時移除，則可以使用 ***down*** 參數進行標記，以保留客戶端IP位址的當前散列。由該伺服器處理的請求會自動發送到組中的下一台伺服器。

```shell
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

# 備援伺服器
backup 不能和ip_hash一起使用，backup 代表，所有伺服器都掛掉之後，此伺服器才會生效。

```shell
upstream backend {
    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
}
```

# 被動健康監測
當 Nginx 認為一台伺服器不可用時，會暫時停止將請求發送到這台伺服器，直到 Nginx 認為這台伺服器可用才會繼續。server 指令的以下參數配置了伺服器在哪些情況下不可用：

fail_timeout ：設置時間段，這段時間內如果發生指定數量的失敗嘗試，則認為伺服器在這段時間內不可用（sets the time during which the specified number of failed attempts should happen and still consider the server unavailable.）。換句話說，fail_timeout 參數設置伺服器不可用的區間。
max_fails ：設置失敗次數，在指定時間段內失敗這個次數時認為伺服器不可用。
默認設置是 10 秒鐘失敗 1 次就認為伺服器不可用。此時 Nginx 只要有一個請求發送失敗或沒有應答，就認為伺服器在 10 秒之內不可用。示例：

```shell
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    server backend3.example.com max_fails=2;
}
```

# 主動健康監測
參考
* http://ningg.top/nginx-series-health-check/

```shell
upstream backend {
    zone backend 64k;
    server backend1.example.com;
    server backend2.example.com;
}

location / {
    proxy_pass http://backend;
    health_check interval=10 fails=3 passes=2;
}
```

# location詳細用法
參考
* https://segmentfault.com/a/1190000013267839

# 限制 IP
參考: 
* https://blog.gtwang.org/linux/nginx-restricting-access-authenticated-user-ip-address-tutorial/
* https://cjk.aiao.today/nginx_internal_ip_acc/

# 參考來源
* https://kknews.cc/zh-tw/code/lra2jy9.html
* https://nginx.org/en/docs/http/load_balancing.html
* http://bigpxuan.blogspot.com/2018/07/nginx-server-location.html
* https://segmentfault.com/a/1190000013267839
* https://www.cnblogs.com/woshimrf/p/nginx-config-location.html
* http://ningg.top/nginx-series-health-check/
* https://blog.gtwang.org/linux/nginx-restricting-access-authenticated-user-ip-address-tutorial/
* https://cjk.aiao.today/nginx_internal_ip_acc/