# hysteria 构建说明

`dev` 分支 = 上游 `apernet/quic-go@a10df75c` + 一个提交，用于给 hysteria2 的服务端限制 QUIC 发送包大小上限。

## 改动

`connection.go` 里三处 `maxPacketSize` 初始化改为经 `capMaxPacketSize` 钳制，读取环境变量 `QUIC_GO_MAX_PAYLOAD`（0 或不设 = 不限制）。

用途：跨境链路存在单向路径 MTU 黑洞（实测 1464），QUIC 按 peer 声明的 1452 发包会被静默丢弃且 DPLPMTUD 无法自愈，表现为 hy2 长连接间歇超时。服务端设 `QUIC_GO_MAX_PAYLOAD=1400` 即可，客户端无需改动。

## 构建

```sh
cd contrib/hysteria
make build                 # 产出 ../../dist/hysteria（linux/amd64）
make deploy SERVER=jp      # 部署并重启 hysteria-8444
```

`build/hysteria`（仓库根下的 `build/`，已 gitignore）是固定版本的 hysteria 源码（`HYSTERIA_COMMIT`），其 `app/go.mod` 通过 `replace github.com/apernet/quic-go => ../../..` 指向本仓库。

回滚：服务端 `/usr/local/bin/hysteria.bak-<日期>` 换回，并去掉 unit 里的 `Environment=QUIC_GO_MAX_PAYLOAD`。
