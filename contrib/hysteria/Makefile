HYSTERIA_COMMIT = fb25dc852ebbc70658e67889f411887a82b7c281

ROOT := $(abspath ../..)
HY_DIR = $(ROOT)/build/hysteria

PAYLOAD ?= 1400
SERVER ?= jp
SERVICE ?= hysteria-8444

# 本仓库（dev 分支）就是打补丁后的 quic-go，hysteria 通过 go.mod replace 指向仓库根
fetch:
	@test -d $(HY_DIR) || { \
		mkdir -p $(ROOT)/build && \
		curl -sL https://codeload.github.com/HyNetworks/hysteria/tar.gz/$(HYSTERIA_COMMIT) | tar xz -C $(ROOT)/build && \
		mv $(ROOT)/build/hysteria-$(HYSTERIA_COMMIT) $(HY_DIR) && \
		echo "replace github.com/apernet/quic-go => ../../.." >> $(HY_DIR)/app/go.mod; }

build: fetch
	cd $(HY_DIR)/app && GOWORK=off CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
		go build -trimpath -ldflags "-s -w" -o $(ROOT)/dist/hysteria .

deploy: build
	scp -q $(ROOT)/dist/hysteria $(SERVER):/tmp/hysteria-new
	ssh $(SERVER) 'set -e; \
		cp -a /usr/local/bin/hysteria /usr/local/bin/hysteria.bak-$$(date +%Y%m%d); \
		install -m 755 -o root -g root /tmp/hysteria-new /usr/local/bin/hysteria; \
		rm -f /tmp/hysteria-new; \
		grep -q "^Environment=QUIC_GO_MAX_PAYLOAD" /etc/systemd/system/$(SERVICE).service || \
			sed -i "/^Environment=/i Environment=QUIC_GO_MAX_PAYLOAD=$(PAYLOAD)" /etc/systemd/system/$(SERVICE).service; \
		systemctl daemon-reload; systemctl restart $(SERVICE); sleep 2; systemctl is-active $(SERVICE)'
