
基于 GitHub Actions 实现 vue 项目的自动构建部署

## GitHub Actions 术语
（1）workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。

（2）job （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。

（3）step（步骤）：每个 job 由多个 step 构成，一步步完成。

（4）action （动作）：每个 step 可以依次执行一个或多个命令（action）。

## 一、创建workflow 文件
在项目里创建一个workflow文件，文件格式为yaml类型。文件名可以随意起，文件后缀可以为yml 或 .yaml, 这里我们创建文件 .github/workflows/deploy.yaml，注意这里的路径。

如果你的仓库中有项目文件的话，当你点击“Actions"时，系统会自动根据你的开发语言推荐一个常用的actions，在页面的右侧也会推荐一些相应的actions.
## 二、在部署服务器上生成部署用户密钥
```ssh
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_rsa_actions
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa_actions.
Your public key has been saved in /root/.ssh/id_rsa_actions.pub.
The key fingerprint is:
SHA256:SlI1ZYhwSBUemB8F5vxnWrrI2QNbPPv9ZIS3FCx0l0k root@node2
The key's randomart image is:
+---[RSA 2048]----+
|   .o*B=+oo . oE+|
|    +*oo.o . o + |
|     .=.    . o  |
|     ...     o . |
|    . ..S + . +  |
|     o..+*   + . |
|      .+oo    +  |
|     ..+o. . o   |
|      + oo. ...  |
+----[SHA256]-----+
# 将公钥添加到 authorized_keys
cat id_rsa_actions.pub >> authorized_keys
# 输出私钥
cat id_rsa_actions
```
## 三、准备工作
对于服务的部署所以这里选择了 ssh-deploy 这一个actioins，官方网址 https://github.com/marketplace/actions/ssh-deploy。

下面开始添加deploy.yaml文件中用到了一些变量。

在项目首页右上角点击 Seetings->Secets, 找到 Add a new secret，分别添加以下变量

```bash
SERVER_SSH_KEY 登录私钥，就是上面保存的私钥内容
REMOTE_HOST 服务器地址，如202.102.224.68
REMOTE_PORT (可选项)服务器ssh端口，一般默认为22
REMOTE_USER 登录服务器用户名，这时指密钥所属的用户
SOURCE (可选项)，默认为‘’, 构建服务器路径，这里为相应 $GITHUB_WORKSPACE 根目录而言的相对路径, 例如 dist/
REMOTE_TARGET 服务器部署路径，如 /data/ghactions
ARGS (可选项)默认值为 -rltgoDzvO
```
## 四、deploy.yaml文件示例
```
name: Node CI

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3
    - name: Install Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '16.x'
    - name: Install npm dependencies
      run: npm install
    - name: Run build task
      run: npm run build --if-present
    - name: Deploy to Server
      uses: easingthemes/ssh-deploy@main
      with:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          ARGS: "-rlgoDzvc -i --delete"
          SOURCE: "dist/"
          REMOTE_HOST: ${{ secrets.REMOTE_HOST }}
          REMOTE_USER: ${{ secrets.REMOTE_USER }}
          TARGET: ${{ secrets.REMOTE_TARGET }}
          EXCLUDE: "/dist/, /node_modules/"
```