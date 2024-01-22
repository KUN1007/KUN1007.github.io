# GPG key 实现 GitHub `Verified` 认证（Authenticate & Sign）

## 介绍

有朋友和我说我 GitHub 的提交全没有 `Verified` 绿色的验证，不安全，不能证明是我自己提交的

我之前不知道有 GPG key 这个东东，前几个月才知道，今天找机会加一下这个东东

我参考了这篇文章：[GnuPG 密钥生成三步走](https://lab.jinkan.org/2020/04/30/gnupg-in-three-steps/)

这篇文章步骤很详细，对着这篇文章操作完理应得到下面的文件

```bash
sec  rsa4096/70A05553D73D8AFB
     created: 2024-01-22  expires: never       usage: C
     trust: ultimate      validity: ultimate
ssb  ed25519/1845322DB3B9DDB2
     created: 2024-01-22  expires: 2027-01-21  usage: S
ssb  cv25519/71F28CE7A795B7AF
     created: 2024-01-22  expires: 2027-01-21  usage: E
ssb  ed25519/DFE9D95DFEFF728A
     created: 2024-01-22  expires: 2027-01-21  usage: A
[ultimate] (1). KUN1007 (KUN IS THE CUTEST!) <yuyuyukunkunkun@gmail.com>
```

这个 C 是主密钥，下面的 `ssb` 都是子密钥，我和文章中设置的都是 `3y` 的过期时间

S, E, A 这三种 subkeys 分别为: 签名（Sign）、加密（Encrypt）和认证（Authenticate）

## GitHub Verified

就是提交的时候会出现 `Verified` 的绿色文字，证明这个提交是自己提交的，看着更安全

现在来操作一下，首先打开 `GitHub -> settings -> SSH and GPG keys`

我们要添加一个用于认证的 GPG key，也就是那个 `A` 的 subkey，现在我们要找到这个 subkey 的 ID 从而拿到这个 subkey 的公钥

### 找到 subkeys 的 ID

我们使用下面的命令可以列出目前所有 key 的 ID

```bash
~ gpg --keyid-format LONG --list-keys

[keyboxd]
---------
pub   rsa4096/70A05553D73D8AFB 2024-01-22 [C]
      D28034F34AE747A6FE75C0F570A05553D73D8AFB
uid                 [ultimate] KUN1007 (KUN IS THE CUTEST!) <yuyuyukunkunkun@gmail.com>
sub   ed25519/1845322DB3B9DDB2 2024-01-22 [S] [expires: 2027-01-21]
sub   cv25519/71F28CE7A795B7AF 2024-01-22 [E] [expires: 2027-01-21]
sub   ed25519/DFE9D95DFEFF728A 2024-01-22 [A] [expires: 2027-01-21]
```

### 拿到 `A` 这个 subkey 的公钥

```bash
gpg --armor --export DFE9D95DFEFF728A
```

这个公钥大概长这样

:::details
-----BEGIN PGP PUBLIC KEY BLOCK-----

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
k7vpqxyxkieawEDktYOJq0WClV4BRw==
=P2or
-----END PGP PUBLIC KEY BLOCK-----
:::

注意，连着 `-----BEGIN PGP PUBLIC KEY BLOCK-----` 和 `-----END PGP PUBLIC KEY BLOCK-----` 都算

### 复制这个公钥到 GitHub `Add new GPG key` 窗口里

![](https://cdn.jsdelivr.net/gh/kun-moe/kun-image@main/blog/202401221226202.png)

然后添加上就可以了


### 设置 Git 提交自动签名认证

签名的过程需要用到 `S` 这个 subkey，我们按照上面一样的方法获得它的 ID

然后使用 Git 借助这个 `S` 密钥进行签名

```bash
~ git config --global user.signingkey 1845322DB3B9DDB2
```

然后设置 Git 提交的时候自动签名

```bash
git config --global commit.gpgsign true
```

好，让我看看这个提交有没有生效
