# xray 使用指南

**社区版非开源版，请直接到 [release](https://github.com/chaitin/xray/releases) 中下载使用。无需 clone 此仓库。**

**点击右上角 'Watch' 然后选择 'Release Only' 可以在新版本发布的时候收到通知，而不会被其他的消息（比如 issue、pr 等）所打扰。**

## 快速使用方式

1. 扫描单个 url
    
    ```bash
    xray webscan --url "http://example.com/?a=b"
    ```

1. 使用 HTTP 代理进行被动扫描
    
    ```
    xray webscan --listen 127.0.0.1:7777
    ```
   设置浏览器 http 代理为 `http://127.0.0.1:7777`，就可以自动分析代理流量并扫描。
   
   >如需扫描 https 流量，请阅读下方 `抓取 https 流量` 部分

1. 手动指定本次运行的插件
   
   默认情况下，将会启用所有内置插件，可以使用下列命令指定本次扫描启用的插件。
   
   ```bash
   xray webscan --plugins cmd_injection,sqldet --url http://example.com
   xray webscan --plugins cmd_injection,sqldet --proxy 127.0.0.1:7777
   ```
      
1. 指定插件输出

    可以指定将本次扫描的漏洞信息输出到某个文件中:
    
    ```
    xray webscan --url http://example.com/?a=b --output result.txt
    ```

   
## 进阶使用

### 配置文件

引擎初次运行时，会在当前目录内生成一个 `config.yaml` 文件，该文件中的配置项可以直接左右引擎在运行时的状态。通过调整配置中的各种参数，可以满足不同场景下的需求。在修改某项配置时，请务必理解该项的含义后再修改，否则可能会导致非预期的情况。下列进阶使用的方法均与该配置文件相关。

### 默认开启/关闭某个插件

插件配置中的 `plugins` 部分如下：

```yaml
plugins:
  cmd_injection:
    enabled: true
  sqldet:
    enabled: true
  dirscan:
    enabled: true
  redirect:
    enabled: false
  ...
```

通过修改`enabled` 项，可以默认开启/关闭某个插件。值得一提的是，通过 `--plugins` 指定的插件将会被强制启用。

### 抓取 https 流量

mitm 的配置项主要用于被动扫描模式下的代理的配置。

```yaml
mitm:
  ca_cert: ./ca.crt
  ca_key: ./ca.key
  includes:
    - "*"
  excludes:
    - "*google*"
```

配置项中的前两项： `ca_cert` 和 `ca_key` 用于指定中间人的根证书路径。和 burp 类似，抓取 https 流量需要信任一个根证书，这个根证书可以自行生成，也可用下列自带的命令生成:

```
xray genca
```


运行后将在当前目录生成 `ca.key` 和 `ca.crt`， 用户需要手动信任 `ca.crt`。操作完成后就可以正常抓取 https 流量了。

在 mitm 的配置部分还有两项配置值得注意：

1. `includes`表示只扫描哪些域。比如 `*.example.com` 只扫描 `example.com` 的子域
1. `excludes` 表示不扫描哪些域。比如 `t.example.com` 表示不扫描 `t.example.com`

两个都配置的情况下会取交集，这两个配置常用于想要过滤代理中的某些域，或者只想扫描某个域的请求时。默认配置为抓取所有的域。

### HTTP 配置

这里的配置主要影响到引擎的 http 发包，如有需求，参考 yaml 中注释进行对应的修改。

```yaml
http:
  dial_timeout: 5 # 建立 tcp 连接的超时时间
  read_timeout: 30 # 读取 http 响应的超时时间，不可太小，否则会影响到 sql 时间盲注的判断
  fail_retries: 1 # 请求失败的重试次数，0 则不重试
  max_qps: 500 # 每秒最大请求数
  max_redirect: 5 # 单个请求最大允许的跳转数
  max_conns_per_host: 50 # 同一 host 最大允许的连接数，可以根据目标主机性能适当增大。
  max_resp_body_size: 8388608 # 8M，单个请求最大允许的响应体大小，超过该值 body 就会被截断
  headers: # 每个请求预置的 http 头
    UserAgent:
      - Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_4) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.169
  cookies: # 每个请求预置的 cookie 值，效果上相当于添加了一个 Header: Cookie: key=value
    key: value
  allow_methods: # 允许使用 http 方法
    - HEAD
    - GET
    - POST
    - PUT
    - DELETE
    - OPTIONS
    - CONNECT
  tls_skip_verify: true # 是否验证目标网站的 https 证书。
```

## 讨论区

1. Github issue: https://github.com/chaitin/xray/issues
2. QQ 群: 717365081
3. 微信群: 扫描以下二维码加我的个人微信，会把大家拉到 `xray` 官方微信群    
 
<img src="./assets/wechat.jpg" height="150px">
