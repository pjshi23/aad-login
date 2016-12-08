
# 中文版 #


首先，我们在中国azure里新建一台Ubuntu。

我这里演示用的是14.04，也欢迎测试新的版本或者centos来给我提issue。

    Sudo useradd -m podvma
    Sudo useradd -m stanpeng

这里的user就是我们的aad 用户名，这里以podvma为例。

新建完毕后，


    git clone https://github.com/pjshi23/aad-login.git

 
    Sudo apt-get install npm
    sudo tar xzf aad-login_0.1.tar.gz -C /
    cd /opt/aad-login
    sudo npm install

安装完毕后，nodejs程序就已经在你的 /opt/aad-login里了。

我们回到azure ad.

这里我们创建我们要用的用户，我这里新建了podvma和stanpeng 2个账号作为测试。




新建的用户记得登录一下更改temp password。

新建完毕后，点击应用程序并添加新的native client. Reply uri可以添加http://localhost:8000等。

我这里新建一个linuxlogin的app



记住这个client id，一会要在configuration file里填入。

现在回到VM，进入修改/opt/aad-login/aad-login.js

添加你的用户名directory->例如 stan@gmail.com即gmail

    var directory = '<gmail.com>';
    var clientid  = '<native client id>';
这样就设置完毕了。

我们快速测试一下：

    Nodejs /opt/aad-login/aad-login.js podvma <mypasswd>



拿到token，说明认证成功。

接着我们把/opt/aad-login/aad-login copy到/usr/local/bin/aad-login里。

然后我们打开/etc/pam.d/common-auth, 在最上面添加：


    auth sufficient pam_exec.so expose_authtok /usr/local/bin/aad-login

这样我们就能用Pam来获取我们拿到的token了

这时，你就已经能够通过你的aad 帐号密码登录这台Linux机器了，是不是很方便？：



同理，我们也可以轻松给新建的aad user stanpeng添加登录权限。 到此为止，所有的配置和设置都已经完成。欢迎测试和试用。

这里仅提供一个想法，实际生产环境请根据实际情况部署 ：）




----------






#this is forked from Bureado. Modified for Mooncake

# aad-login

Allows Linux user authentication to Azure AD via pam_exec

## Prerequisites

* An Azure AD directory has been created, and some users exist
* Node.js and npm are installed in the Linux VM
* A directory application has been created (native client type) and you have the Client ID
* Your PAM distribution has pam_exec.so

## User provisioning

This utility doesn't provision the user. In other words, you need to ensure the user
you'll be logging in with is visible by NSS. A simple `sudo useradd -m <user>` might
be enough for a handful of users.

Don't forget to add sudoer list and default profile for your new user

## Installing

You can download the tarfile and:

    sudo tar xzf aad-login_0.1.tar.gz -C /
    cd /opt/aad-login
    sudo npm install

## Configuring

First, open `/opt/aad-login/aad-login.js` with your favorite editor and put your directory
and client ID in.

Then, open `/etc/pam.d/common-auth` and add:

    auth sufficient pam_exec.so expose_authtok /usr/local/bin/aad-login

ideally at the beginning of your ruleset. Other rules might need to use `try_firstpass` for
convenience.

CentOS doesn't have `common-auth` so you need to include this rule in the relevant PAM file,
such as `/etc/pam.d/sshd` or `/etc/pam.d/system-auth`.

## Caveats

A freshly created user will have a temporary password that has to be changed via the portal. A
convenient way to get this done is to visit portal.azure.com (even if you don't have an Azure
account) with those credentials and change them before attempting to SSH.

In CentOS 7.x (and other SELinux-enabled distros) you need to disable the policy:

    sudo setenforce 0

The self-provisioning beta doesn't guarantee UID consistency across VMs, nor delegates access
to groups like sudo. Therefore, an important TODO is to detect group membership.

## Warnings

This is sample code and comes with no warranties.

Tested in Ubuntu 14.04. Any changes to common-auth might result in unexpected behaviour in
authentication including multiple password prompts and inability to join with local credentials.



----------


