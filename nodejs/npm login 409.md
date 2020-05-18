## npm login 报错
在登录 npm 时报错了。
```bash
npm login
Username: neeky
Password: 
Email: (this IS public) neeky@live.com
npm ERR! code E409
npm ERR! 409 Conflict - PUT https://registry.npm.taobao.org/-/user/org.couchdb.user:neeky - [conflict] User neeky already exists

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/jianglexing/.npm/_logs/2020-05-18T08_06_09_184Z-debug.log
```
前几天还好好的，因为官方的源安装软件比较慢，所以换成了阿里的源。从报错信息来看是把我的登录信息提交到了阿里的源，难不成真是这个问题？ google 了一下还真是这个原因。

---


## 解决方案
npm 的配置文件在 `~/.npmrc` 所以我要把 registry 的指向改回去(官方)。
```
//registry=https://registry.npm.taobao.org
registry=http://registry.npmjs.org/
```
让 registry 指向官方 `http://registry.npmjs.org/`。

---


## 检查
再次登录成功了。

```bash
npm login
Username: neeky
Password: 
Email: (this IS public) neeky@live.com
Logged in as neeky on http://registry.npmjs.org/.
```

---