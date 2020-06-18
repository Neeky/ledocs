## 背景
由于 npm 的软件源(registry)在国外，这个使得 npm install 时经常超时；为了解决这个问题我们国内的镜像源，目前国内的两大云服务商都有镜像。

![sqlpy](static/2020-25/sqlpy-npm-registery.jpg)

---

## 使用腾讯云的 npm 源
配置 npm 使用腾讯云的 npm 源。
```bash
npm config set registry http://mirrors.cloud.tencent.com/npm/
```

---

如果你使用的是腾讯云的云服务器，走内网流量可能会更加适合你，这样就不用当心外网的流量费用了。
```bash
npm config set registry http://mirrors.tencentyun.com/npm/
```

---

## 使用阿里云的 npm 源
使用阿里云的源可以这样配置。
```bash
npm config set registry https://registry.npm.taobao.org
```

---