### 如何下载Android源代码

#### 安装 Repo

1. 确保主目录下有一个 bin/ 目录，并且该目录包含在路径中：

   ```shell
   mkdir ~/bin
   PATH=~/bin:$PATH
   ```

2. 下载 Repo 工具，并确保它可执行：

   ```shell
   curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
   chmod a+x ~/bin/repo
   ```

   #### 初始化 Repo 客户端

   安装 Repo 后，设置您的客户端以访问 Android 源代码代码库：

   1. 创建一个空目录来存放您的工作文件。如果您使用的是 MacOS，必须在区分大小写的文件系统中创建该目录。为其指定一个您喜欢的任意名称：

      ```shell
      mkdir WORKING_DIRECTORY
      cd WORKING_DIRECTORY
      ```

   2. 使用您的真实姓名和电子邮件地址配置 Git。要使用 Gerrit 代码审核工具，您需要一个与[已注册的 Google 帐号](https://www.google.com/accounts)关联的电子邮件地址。确保这是您可以接收邮件的有效地址。您在此处提供的姓名将显示在您提交的代码的提供方信息中。

      ```shell
      git config --global user.name "Your Name"
      git config --global user.email "you@example.com"
      ```

   3. 运行 `repo init` 以获取最新版本的 Repo 及其最近的所有错误更正内容。您必须为清单指定一个网址，该网址用于指定 Android 源代码中包含的各个代码库将位于工作目录中的什么位置。

      ```shell
      repo init -u https://android.googlesource.com/platform/manifest
      ```

      要对“master”以外的分支进行校验，请使用 `-b` 来指定相应分支。要查看分支列表，请参阅[源代码标记和版本](https://source.android.google.cn/source/build-numbers.html#source-code-tags-and-builds)。

      ```shell
      repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
      ```

   初始化成功后，系统将显示一条消息，告诉您 Repo 已在工作目录中完成初始化。客户端目录中现在应包含一个 `.repo` 目录，清单等文件将保存在该目录下。

   注：在国内google大网上不了，要使用其它下载源，如清华：

   ```shell
   repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.0.0_r17 //下载指定版本
   ```

   

#### 下载 Android 源代码树

要将 Android 源代码树从默认清单中指定的代码库下载到工作目录，请运行以下命令：

```shell
sudo repo sync
```

Android 源代码文件将位于工作目录中对应的项目名称下。初始同步操作将需要 1 个小时或更长时间才能完成。要详细了解 `repo sync` 和其他 Repo 命令，请参阅[开发](https://source.android.google.cn/source/developing.html)部分。

#### 遇到的问题点：

1. 如果执行该命令的过程中,提示无法连接到 gerrit.googlesource.com，那么我们只需要编辑 ~/bin/repo文件，找到REPO_URL这一行,然后将其内容修改为:

   REPO_URL = 'https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
   
   
   
2. python 没安装

   ```shell
   yufenzhi@yufenzhi:/media/yufenzhi/黑豆/studio/android_source$ repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-8.0.0_r17
   /usr/bin/env: “python”: 没有那个文件或目录
   
   ```

   安装一个Python:

   ```shell
   sudo apt-get install python
   ```

3. Checking out projects:  55% (313/568) platform/frameworks/baseerror: unable to create file tests/P_str_escape/str\\escape.rs: 无效的参数

   ```shell
   Checking out projects:  55% (313/568) platform/frameworks/baseerror: unable to create file tests/P_str_escape/str\\escape.rs: 无效的参数
   error: Cannot checkout platform/frameworks/compile/slang: GitError: Cannot initialize work tree for platform/frameworks/compile/slang
   Traceback (most recent call last):
     File "/media/yufenzhi/BB/studio/android_source/.repo/repo/main.py", line 530, in <module>
       _Main(sys.argv[1:])
     File "/media/yufenzhi/BB/studio/android_source/.repo/repo/main.py", line 505, in _Main
       result = run()
     File "/media/yufenzhi/BB/studio/android_source/.repo/repo/main.py", line 498, in <lambda>
       run = lambda: repo._Run(name, gopts, argv) or 0
     File "/media/yufenzhi/BB/studio/android_source/.repo/repo/main.py", line 201, in _Run
       result = cmd.Execute(copts, cargs)
     File "/media/yufenzhi/BB/studio/android_source/.repo/repo/subcmds/sync.py", line 994, in Execute
       self._Checkout(all_projects, opt)
   
   ```

   找到 `rameworks\compile\slang\tests\P_str_escape`目录并打开， 创建str目录并在str目录下创建空文件escape.rs:再重新运行 repo sync
   
4. autotest目录无法checkout

   ```shell
   ..
   Fetching project platform/external/jemalloc
   Fetching projects: 100% (545/545), done.
   error: unable to create file frontend/client/src/autotest/public/Open+Sans:300.woff (Invalid argument)
   error: unable to create file frontend/client/src/autotest/public/Roboto+Bold:700.woff (Invalid argument)
   error: unable to create file frontend/client/src/autotest/public/Roboto+Light:300.woff (Invalid argument)
   error: unable to create file frontend/client/src/autotest/public/Roboto+Medium:500.woff (Invalid argument)
   error: unable to create file frontend/client/src/autotest/public/Roboto+Regular:400.woff (Invalid argument)
   error: unable to create file server/site_tests/display_EdidStress/test_data/edids/weekly/SCT_272_STEELCASE_m:s_HDMI.txt (Invalid argument)
   Checking out files: 100% (9442/9442), done.
   Traceback (most recent call last):
     File "/mnt/d/aosp/.repo/repo/main.py", line 531, in <module>
       _Main(sys.argv[1:])
   ```

   目前没找到根本的解决方案，但可以使用下面的办法跳过这些错误从而能checkout其它文件：

   打开 ./repo/repo/project.py 文件，注释掉下面的语句：

   ```
   # if GitCommand(self, cmd).Wait() != 0:
   # raise GitError("cannot initialize work tree")
   ```

   最后失败的项目如下，但不妨碍我们看Android源码：

   ```shell
   error: Exited sync due to checkout errors
   Failing repos:
   external/autotest
   external/libunwind
   prebuilts/tools
   ```

   

更多问题可参考：https://source.android.google.cn/source/downloading