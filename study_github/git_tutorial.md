## github使用指南[引用地址1](https://www.zhihu.com/question/27712995)[引用地址2](https://git-scm.com/book/zh/v2)[引用地址3](https://learngitbranching.js.org/?locale=zh_CN)[引用地址4](https://docs.github.com/en/get-started/quickstart/set-up-git)
>首先得说github大致上是用于在线开发的这么一个工具，他可以支持多人同时编辑一个项目。有许多强大的功能，其中一个主要的功能是repository库，他可以把创建的项目放入这个库里。并可以实现对代码的分支编辑（对一个程序代码的新增功能单独建立分支）然后把分支的程序代码合并到主程序里。还有时间线的的返回，可以回到以前编辑的代码时间点的功能等等。
1. 先在网络上申请一个帐号，输入你的邮箱和用户名（用户名不能重复），顺利生成后就可以登录github了。
2. 若是要在线下进行编辑，并能同步到github并且有时间线和分支就要安装git工具，可以把他看成是一个命令行工具，里面有专门针对代码（程序）的提交，查看日志，创建分支和恢复时间线的功能。git分为github cli与github desktop，前者是命令行，后者是页面操作。
3. 建议安装github cli，能更清楚的知道代码变动的具体信息。
3. 若要通过将本地内容上传到github，需要创建sshkey，用于客户端访问授权。
4. 当你使用一个新的文件夹作为repository时（private是付费的，public是免费的），要用git进行初始化。
5. 当创建了一个代码时，需要用git将其加入缓存中，并提交到主时间线里。
6. 根据复杂程度可以对代码创建分支，进行单独功能的编辑，而不影响主分支。
7. push代码到github上。
8. 根据需要退到历史中的节点（每一次提交的位置），用校验码的前六位
9. 将完善后的代码合并到主分支main。

## 设置git的环境
```
	//配置用户名,添加到~/.gitconfig文件里
	git config --global user.name "firstname lastname"
	//配置邮箱
	git config --global user.email "name@company.name"
	//配置颜色显示，便于查看错误
	git config --global color.ui auto
```
## 生成ssh key
```
	//生成一个sshkey
	ssh-keygen -t ed25519 -C "your_email@example.com"
	//然后会让你选择一个存储路径，如果不选就直接回车
	//还会让你设置一个密码，并进行密码确认
```
1. 生成完sshkey需要把public key的信息填入到github的sshkey里，就添加成功了。
2. 然后可以在本地测试一下与github的联通。
```
	ssh -T git@github.com
```
3. 会要求你输入密码，输入之前设置的密码就可以看到成功的提示。

## 克隆github上的库到本地
```
	//username换成用户名，repositoryname换成库名，这条信息可以在code里查询到，并直接复制。
	git clone git@github.com:username/repositoryname.git

	//复制好后，就可以进入这个路径查看里面的文件
	cd repositoryname
```
1. 然后就可以向这个库里添加代码了。
2. 在git（bash里）可以用vi来编辑代码，要直接创建在这个库文件夹下。
3. 比如创建一个hello.py文件。
```
	//在库文件夹下，比如 python-learning 是库名
	vi hello.py
	//在文件里添加如下代码
	print ("hello world!")
```

## 添加代码文件到git库
>创建好文件后就可以把文件放入到git库了
```
	//查看状态使用以下命令
	git status
	//先要将文件加入暂存区
	git add hello.py
	//将文件提交，保存到仓库的历史记录中，-m后面的字符串是提交信息
	git commit -m "这里输入对此次提交文件的修改信息，尽量简明扼要便于查看。"
	//查看提交日志
	git log

	//更新本地库到github库
	git push
```
```
	//同时增加文件到暂存区并提交
	git commit -am "Add feature-C"
```

## 初始化库
```
	//创建一个库的目录
	mkdir git-tutorial
	//进到这个目录下
	cd git-tutorial
	//初始化此目录为库,自动生成"附属该仓库的工作树"
	git init

	//然后就可以在库中添加代码文件了
	vi README.md
	//加入到暂存区
	git add README.md
	//提交到库，如果不加参数 -m 会出现个文本编辑窗口，可以输入更加详细的提交信息。
	git commit

	//显示短的log日志
	git log --pretty=short
	//只显示指定的文件或目录
	git log README.md
	//查看提交前后的日志变化
	git log -p

	//查看更改前后的区别
	git diff
	//查看与最新的提交的区别
	git diff head
```
## 使用git的分支进行作业
1. 创建与切换分支
```
	//查看git的分支列表，当前分支有"*"显示
	git branch
	//查看git的分支列表，包括远端的分支列表
	git branch -a

	//在当前分支上创建新分支，并切换到新分支。参数 -b 表示创建分支
	git checkout -b feature-A
	//同上，分开步骤
	//先创建一个分支
	git branch feature-A
	//切换到新建的分支
	git checkout feature-A

	//切换上一分支（有点像linux的切换上一路径 cd -）
	git checkout -
```
2. 合并分支
```
	//切换到主干分支，主干分支的名字因版本有所区别
	git checkout master/main
	//确认切换到主干分支后，合并feature-A分支到主干master
	git merge --no-ff feature-A

	//以图形的方式查看日志
	git log --graph
```

## 回溯历史版本
```
	//回到创建分支前的历史节点（索引的哈希值）的对应时间
	git reset --hard 哈希值
	//通过主干分支的历史节点创建另一分支
	git checkout -b fix-B
	//查看回溯历史之前的分支日志
	git reflog
```
## 将本地的库推送到远端的github库
>首先要在github上新建一个库与本地库同名，不要自动建立README文档。然后依次运行以下命令。
```
	//添加远端库，告诉本地库你远端库的地址，远端库链接的别名是origin
	git remote add origin git@github.com:name/repositoryname.git
	//强制将主分支名称改名为main，为了与远端名称匹配
	git branch -M main
	//推送到上游库的分支，-u表示此操作是向上游推送，并且将本地库与上游库做了映射，方便以后推送就不用输入 -u origin main 了，origin上游库链接的别名，main是上游库的主分支。上游也可以理解为远端。
	git push -u origin main

	//更改远程库链接别名，将origin改为useless
	git remote rename origin useless

	//查看远程库链接别名
	git remote
	//同上，信息更详细
	git remote -v
	
	//删除本地对远端库的映射
	git remote remove origin
```

## 获取远端库的内容
>假设远端库已经被clone到本地的新文件夹，那么现在就可以把远端的内容拽到本地。
```
	//本地只有main分支，从远端获取origin库的feature-D分支，对应到本地新建的feature-D分支
	git checkout -b feature-D origin/feature-D
	//如果其他人使用feature-D改变了README.md文件，需要将该分支的文件同步到本地，先切换到feature-D分支，然后再将远端的分支拖拽到本地。否则可能会造成主分支与其他分支合并的意外状况。
	git checkout feature-D
	//使用远端origin库的feature-D分支来更新本地的feature-D分支
	git pull origin feature-D
```

