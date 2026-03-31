# 在 VS Code Remote SSH 中为 Codex 扩展配置远程代理教程

## 1. 适用场景

### 背景
在使用 VS Code 通过 Remote SSH 连接远程服务器时，`Codex` 相关扩展的实际运行位置通常在**远程服务器侧**。  
因此，即使本地电脑已经可以正常使用代理，远程服务器上的扩展仍然可能无法正常登录、拉取资源或调用接口。


适用于以下情况：

- 本地电脑可以正常科学上网并可以登陆codeX
- 使用 VS Code 的 **Remote SSH** 连接 Linux 服务器
- 远程服务器本身不能直接访问 OpenAI 相关服务
- 希望在远程环境中使用 ChatGPT / Codex / OpenAI 扩展

---

### 原理

```text
远程服务器上的 VS Code 扩展 -> 远程服务器 127.0.0.1:xxxx -> 本地电脑 127.0.0.1:yyyy -> 本地代理工具 -> OpenAI 服务
````
其中：`xxxx`：本地代理真实监听端口；`yyyy`：远程服务器使用的映射端口；`xxxx` 和 `yyyy` **不需要相同**

---

## 2. 具体步骤
### 第一步：确认本地代理端口

先在本地代理工具中确认真实代理端口。

例如在 Clash 等软件中：`HTTPs Port = 7890`

则说明本地实际代理端口是：`127.0.0.1:7890`

注意区分：

* `7890`：代理端口，即：`xxxx = 7890`
* `9090`：通常是控制面板/API 端口，不是给 `http_proxy` 使用的

### 第二步：配置 SSH 远程端口转发

编辑本地 `~/.ssh/config`：

```sshconfig
Host abc
    HostName 你的服务器IP或域名
    User xyz
    RemoteForward yyyy 127.0.0.1:xxxx
```

说明：在**远程服务器**上开放 `127.0.0.1:yyyy`，将该端口的数据转发到**本地电脑**的 `127.0.0.1:xxxx`

配置完成后，重新启动并连接

### 第三步：将本地 `~/.codex` 复制到远程服务器
当在本地 IDE 成功登录 Codex 插件后，插件会在你的用户目录下生成目录，在本地执行打包：

```bash
tar -cf codex.tar ~/.codex
```

然后将 `codex.tar` 上传到远程服务器家目录，例如：

```bash
scp codex.tar xyz@你的服务器地址:~/
```

在远程服务器解压

```bash
tar -xf ~/codex.tar -C ~/
```

### 第四步：在远程服务器中配置代理环境变量

在远程服务器 shell 中执行：

```bash
export http_proxy=http://127.0.0.1:yyyy
export https_proxy=http://127.0.0.1:yyyy
export HTTP_PROXY=http://127.0.0.1:yyyy
export HTTPS_PROXY=http://127.0.0.1:yyyy
source ~/.bashrc
```

### 第五步：在 VS Code 中设置远程工作空间代理

> 必须设置在 **Remote Settings** 中，而不是本地 Settings。

在 VS Code 中快捷键：`Cmd/Ctrl + Shift + P`

输入：`Open Remote Settings`

进入远程服务器的设置页面，搜索：`proxy`，将：`http-proxy` 设置为：`yyyy`

重启远程连接

## 检查清单

完成配置后，请逐项确认：

* [ ] 本地代理实际端口为 `xxxx`
* [ ] SSH 已配置 `RemoteForward yyyy 127.0.0.1:xxxx`
* [ ] 远程服务器上存在 `~/.codex`
* [ ] 远程 shell 中 `http_proxy` / `https_proxy` 为 `http://127.0.0.1:yyyy`
* [ ] VS Code 的 **Open Remote Settings** 中 `http-proxy` 设置为 `yyyy`
* [ ] 重新连接 Remote SSH 后，扩展可正常工作
