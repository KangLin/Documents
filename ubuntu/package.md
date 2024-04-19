# 打包、发布

作者：康林 <kl222@126.com>

### 打包、发布流程

#### 打包

- 用 dh_make 生成 debian 模板文件。详见：[2.8. 初始化外来 Debian 软件包](https://www.debian.org/doc/manuals/maint-guide/first.zh-cn.html#non-native-dh-make)
- 修改 debian 下的内容。详见：
[第 4 章 debian 目录中的必需内容](https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html)
[第 5 章 debian 目录下的其他文件](https://www.debian.org/doc/manuals/maint-guide/dother.zh-cn.html)
- 构建软件包。详见：[第 6 章 构建软件包](https://www.debian.org/doc/manuals/maint-guide/build.zh-cn.html)

##### 参考
- [Debian 软件包制作介绍](https://www.debian.org/doc/manuals/packaging-tutorial/packaging-tutorial.en.pdf)
- [Debian 新维护者手册](https://www.debian.org/doc/manuals/maint-guide/index.zh-cn.html)
- [Debian 维护者指南](https://www.debian.org/doc/manuals/debmake-doc/index.zh-cn.html)

#### 发布

##### 发布流程

参考：[维护者简介：我的软件包将如何进入 Debian?](https://mentors.debian.net/intro-maintainers/)

1. 打包 \
1.1. 自己打包。详见：[打包](#打包) \
1.2. 找个帮助你打包。 \
向 [WNPP](https://www.debian.org/devel/wnpp/) 发一个 RFP（Request for package) BUG
2. 上传到 mentors.debian.net \
2.1. 上传deb包前先要创建自己gpg公钥(需要4096bit)，其他默认即可。

       $ gpg --full-gen-key

2.2. 同时在 https://mentors.debian.net/ 创建一个用户，设置中填写公钥。

       $ gpg --export --export-options export-minimal --armor [密钥id 的输出] > public_key.txt

参见: [GPG](../base/gpg.md)

2.3. 用 debsign 工具对打包后的 .dsc 、 .changes 文件签名

       $ sudo apt install devscripts
       $ debsign example.changes

2.4. 开始上传软件 \
2.4.1. 配置 dput，配置文件（~/.dput.cf) \
2.4.1.1. Https 方式

       [mentors]
       fqdn = mentors.debian.net
       incoming = /upload
       method = https
       allow_unsigned_uploads = 0
       progress_indicator = 2
       # Allow uploads for UNRELEASED packages
       allowed_distributions = .*
       
2.4.1.2. ftp 方式

        [mentors-ftp]
        fqdn = mentors.debian.net
        login = anonymous
        progress_indicator = 2
        passive_ftp = 1
        incoming = /pub/UploadQueue/
        method = ftp
        allow_unsigned_uploads = 0
        # Allow uploads for UNRELEASED packages
        allowed_distributions = .*

2.4.2 上传

    $ dput mentors your_package.changes 

3. 上传成功后，你会收到接受邮件。这时你可以在 [你账户下->我的包](https://mentors.debian.net/packages/my/) 中可以看到刚上传的包。
它也会出现在 https://mentors.debian.net/packages/ 中
4. 寻求赞助者 (sponsor) \
4.1. 打开 https://mentors.debian.net/packages/my/ 中的包，把 Needs a sponsor: 	No 改成 Yes \
4.2. 修改模板 View RFS template ，发送邮件到 submit@bugs.debian.org
