# 一台电脑push多帐号到github操作步骤

<details><summary><h2>如果你有两个github帐号，并且都要在同一台电脑上用sshkey密钥授权来访问不同的库</h2></summary>

>流程大概是这样，首先需要创建两个sshkey密钥对，分别对应两个帐号，当然多个帐号也是同理操作。
1. 首先启动git bash，并运行ssh-agent来启动sshkey代理，启动代理要输入以下命令。
```
	//启动ssh-agent
	eval "$(ssh-agent -s)"

	//关闭ssh-agent，杀掉代理进程
	ssh-agent -k
```
2. 查看现有的公钥和私钥位置，也可以都删掉然后重新建立。默认把密钥对建立在目录~/.ssh/下。[ssh-keygen参考](https://www.ssh.com/academy/ssh/keygen#what-is-ssh-keygen?)
```
	//生成新的公私钥对
	ssh-keygen -t ed22519 -C "email@example.com"
	//在让你选择存储公私sshkey对时，可以选择输入目录和文件名，也可以不要目录只输入文件名，也可以使用默认的目录和文件名。这里建议修改文件名与项目或用户有关，如下使用项目类型做密钥名。
	python_key
	//再生成另一个帐号的密钥对，多个的也是如此添加
	ssh-keygen -t ed22519 -C "user@company.com"

	//-t参数是表示采取哪种加密算法，包括 rsa,dsa,ecdsa,ed25519等算法。-b参数是控制key尺寸的。-f 是指定一个存储路径。-C 是注释(github建议用email注释， 但为什么注释一定要是email呢？)
	ssh-keygen -t ed22519 -b 4096 -f /temp/username/keyfilename -C "user@company.com"
```
3. 在ssh key代理中增加新的私钥key，注意这步完成后就可以使用密钥来同步github了。
```
	//这里要将私钥添加入代理，此步很重要，这里添加的是哪个帐号的私钥，哪个帐号就能同步远端，如果这里添加的是多个帐号，只有第一个才生效，其他的会私钥不起作用。
	ssh-add ~/.ssh/python_key
	//如果钥登录的帐号是A，那么代理的第一个私钥必须是A，否则会报错。如果B的帐号要同步，就要把B帐号之前的私钥在代理中删掉，操作如下。
	ssh-add -d ~/.ssh/python_key

	//显示ssh-add中的密钥
	//显示私钥
	ssh-add -l
	//显示公钥
	ssh-add -L
	//从代理中删除密钥
	ssh-add -d ~/.ssh/private_key
	//删除所有密钥
	ssh-add -D
```
4. 还可以为代理设置自动添加私钥的操作。[ssh-agent参考](https://www.thisfaner.com/p/ssh-agent/)
>因为每次关闭bash后可能会导致ssh-agent关闭，那样代理内的私钥就丢失了。为了让代理在bash开启时自动运行并加载私钥，需要创建一个配置文件~/.bashrc，文件的内容参照这里添加。
```
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2= agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add ~/.ssh/mine_key1		# 添加你自己的私钥
    ssh-add ~/.ssh/mine_key2		# 添加你自己的私钥
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add ~/.ssh/one_rsa
fi

unset env
```
>创建好~/.bashrc文件后重启bash就可以生效了。但是有个问题，如果你在代理中添加了多个私钥，在与github库同步时，如果选择的帐号对应的密钥不是列在前面，就会报错。解决办法就是把代理中此密钥前面的密钥删掉，然后就可以了。

</details>
<details><summary><h2>看到网上还解决多个帐号问题的其他办法</h2></summary>
>如需要创建~/.ssh/config这个配置文件，默认是没有的，文件里面的内容如下。多个帐号需要配置多个参数配置，只是对应的值不一样(没有研究明白，记在这里作为参考)。

```
host webpage
	hostname github.com
	user jacknewnew
	IdentityFile ~/.ssh/python_key1

host webpage1
	hostname github.com
	user peternewnew
	IdentityFile ~/.ssh/python_key2

```
### 上面ssh/目录下的配置文件config参数详解

[参考地址](https://www.jianshu.com/p/1e793e386beb) [参数地址](https://zhuanlan.zhihu.com/p/35922004)
>此文档所有信息均来自互联网网友的博客记录，只是做了一下整理。感谢耶稣带领：）

|选项参数|说明|
|---|---|
|Host \* |选项“Host”只对能够匹配后面字串的计算机有效。"\*"表示所有的计算机。这个选项的值是个昵称，根据需要写就行。|
|HostName github.com|此选项的值是个地址，也可以用ip地址|
|ForwardAgent no| 设置连接是否经过验证代理（如果存在）转发给远程计算机。|
|ForwardX11 no| 设置X11连接是否被自动重定向到安全的通道和显示集（DISPLAY set）|
|RhostsAuthentication no| 设置是否使用基于rhosts的安全验证|
|RhostsRSAAuthentication no| 设置是否使用用RSA算法的基于rhosts的安全验证|
|RSAAuthentication yes| 设置是否使用RSA算法进行安全验证|
|PasswordAuthentication yes| 设置是否使用口令验证|
|FallBackToRsh no| 设置如果用ssh连接出现错误是否自动使用rsh|
|UseRsh no| 设置是否在这台计算机上使用“rlogin/rsh”|
|BatchMode no| 如果设为“yes”，passphrase/password（交互式输入口令）的提示将被禁止。当不能交互式输入口令的时候，这个选项对脚本文件和批处理任务十分有用|
|CheckHostIP yes| 设置ssh是否查看连接到服务器的主机的IP地址以防止DNS欺骗。建议设置为“yes”|
|StrictHostKeyChecking no| 如果设置成“yes”，ssh就不会自动把计算机的密匙加入“$HOME/.ssh/known_\hosts”文件，并且一旦计算机的密匙发生了变化，就拒绝连接|
|IdentityFile ~/.ssh/identity| 设置从哪个文件读取用户的RSA安全验证标识|
|Port 22| 设置连接到远程主机的端口|
|Cipher blowfish| 设置加密用的密码|
|EscapeChar ~| 设置escape字符|

</details>
<details><summary><h2>配置全局变量(附带信息)</h2></summary>

>配置文件是~/.gitconfig
```
	//列出所有全局变量
	git config --list

	//创建全局变量
	git config --global user.name "name"
	//删除全局变量
	git config --global --unset user.name
	//编辑全局变量
	git config --global --edit
```
</details>
