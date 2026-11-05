## 阶段 0 踩坑笔记

### 排查问题的思路
先看现象 → 分层定位（命令写对了吗 → 工具收到参数了吗 → 系统层面生效了吗）→ 找到根因再改，不盲目重试。

### Java / IDEA
- **JDK 要选 LTS 版本**：IDEA 默认推荐最新版（比如 27，不是 LTS）。学习和企业项目都用 LTS（17 / 21 / 25），供应商选 Eclipse Temurin 或 Amazon Corretto。
- **IDEA 下载的 JDK 不在 PATH 里**：命令行里 `java -version` 会提示找不到。需要手动配置 `JAVA_HOME`，并把 `%JAVA_HOME%\bin` 加进 Path。

### Git
- **远程地址不要手打**：到仓库页面点 Code → SSH 复制。占位符没替换掉会报 `is not a valid repository name`。
- **提示 origin 已存在**：修改地址用 `git remote set-url origin <地址>`，而不是再 `add` 一次。
- **在 GitHub 新建仓库时不要勾选初始化 README**：否则远程会多出一个本地没有的 commit，推送会被拒绝。
- **.gitignore 只对未跟踪的文件生效**：已经提交过的文件，再写进 .gitignore 也没用。要先 `git rm -r --cached .`，再 `git add .`。
- **.gitignore 要放在仓库根目录**：`.idea/.gitignore` 是 IDEA 自己生成的，管不到 `.idea` 目录本身。可以用 `git check-ignore -v <路径>` 检查规则是否生效。
- **解决冲突要在合并 commit 里一次完成**：改文件 → `git add` → `git commit`。
- **不小心进了 vim**：按 `Esc`，输入 `:wq`，回车。
- **撤销 commit**：还没 push 的，用 `git commit --amend` 或 `git reset --soft/--mixed/--hard HEAD~1`；已经 push 的，用 `git revert`。操作失误的后悔药是 `git reflog`。

### Docker
- **镜像拉不下来**：Docker Hub 的服务器在国外，需要配置镜像加速或者代理。
- **在 Git Bash 里运行交互命令报 `not a TTY`**：在命令前加 `winpty`。
- **Git Bash 会改写 Linux 路径**：用 `-v` 挂载时，在命令前加 `MSYS_NO_PATHCONV=1`。
- **`-p 宿主机端口:容器端口`**：端口冲突发生在宿主机这一侧，所以改左边。端口映射只能在创建容器时指定，漏掉了只能删掉重建。
- **`docker ps` 的 PORTS 列只显示 `3306/tcp`**：说明没有映射成功，从 Windows 连不上。`docker exec` 是直接进入容器内部，所以不受影响，也就容易发现不了这个问题。
- **端口被 Windows 系统保留**：用 `netsh interface ipv4 show excludedportrange protocol=tcp` 查看保留范围。解决方法：用管理员身份打开 PowerShell，依次执行 `net stop winnat` → `netsh int ipv4 add excludedportrange protocol=tcp startport=3306 numberofports=1` → `net start winnat`。
- **数据持久化**：创建容器时加 `-v 卷名:容器内数据目录`，这样删掉重建容器后数据也不会丢。
- **MySQL 里中文存不进去或乱码**：连接时加上 `-e LANG=C.UTF-8` 和 `--default-character-set=utf8mb4`。

最终可用的 MySQL 启动命令：
```bash
MSYS_NO_PATHCONV=1 docker run -d --name mysql8 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v mysql-data:/var/lib/mysql mysql:8.0
```

### HTTP
- 4xx 是客户端的请求有问题，5xx 是服务器处理时出错。
- GET 的参数放在 URL 的查询串里，POST 的数据放在请求体里。
- token 放在请求头 `Authorization: Bearer xxx` 里，用来证明"我是谁"。
