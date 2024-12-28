# my hugo site

## 开发环境准备(develop environment setup)

1. `docker pull hugomods/hugo:nightly-non-root`
2. `docker run -itd --name hugo -v $(pwd):/src -p 1313:1313 hugomods/hugo:nightly-non-root /bin/sh`

## 开发develop

theme: cleanwhite

## 使用github actions（pipeline）部署deploy

在github actions中配置将build后的静态public目录推送到ALIYUN ECS中nginx的目录中。
