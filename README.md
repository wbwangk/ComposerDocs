### 安装mkdocs
本地环境是vagrant管理下的ubuntu16，已经预装了python2.7，但需要自己装pip。  
pip的安装参考[这个文档](http://pip.readthedocs.io/en/latest/installing/)。  
用pip安装mkdocs并安装必要依赖：
```
pip install mkdocs  
pip install python-markdown-math
```
### 构建和本地调试
下载源文档，并构建：
```
git clone https://github.com/wbwangk/ComposerDocs.git
cd ComposerDocs
$ mkdocs build
```
下列命令会运行一个web服务将site目录下的内容代理到8000端口：
```
$ mkdocs serve  --dev-addr 192.168.16.103:8000
```
由于是在虚拟机中运行，而且不是NAT模式，所以需要加上`--dev-addr`参数确保绑定的IP不是默认的127.0.0.1。  
在windows(宿主机)下打开浏览器访问地址`192.168.16.103:8000`就可以看到文档内容了。注意自己对index.md的修改内容是否生效。  

### 提交到github
mkdocs支持直接发布到github pages。
```
$ mkdocs build
$ mkdocs gh-deploy
INFO    -  Cleaning site directory
INFO    -  Building documentation to directory: /opt/ComposerDocs/site
INFO    -  Copying '/opt/ComposerDocs/site' to 'gh-pages' branch and pushing to GitHub.
Username for 'https://github.com': wbwangk
Password for 'https://wbwangk@github.com':
INFO    -  Your documentation should shortly be available at: https://wbwangk.github.io/ComposerDocs/
```
也就是直接把site目录发布到了同一个库的gh-pages分支，很方便。
### 查看结果
首先，到`https://github.com/wbwangk/hyperledgerDocs/tree/gh-pages`查看提交的结果。  
然后到 https://wbwangk.github.io/hyperledgerDocs/ 去查看github pages解析后的结果。

### 脚本化
可以建一个脚本来减少反复输入命令(如1.sh)，不过对bash脚本不熟悉，不会编写自动输入github账号和口令。
```
rm -rf ComposerDocs
git clone https://github.com/wbwangk/ComposerDocs
cd ComposerDocs
mkdocs build
mkdocs gh-deploy
```
`mkdocs gh-deploy`会提示输入github账号和口令。（fork到自己github账号下再运行上述脚本）
