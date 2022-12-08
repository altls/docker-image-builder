build:
	docker build -t arnoluo/nginx-php-node:alpine3.16 .

# 境外服务器构建
outerbuild:
	docker build \
		--build-arg ALPINE_MIRROR="dl-cdn.alpinelinux.org" \
		-t arnoluo/nginx-php-node:alpine3.16 .

run:
	docker run -it --name npnac arnoluo/nginx-php-node:alpine3.16 /bin/sh
