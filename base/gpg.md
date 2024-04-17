## GPG

作者：康林 <kl222@126.com>

### 概念

GPG 可以分成两部分来看：

- GPG 是一个加密、解密、签名、验证工具。

- GPG 还是一个密钥管理工具，可以用于管理自己的私钥，其他人的公钥，
以及提供了一套公钥信任体系。

另外在使用 GPG 时，可能还会看到 PGP、OpenPGP 等名词，
他们是什么关系？PGP (Pretty Good Privacy) 是最早的于 1991 年发布的此类工具。PGP 很好用，
但他是商业软件。于是 GNU 计划在 1999 年发布了开源版本的 GNU Privacy Guard (GnuPG 或 GPG)。
而 OpenPGP 是于 1997 年制定的一套标准，GPG、PGP 等工具都实现了这套标准。

参考： https://linux.cn/article-9524-1.html

#### 密钥
##### 功能类型
- PGP 密钥有四种功能类型
  - [S] 密钥可以用于签名(Signing)
  - [E] 密钥可以用于加密(Encryption)
  - [A] 密钥可以用于身份认证(Authentication)
  - [C] 密钥可以用于认证其他密钥(Certification)

- “主密钥”和“子密钥”
  - 在“主密钥”和“子密钥”之间没有技术上的区别。
  - 在创建时，指定每个密钥特定的功能类型来限制分配的功能。
  - 一个密钥可能具有多种功能类型。

带有 **[C] （认证）** 功能的密钥被认为是“主”密钥，因为它是唯一可以用来表明与其他密钥关系的密钥。
只有 [C] 密钥可以被用于：

  - 添加或撤销其他密钥（子密钥）的 S/E/A 能力
  - 添加、更改或撤销密钥关联的身份（uid）
  - 添加或更改本身或其他子密钥的到期时间
  - 为了网络信任目的为其它密钥签名

在自由软件的世界里，[C] 密钥就是你的数字身份。一旦你创建该密钥，
你应该格外小心地保护它并且防止它落入坏人的手中。参见：[删除主密钥](#删除主密钥)

- 参考： https://linux.cn/article-9529-1.html

##### 选择密钥

\<key-id\> 可以用以下几种不同形式来选择：

- 指纹(fingerprint): 一个完整的 40 个字符的密钥标识符
- 密钥 ID（Key ID）
  - 长密钥 ID(Long key ID): 指纹的最后 16 个字符（AAAABBBBCCCCDDDD）
  - 短密钥 ID(Short key ID): 指纹的最后 8 个字符（CCCCDDDD）
- 用户 ID(User ID[uid]):
每个主密钥可以有一个或多个 uid，用于标识用户身份。格式为 `Real Name (Comment) <Email>`。
  - 精确匹配的邮箱：\<xxx@xxx\>
  - 部分匹配的邮箱：@xxx
  - 部分匹配 uid：someone，不区分大小写

所有需要选择某个 key 的地方，都可以使用上面这些形式。注意除了指纹和 Key ID 外，
其他都可能匹配到多个，将会选择第一个使用。

除了这些基本的匹配外，对于 Key ID 和指纹，还有个特殊的用法：`!` 。

GPG 支持一对“主密钥”和若干对“子密钥”。而如果主密钥支持 [SC]，某个子密钥也支持 [S]。
那么签名时将会使用哪个呢？答案是使用子密钥。
在选择密钥时，GPG 会自动选择一个支持对应能力的子密钥，如果存在多个，则选择较新的一个。
那么如果就是要使用某个特定的主密钥或子密钥呢？这时就可以在 key ID 或指纹后面加上 `!` ，
强制使用这个密钥，而不要自动推算。

这种用法还可以用在导出密钥时，仅导出某个子密钥或主密钥。

#### 信任网络

事实上，几乎很难直接验证每个密钥的有效性，这时可以利用 GPG 的信任网络来间接验证密钥的有效性。

简单来说就是如果我确认了某个密钥的主人确实是美国总统，而我信任他不会乱认人（但愿吧）。
那么如果这个密钥对某个叫做国务卿的密钥进行了签名，我就也认为这个国务卿确实是有效的，
而不用亲自去认证。

信任分为几个不同的等级：

- ultimate：终极信任。一般只应该对自己的密钥进行终极信任，可以理解为所有信任链的根节点。
- full：完全信任。对于这个人签名的其他密钥，我也认可。
- marginal：信了，但没完全信。这个人签名过的密钥，我不一定认可，但如果三个这样的人都签了，
那我就信了。
- never：不信任。这个人对其他密钥的签名，我一律视而不见。

需要注意信任和有效性不一定有关系。信任表示的是对这个密钥签名其他密钥的信任程度，
而有效性是对这个密钥本身的验证。例如我知道某个密钥确实是特朗普的，他是有效的。
但我觉得这个人满嘴跑火车，他签名的其他密钥我一概不认。

有了信任网络，一个密钥有效的条件就变成了：被一个 full 签名或三个 marginal 签名，
同时这个签名传递的路径距离我（ultimate）的长度不超过 5。这些值都可以修改，具体可以参考 man gpg。
更具体的说明可以参考 The GNU Privacy Handbook。

如何修改信任值将在[编辑密钥](#编辑密钥)具体介绍。

### 使用

#### 生成密钥

- 交互式生成

      gpg --gen-key
      或者
      gpg --generate-key

将会交互式的要求输入真实姓名、邮箱的信息。输入完成后就会生成一对主密钥 [SC] 和一对子密钥 [E]。
这样将会使用主密钥进行签名，使用子密钥进行解密。但是为了安全考虑，一般会再生成一个用于签名的子密钥。
而主密钥仅用于认证子密钥(Certification)。

- 快速生成

      gpg --quick-generate-key 'Kang Lin <kl222@126.com>' rsa4096 cert
   
- 获得一个全功能的密钥生成对话框

      gpg --full-generate-key

#### 创建子密钥

- 用法：

      gpg [选项] --quick-add-key 指纹 [算法 [功能 [期限]]]

- 参数：
  - 算法：可以使用 `gpg --version` 来查看
  - 功能：`default` 或 `-`， 保持默认值。或者以 `,` 分隔的以下值的列表：
    - encr: 加密
    - sign: 签名
    - auth: 验证

- 示例：

      l@l:~$ gpg --quick-add-key 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7 rsa4096 sign

- 也可以使用[编辑密钥](#编辑密钥)中命令：`addkey`， 参见：[子密钥](#子密钥)

#### 查看密钥

- 用法：

      gpg (-k | -K) [--with-fingerprint] [--with-subkey-fingerprints] [--with-sig-list] [--with-sig-check] [<key-id>]
      gpg --fingerprint
      gpg --list-sigs

- 参数说明：

  - -k/--list-keys/--list-public-keys：查看公钥。
  - -K/--list-secret-keys：查看私钥。
  - --with-xxx：根据字面意思，将会额外打印一些信息，具体信息可以通过 man gpg 查看。
  - --fingerprint：打印密钥的指纹，等价于 gpg -k --with-fingerprint。
  - --with-subkey-fingerprint：打印子密钥的指纹
  - --list-sigs：打印密钥的签名，等价于 gpg -k --with-sig-list。
  - \<key-id\>：仅打印指定的密钥。可选，否则打印所有密钥。如何选择一个密钥请参考：[选择密钥](#选择密钥)
    
- 示例：
  - 查看公钥

        l@l:~$ gpg -k
        /home/l/.gnupg/pubring.kbx
        --------------------------
        pub   rsa4096 2021-08-02 [SC]
              08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
        uid             [ 绝对 ] Kang Lin <kl222@126.com>
        sub   rsa4096 2021-08-02 [E]
        sub   rsa4096 2024-01-11 [A]
        sub   rsa4096 2024-01-11 [S]
        sub   rsa4096 2024-01-18 [S]

  - 查看私钥

        l@l:~$ gpg -K
        /home/l/.gnupg/pubring.kbx
        --------------------------
        sec   rsa4096 2021-08-02 [SC]
              08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
        uid             [ 绝对 ] Kang Lin <kl222@126.com>
        ssb   rsa4096 2021-08-02 [E]
        ssb   rsa4096 2024-01-11 [A]
        ssb   rsa4096 2024-01-11 [S]
        ssb   rsa4096 2024-01-18 [S]

  - 查看公钥指纹

        gpg -k --with-fingerprint
        /home/l/.gnupg/pubring.kbx
        --------------------------
        pub   rsa4096 2021-08-02 [SC]
              08C9 D8FE 2093 0F2A 9BDF  565A B5AF 9300 7DC8 E9E7
        uid             [ 绝对 ] Kang Lin <kl222@126.com>
        sub   rsa4096 2021-08-02 [E]
        sub   rsa4096 2024-01-11 [A]
        sub   rsa4096 2024-01-11 [S]
        sub   rsa4096 2024-01-18 [S]
      
  - 查看公钥子键指纹

        l@l:~$ gpg -k --with-subkey-fingerprint
        /home/l/.gnupg/pubring.kbx
        --------------------------
        pub   rsa4096 2021-08-02 [SC]
              08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
        uid             [ 绝对 ] Kang Lin <kl222@126.com>
        sub   rsa4096 2021-08-02 [E]
              75E0F1AA77805AE86E51C7F100C7D98BE886845F
        sub   rsa4096 2024-01-11 [A]
              3B0A1A828963E34CB8360A3EC7DFE5CE9DBABAAC
        sub   rsa4096 2024-01-11 [S]
              1278EA6F9E3E6A7995411F0AE9729414857DE6C4
        sub   rsa4096 2024-01-18 [S]
              711C59F16B91A9E1DB5A3602B21A1E4D8BF6B2D1

- 密钥类型

每个密钥开头都会有这样几个字母：pub/sub/sec/ssb，表示这个密钥的类型。

在非对称加密算法中，密钥都是成对出现的。pub 和 sec 就分别表示这一对密钥的公钥和私钥，
而 sub 和 ssb 则分别表示一对子密钥的公钥和私钥。

#### 加密

- 用法：

        gpg -r <key-id> [-o <output-file>] [-a] -e [<input-file>]
        
- 参数：

  - -r/--recipient：指定接受者的公钥 ID，消息将会使用这个公钥进行加密，
也就是只有拥有这个私钥的人才能解密信息。可以指定多个，则多个接受者都能解密信息。
参考：[选择密钥](#选择密钥)
  - -o/--output：指定加密后的信息输出到哪个文件。可选，如果不指定将会输出到标准输出。
  - -a/--armor：将加密后的信息转为可打印的 ASCII 字符。可选，如果不指定将会输出二进制信息。
  - -e/--encrypt：加密。相应的还有解密、签名、验证等参数，将在后面介绍。
  - \<input-file\>：要加密的文件。可选，如果不指定将会从标准输入读取。

注意：只有 -r 指定的私钥才能解密信息，如果没有指定自己，则自己也无法解密。
也可以仅指定自己，用于加密隐私文件。可以通过配置，加密时默认将自己包含在内，后面将会介绍。

- 示例：

        l@l:~$ gpg -r 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7 -o myfile.gpg -e myfile
        l@l:~$ echo 'some message' | gpg -r 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7 -ae

- 注意：在选择密钥时，GPG 会自动选择一个有加密功能的子密钥，如果存在多个，则选择较新的一个。
如果要使用某个特定的主密钥或子密钥呢？这时就可以在 key ID 或指纹后面加上 ! ，
强制使用这个密钥，而不要自动推算。

#### 解密

- 用法：

        gpg [-o <output-file>] -d [<input-file>]

- 参数：
  - -o/--output：指定解密后的信息输出到哪个文件。可选，如果不指定将输出到标准输出。
  - -d/--decrypt：解密。
  - \<input-file\>：待解密的文件。可选，如果不指定将尝试从标准输入读入。
- 示例：

        l@l:~$ gpg -o myfile -d myfile.gpg
        l@l:~$ echo 'some message' | gpg -r 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7 -e | gpg -d

- 注意：解密时没有指定私钥，因为一般加密信息中会包含使用的公钥 ID，
GPG 将会自动在本地寻找对应的私钥，如果找不到将会解密失败。

#### 签名
- 用法：

        gpg [-u <key-id>] [-o <output-file>] [-a] (-s | -b | --clearsign) [<input-file>]

- 参数：
  - -u/--local-user：指定用来签名的密钥 ID，签名后，只有对应的公钥可以验证成功。参考：[选择密钥](#选择密钥)
  - -o/--output：指定签名后的信息输出到哪个文件。可选，如果不指定将输出到标准输出。
如果不指定将使用默认密钥，如果没有默认密钥将使用第一个可用密钥。默认密钥参考：配置文件。
  - -a/--armor：将加密后的信息转为可打印的 ASCII 字符。可选，如果不指定将会输出二进制信息。
  - -s/--sign：签名。
  - --clearsign：签名并保持原始信息。-s 签名后，信息将会打包成 GPG 的格式。
虽然没有加密，但仍需 GPG 命令才能解析查看。
 --clearsign 签名，会保持原始信息，额外附加一段签名信息，这样任何人都可以直接看到原始信息，需要验证的再使用 GPG 验证。
  - -b/--detach-sign：分离签名。如果是文本信息不想修改原始信息，可以使用 --clearsign 保持原始信息。
而对于二进制文件，则可以使用 -b 创建一个独立的签名文件。
- 示例：

        l@l:~$ echo 'some message' | gpg -u myself --clearsign
        l@l:~$ gpg -b file.zip

#### 验证
- 用法：

        gpg (--verify | -d) [<file>...]

- 参数：
  - --verify：仅验证签名是否正确，不输出原始信息。
  - -d/--detach-sign：验证签名是否正确，并输出原始信息。
  - \<file\>...：对于 -s/--clearsign 的签名，输入为签名文件。
对于 -b 的签名，第一个参数为签名文件，第二个参数为被签名的数据，如果不指定第二个参数，将会自动尝试去掉后缀作为原始文件名。
- 示例：

        echo 'some message' | gpg -u myself -s | gpg -d
        gpg --verify file.zip.sig

#### 导出密钥

- 用法：

      gpg [-a] [-o <output-file>] [--export-options <option>[,<option>...]] (--export | --export-secret-keys | --export-secret-subkeys) [<key-id>]
    
- 参数：
  - -a/--armor：将加密后的信息转为可打印的 ASCII 字符。可选，如果不指定将会输出二进制信息。
  - --export：导出公钥。
  - --export-secret-keys：导出私钥。
  - --export-secret-subkeys：仅导出子密钥的私钥。
  - --export-options：导出选项 `,` 逗号分割，常用的有：
    - backup：导出用于备份。会将信任值、本地签名等一起导出。
    - export-clean/export-minimize：参见：[清理签名](#清理签名)
  - \<key-id\>：如果不指定将会导出所有公钥或私钥，如果指定仅会导出指定的密钥，
使用 `!` 可以仅导出主密钥或子密钥。参考：[选择密钥](#选择密钥)

- 示例：

      # 导出公钥
      l@l:~$ gpg -a -o KangLin.gpg --export-options backup,export-clean --export 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      # 导出私钥
      l@l:~$ gpg -a -o KangLin.pri.gpg --export-options backup,export-clean --export-secret-keys 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      # 仅导出子密钥的私钥。
      l@l:~$ gpg -a -o KangLin.pri.gpg --export-options backup,export-clean --export-secret-subkeys 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7

#### 导入密钥

- 用法：

      l@l:~$ gpg --import [--import-options <option>[,<option>...]] [<key-file>]

- 参数：
  - --import：导入密钥。
  - --import-options：导出选项 `,` 逗号分割，常用的有：
    - restore：恢复，与导出选项的 backup 对应。
    - import-show：回显导入的密钥信息。
  - \<key-file\>: 密钥文件
- 示例：

      l@l:~$ gpg --import KangLin.gpg

#### 删除密钥

- 用法：

      gpg (--delete-keys | --delete-secret-keys | --delete-secret-and-public-keys) <key-id>...
     
- 参数：
  - --delete-keys：删除公钥。
  - --delete-secret-keys：删除私钥。
  - --delete-secret-and-public-keys：同时删除私钥和公钥。
  - \<key-id\>：如果不指定将会导出所有公钥或私钥，如果指定仅会导出指定的密钥，
使用 `!` 可以仅导出主密钥或子密钥。参考：[选择密钥](#选择密钥)
- 示例：

      l@l:~$ gpg --delete-keys someone
      l@l:~$ gpg --delete-secret-and-public-keys someone

#### 密钥服务器

- 用法：

      gpg [--keyserver <key-server>] [--keyserver-options <option>[,<option>...]] (--send-keys | --recv-keys | --refresh-keys | --search-keys) [<key-id>]
     
- 参数：
  - --send-keys：将密钥发送到密钥服务器。
  - --recv-keys：从密钥服务器接收密钥。
  - --refresh-keys：从密钥服务器更新本地密钥。
  - --search-keys：从密钥服务器搜索密钥。
  - --keyserver：指定一个密钥服务器，大部分密钥服务器之间都会同步密钥。
如果不指定，默认：https://keys.openpgp.org
  - --keyserver-options：导入或导出密钥时的一些选项，
支持所有 --import-options 和 --export-options，另外还支持像 http-proxy 来指定代理等。
  - \<key-id\>：如果不指定将会导出所有公钥或私钥，如果指定仅会导出指定的密钥，
使用 `!` 可以仅导出主密钥或子密钥。参考：[选择密钥](#选择密钥)
- 示例：

      l@l:~$ gpg --search-keys 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      l@l:~$ gpg --recv-keys 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      l@l:~$ gpg --send-keys 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7

#### 编辑密钥

- 用法：

使用 `gpg --edit-key <key-id>` 命令，可以交互式的编辑一个密钥。
输入 `help` 可以查看帮助，输入 `save` 可以保存并退出编辑，输入 `quit` 可以不保存退出编辑。

- 示例：

      l@l:~$ gpg --edit-key 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      gpg (GnuPG) 2.2.27; Copyright (C) 2021 Free Software Foundation, Inc.
      This is free software: you are free to change and redistribute it.
      There is NO WARRANTY, to the extent permitted by law.
      
      私钥可用。
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1). Kang Lin <kl222@126.com>

      gpg> list
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1). Kang Lin <kl222@126.com>
      
      gpg> quit

##### 对 uid 签名

使用 `sign` 或 `lsign` 可以对 uid 进行签名。两者的区别是，
`sign` 的签名可以被导出和上传到密钥服务器，而 `lsign` 的签名仅可以在本地查看和生效。
一般我们验证了一个人的身份，如果没有得到公钥主人的允许，应该优先使用 `lsign` 进行签名。

如果需要撤销一个签名，可以使用 `revsig` 命令。
如果签名还没有发布（上传到密钥服务器或者导出给其他人），也可以使用 `delsig` 直接删除。

这里的签名将会使用默认的私钥，如果没有指定将会使用第一个可用的私钥。
或者也可以在 `--edit-key` 时通过 `-u` 参数指定一个私钥。

如果不指定某个 uid，将会对所有 uid 签名。也可以先通过 `uid [n]` 命令选择某个或某些 uid，
再进行签名。`uid 0` 将取消选择所有 uid。

##### 信任密钥

使用 `trust` 命令可以设置对密钥的信任值。
关于信任值的解释和每个信任值的含义参考：[信任网络](#信任网络)。

信任值是一个很主观的事情，因此信任值仅会保存到本地的数据库中，不会被导出或上传到数据库中。
除非使用 `--export-options backup` 选项导出。

##### uid

使用 `adduid` 命令可以创建一个新的 uid。并可以使用 `uid [n]` 命令选择一个 uid 后，
使用 `primary` 命令指定一个主 uid。

如果某个 uid 不再使用，可以先使用 `uid [n]` 选中，再使用 `revuid` 命令撤销。
如果 uid 还没有发布，也可以使用 `deluid` 直接删除。

- 示例：

      gpg> adduid
      真实姓名： Kang Lin
      电子邮件地址： kl222@qq.com
      注释： Kang Lin
      您选定了此用户标识：
          “Kang Lin (Kang Lin) <kl222@qq.com>”
      
      更改姓名（N）、注释（C）、电子邮件地址（E）或确定（O）/退出（Q）？ o
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1)  Kang Lin <kl222@126.com>
      [ 未知 ] (2). Kang Lin (Kang Lin) <kl222@qq.com>

      gpg> uid 2
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1)  Kang Lin <kl222@126.com>
      [ 未知 ] (2)* Kang Lin (Kang Lin) <kl222@qq.com>

注意：选中的 uid，用 `*` 表示。如果再次输入 uid 2，则取消选中。

      gpg> deluid 
      真的要移除此用户标识吗？(y/N) y
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1)  Kang Lin <kl222@126.com>
            


##### 子密钥

使用 `addkey` 命令可以创建一对子密钥。创建过程中可以交互式的选择使用的算法、用途等。

如果某个子密钥不再使用，可以先使用 `key [n]` 选中，再使用 `revkey` 命令撤销。
如果子密钥还没有发布也没有使用过，可以使用 `delkey` 直接删除。
但要注意，如果删除，使用此子密钥加密过的信息将永远无法解密！

- 示例：

      gpg> addkey
      请选择您要使用的密钥类型：
         (3) DSA（仅用于签名）
         (4) RSA（仅用于签名）
         (5) ElGamal（仅用于加密）
         (6) RSA（仅用于加密）
        （14）卡中现有密钥
      您的选择是？ 6
      RSA 密钥的长度应在 1024 位与 4096 位之间。
      您想要使用的密钥长度？(3072) 4096
      请求的密钥长度是 4096 位
      请设定这个密钥的有效期限。
               0 = 密钥永不过期
            <n>  = 密钥在 n 天后过期
            <n>w = 密钥在 n 周后过期
            <n>m = 密钥在 n 月后过期
            <n>y = 密钥在 n 年后过期
      密钥的有效期限是？(0) 
      密钥永远不会过期
      这些内容正确吗？ (y/N) y
      真的要创建吗？(y/N) y
      我们需要生成大量的随机字节。在质数生成期间做些其他操作（敲打键盘
      、移动鼠标、读写硬盘之类的）将会是一个不错的主意；这会让随机数
      发生器有更好的机会获得足够的熵。
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      ssb  rsa4096/0881F0D167CE7BB1
           创建于：2024-04-18  有效至：永不       可用于：E   
      [ 绝对 ] (1). Kang Lin <kl222@126.com>

      gpg> key 5
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      ssb* rsa4096/0881F0D167CE7BB1
           创建于：2024-04-18  有效至：永不       可用于：E   
      [ 绝对 ] (1). Kang Lin <kl222@126.com>

注意：选中的 key，用 `*` 表示。如果再次输入 key 5，则取消选中。

      gpg> delkey 
      您真的要删除此密钥吗？(y/N) y
      
      sec  rsa4096/B5AF93007DC8E9E7
           创建于：2021-08-02  有效至：永不       可用于：SC  
           信任度：绝对        有效性：绝对
      ssb  rsa4096/00C7D98BE886845F
           创建于：2021-08-02  有效至：永不       可用于：E   
      ssb  rsa4096/C7DFE5CE9DBABAAC
           创建于：2024-01-11  有效至：永不       可用于：A   
      ssb  rsa4096/E9729414857DE6C4
           创建于：2024-01-11  有效至：永不       可用于：S   
      ssb  rsa4096/B21A1E4D8BF6B2D1
           创建于：2024-01-18  有效至：永不       可用于：S   
      [ 绝对 ] (1). Kang Lin <kl222@126.com>

##### 撤销密钥

密钥服务器为了防止被篡改，被设计为只增不删的形式。也就是说如果密钥一旦上传到服务器，
将永远不能删除。那么如果主密钥丢失或者泄露了，我们只能撤销它。

不选择任何子密钥，执行 `revkey` 命令，将撤销整个密钥。
撤销之后，重新将密钥发送到密钥服务器，密钥服务器上的密钥也会被撤销。
其他人从密钥服务器更新之后，将会知道密钥已经被撤销，以后就不会再使用此密钥进行加密，
也将不会信任此密钥的签名。

上面的操作必须在我们拥有私钥的情况下进行，而如果私钥丢失了，我们就永远无法更新密钥的状态了。
所以为了避免出现这种情况，一般建议生成一个撤销证书，和私钥分开保存。
撤销证书本质上就是一个撤销密钥的签名。如果私钥丢失，需要将撤销证书导入，并发送到密钥服务器，
密钥服务器上的公钥也将被撤销。

- 生成撤销证书：`gpg [-a] [-o <output-file>] --gen-revoke <key-id>`
- 通过导入撤销证书撤销密钥：`gpg --import [<input-file>]`
- 发布撤销密钥：
  - 如果之前使用了密钥服务器，则发布到密钥服务器即可：`gpg --send-keys <key-id>`
  - 如果是手动导出的方式，则再次导出并发送给其他人导入即可：`gpg --export-keys <key-id>`

注意： 密钥一旦撤销将无法恢复。

其实在拥有私钥的情况下，无需撤销证书即可撤销整个密钥。
撤销证书单独保存的意义在于，他可以保存在比私钥稍低一个安全等级的地方。
因为一旦私钥泄露，以往的信息都将泄露。而如果安全证书泄露，只会使密钥失效，不会泄露任何信息。
最多是需要重新生成一对密钥发布给其他人，比较麻烦而已。
例如撤销证书可以放在信任的人那里备份，而私钥只能放在不会被任何人获取的地方备份。

- 示例：

        l@l:~$ gpg -a -o revoke.gpg --gen-revoke 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
        
        sec  rsa4096/B5AF93007DC8E9E7 2021-08-02 Kang Lin <kl222@126.com>
        
        要为这个密钥创建一个吊销证书吗？(y/N) y
        请选择吊销的原因：
         0 = 未指定原因
         1 = 密钥已泄漏
         2 = 密钥被替换
         3 = 密钥不再使用
         Q = 取消
        （也许您会想要在这里选择 1）
        您的决定是什么？ 1
        请输入描述（可选）；以空白行结束：
        > 
        吊销原因：密钥已泄漏
        （未给定描述）
        这样可以吗？ (y/N) y
        已创建吊销证书。
        
        请把这个文件转移到一个您可以藏起来的介质上；如果坏人获取到了这
        份证书的话，那么他就能使用它并让您的密钥无法继续使用。把此证书
        打印出来再存放到安全的地方也是很好的方法，以免您的保存媒体变得
        不可读。但是千万小心：您机器上的打印系统可能会在打印过程中储存
        这些数据，并使得其他人看到！

##### 过期时间

使用 `expire` 命令可以设置密钥的过期时间。默认会设置主密钥的，使用 key [n] 选中子密钥后，
将设置选中密钥的。

##### 密码

使用 `passwd` 命令可以修改密钥的密码。

##### 清理签名

如果 uid 上收集的很多签名无效了，例如撤销了的签名，不会删除原有的签名，而是在生成一条撤销的信息。
可以使用 `clean` 命令，清除所有无效的签名。

如果更进一步，不需要所有其他人的签名，则可以使用 `minimize` 清除自签名外的所有签名。

#### 删除主密钥

注意：操作这一步前，必须先备份 `~/.gnupg`。或者导出备份(包括所有私钥)。参见：[导出密钥](#导出密钥)

- 识别你的主密钥的 keygrip：

      l@l:~$ gpg --with-keygrip --list-key 
      /home/l/.gnupg/pubring.kbx
      --------------------------
      pub   rsa4096 2021-08-02 [SC]
            08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
            Keygrip = 7CEB022E02A216D66616102FCC21CC4ECBBF822C
      uid             [ 绝对 ] Kang Lin <kl222@126.com>
      sub   rsa4096 2021-08-02 [E]
            Keygrip = B0D6894533AEEEA63D9A6B801B2879B50ECB2268
      sub   rsa4096 2024-01-11 [A]
            Keygrip = DC61B8FE567C7B79524DBA849631241A94E67956
      sub   rsa4096 2024-01-11 [S]
            Keygrip = B7B07DCB175E25F99336AB9FA25353A16D169CED
      sub   rsa4096 2024-01-18 [S]
            Keygrip = C79157D4BCABB74FF4AA035FCF34CE044420CB4C

- 找到 pub 行下方的 Keygrip 条目（就在主密钥指纹的下方）。
它与你的家目录下 .gnupg 目录下的一个文件是一致的：

      l@l:~/.gnupg/private-keys-v1.d$ ls
      7CEB022E02A216D66616102FCC21CC4ECBBF822C.key
      986EF8DDC633083C7CD696EC9FA8754376031B2D.key
      B0D6894533AEEEA63D9A6B801B2879B50ECB2268.key
      B7B07DCB175E25F99336AB9FA25353A16D169CED.key
      C79157D4BCABB74FF4AA035FCF34CE044420CB4C.key
      DC61B8FE567C7B79524DBA849631241A94E67956.key

- 删除与主密钥 keygrip 一致的 .key 文件：

      l@l:~$ cd ~/.gnupg/private-keys-v1.d
      l@l:~/.gnupg/private-keys-v1.d$ rm 7CEB022E02A216D66616102FCC21CC4ECBBF822C.key

- 参考： https://linux.cn/article-10402-1.html

#### 将子密钥移到一个硬件设备中

参见：https://linux.cn/article-10415-1.html

### 配置文件

前面的介绍中可以看到，很多时候我们都需要通过 -u 参数指定使用哪个密钥进行签名。
还有加密时如果不指定自己，以后将会无法解密自己发出的信息。
我们希望将这些参数默认配置下来，不用每次都在命令行中输入。

GPG 命令中的所有长参数 (--开头的参数) 都可以配置在 ~/.gnupg/gpg.conf 中，
这样每次执行 GPG 命令时，都会默认带上这些参数。

例如指定默认的用于签名的密钥，指定每次加密都默认也加密给自己等等。

- 示例：

      default-key 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7
      encrypt-to 08C9D8FE20930F2A9BDF565AB5AF93007DC8E9E7

### 参考

- https://zhuanlan.zhihu.com/p/436962516
