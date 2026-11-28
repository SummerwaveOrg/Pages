# Summerwave 使用手册

### Welcome to Wave World!

欢迎使用 Summerwave！使用前建议你阅读下面的手册：

### 基本命令

`wave` 的基本命令你可能比较熟悉，为了让你更快了解，我们提供了你可能常用的参数命令组合：

| 组合 | 功能 | 什么时候用 | 备注 |
| --- | --- | --- | --- |
| `wave -h` | 显示帮助 | 不确定某个命令怎么用 | `wave -h --zh` 显示中文帮助 |
| `wave manual --color` | 以彩色显示完整手册 | 同上，但更详细 | `wave manual --zh` 显示中文手册 |
| `wave -V` | 显示 Summerwave 版本 | 想知道自己是哪一版 | |
| `wave install <包名>` | 安装软件包 | 装软件 | `wave install <包名>@<版本>` 安装指定版本 |
| `wave upgrade -X` | 升级软件包 | 日常升级 | 面向普通用户。`-X, --remove-old-versions` 会删掉旧版本，用之前先确认不再需要 |
| `wave latest <包名>` | 查有没有比当前更新的版本 | 想知道某个包能不能升 | `wave latest -a` 显示所有软件包的状态 |
| `wave selfupdate` | 更新 Summerwave 自身 | 想用更新的 Summerwave | |
| `wave list` | 列出已安装的软件包及版本 | 看自己装了什么 | `wave list -a` 效果相同 |
| `wave info <包名>` | 显示某个软件包装的是哪一版（没装时输出 `None`） | 不确定装没装、装的哪一版 | |
| `wave search <包名>` | 搜索软件包（匹配名字的一部分） | 想知道 `wave` 里有没有这个包 | |
| `wave uninstall <包名>` | 卸载软件包 | 卸载软件 | `wave uninstall <包名>@<版本>` 卸载指定版本 |

更多用法见 `wave -h`、`wave -h --zh`、`wave manual`、`wave manual --zh`。

注意：实际执行时，把 `<尖括号>` 里的部分换成真实值。

### 安装位置

**每个平台只有一个安装位置，且不可配置：**

| 平台 | 位置 |
| --- | --- |
| macOS | `/opt/macwave` |
| Linux | `/home/linuxwave/.linuxwave` |

**没有** `--prefix`、没有自定义目录、也不会回落到 `~/.local`。这是产品决定，不是遗漏：

- **一台机器只有一棵树。** 同一个包管理器装两份，最容易出现「`wave` 读的是这个索引、装进的是另一棵树」。
- **依赖共享。** 只有一棵树，依赖就只存在一份，不论多少用户装包（`Shore/` 常见几百 MB）。
- **路径不依赖 `$HOME`。** 改用户名、迁移机器、用 `sudo` 跑脚本，都不会让 `wave` 指向别处。
- **出问题时位置是确定的。** 排查不用先问「你装在哪」。

`wave basedir` 显示当前安装在哪。

#### 权限模型

**macOS** —— 整棵树是 `root:<管理员组>` 775，目录带 setgid。管理员组成员**装包不用 sudo**；其他本地用户只读，谁也不能替换 `bin/` 里的链接。安装时需要输**一次**管理员密码。

**Linux** —— 树属于专用的 `linuxwave` 账户及其同名组，组成员**装包不用 sudo**，服务也可以以这个低权限账户运行而不是 root。加人：`sudo usermod -aG linuxwave <用户>`，然后让他**重新登录**（组权限只对新会话生效）或执行 `newgrp linuxwave`。

#### 没有配置文件

没有 `config.toml`、没有 `~/.config/…`、也没有 `/opt/macwave_config`。以前放在那里的东西，要么已经**写死**（就是上面的安装根），要么**只能用环境变量**（下面的镜像地址）。这消掉了一整类「在我这儿是好的」的问题：磁盘上没有任何东西能与产品的默认值打架。

### 镜像

如果你在中国大陆，多半需要镜像：`github.com` 从大陆经常连不上（2026-11 实测，三次尝试里两次连 TCP 握手都没在 20 秒内完成）。要用完整镜像，把下面这些环境变量加进 `~/.zshrc`（bash 用户用 `~/.bashrc`）。好的镜像站一般会直接告诉你该跑哪些命令：

| 变量 | 指向什么 | 默认值 |
| --- | --- | --- |
| `SUMMERWAVE_API_DOMAIN` | 软件包与依赖的索引 | raw.githubusercontent 上的 `SeaLane`（`…/SeaLane/HEAD`） |
| `SUMMERWAVE_BOTTLE_DOMAIN` | 预编译软件包 | GitHub Releases |
| `SUMMERWAVE_DEPS_DOMAIN` | 依赖库 | GitHub Releases |
| `SUMMERWAVE_CORE_DOMAIN` | Summerwave 自身的文件 | raw.githubusercontent 上的 `Summerwave` |
| `SUMMERWAVE_PYTHON_DOMAIN` | 自带 CPython | GitHub Release 资产（留空即默认） |
| `SUMMERWAVE_RELEASE_DOMAIN` | 该装哪一版 | `api.github.com/repos/SummerwaveOrg/Summerwave` |
| `SUMMERWAVE_PIP_INDEX` | `pip` 索引（只影响三个 Python 依赖） | `https://pypi.org/simple` |

每个值都是**域名根**，客户端往后接固定路径 —— 所以镜像只要铺同样的树：

| 取什么 | 往后接的路径 |
| --- | --- |
| 索引记录 | `/⟨包名⟩/_⟨包名⟩@⟨版本⟩.⟨os⟩-⟨arch⟩` |
| 预编译包 | `/⟨包名⟩/⟨包名⟩@⟨版本⟩.⟨os⟩-⟨arch⟩.tar.bz2` |
| Summerwave 自身文件 | `/⟨ref⟩/⟨相对路径⟩` |

**`HEAD` 这一段不能省**：索引在 `…/SeaLane/HEAD/…`（HEAD 是默认分支）。漏掉它，`wave search` / `wave latest` 读清单时只会拿到 404。

执行下面这些就可以加上（把主机换成你的镜像）：

```zsh
export SUMMERWAVE_API_DOMAIN=https://mirror.example.cn/Summerwave/sealane/HEAD
export SUMMERWAVE_BOTTLE_DOMAIN=https://mirror.example.cn/Summerwave/bottle
export SUMMERWAVE_DEPS_DOMAIN=https://mirror.example.cn/Summerwave/deps
export SUMMERWAVE_CORE_DOMAIN=https://mirror.example.cn/Summerwave/core
export SUMMERWAVE_PYTHON_DOMAIN=https://mirror.example.cn/Summerwave/python
export SUMMERWAVE_RELEASE_DOMAIN=https://mirror.example.cn/Summerwave/release
export SUMMERWAVE_PIP_INDEX=https://pypi.mirror.example.cn/simple
```

`export` 只对当前终端有效，关掉就没了。把这段追加到 `~/.zshrc`，之后每个终端都会继承：

```zsh
cat >> ~/.zshrc <<'MWMIRROR'

# Summerwave 镜像
export SUMMERWAVE_API_DOMAIN=https://mirror.example.cn/Summerwave/sealane/HEAD
export SUMMERWAVE_BOTTLE_DOMAIN=https://mirror.example.cn/Summerwave/bottle
export SUMMERWAVE_DEPS_DOMAIN=https://mirror.example.cn/Summerwave/deps
export SUMMERWAVE_CORE_DOMAIN=https://mirror.example.cn/Summerwave/core
export SUMMERWAVE_PYTHON_DOMAIN=https://mirror.example.cn/Summerwave/python
export SUMMERWAVE_RELEASE_DOMAIN=https://mirror.example.cn/Summerwave/release
export SUMMERWAVE_PIP_INDEX=https://pypi.mirror.example.cn/simple
MWMIRROR
source ~/.zshrc
```

`SUMMERWAVE_PIP_INDEX` 只影响 `pip` 装的三个 Python 依赖（`requests`、`toml`、`colorama`），所以它可以指向任何 PyPI 镜像（国内常用中科大与清华）。

这段只需要跑一次。新开的终端会自己读到；结尾那句 `source` 只是让**当前**这个终端立刻生效。不需要的行删掉也行 —— 少几行照样工作，没设的变量会回落到官方地址。

将来不想用镜像了，把那些行删掉或注释掉即可。

Linux 上用同一套变量（路径里的 `MACWAVE_*` 没变，认的是上面的变量名）。shell 是 bash 就写进 `~/.bashrc`。

### 联系我们

出问题时，在 GitHub 上开 issue，或写信到 `macwaveorg@outlook.com`。

**Have a good surf, waver!**
