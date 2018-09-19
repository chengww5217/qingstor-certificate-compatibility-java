### 关于青云对象存储更换 Let's Encrypt 证书对应 Java 问题说明

青云对象存储原 https 证书将于 2018.10.11 过期，届时将更换为 Let's Encrypt 新证书。Let's Encrypt 是由 Mozilla、Cisco、Akamai、IdenTrust、EFF 等组织人员发起，主要的目的是为了推进网站从 HTTP 向 HTTPS 过度的进程，目前已经有越来越多的商家加入和赞助支持，其证书现在已经可以被所有主流的浏览器所信任。

因 Let's Encrypt 证书较新，下列 JDK/JRE 旧版本会不信任 Let's Encrypt 证书（查看您的 Java 版本：命令行输入 ``` java -version ```）：

-  Java 7 < 7u111
-  Java 8 < 8u101

而抛出以下异常：

```
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException
[... 以下输出省略 ...]
```

请您参考以下方法查看您的 Java 环境是否支持 Let's Encrypt 证书。

#### 1.检查您的业务是否受到影响
私有云和设置使用 http 连接（不安全）的用户不会对您的业务造成任何影响。

其他用户请进行以下检查以确认您的业务是否受到影响：

#### 1.1.检查您的 java 环境是否支持 Let's Encrypt 证书
##### 1.1.1.自行编写程序测试
您可以自行编写测试程序查看您的 Java 环境是否支持 Let's Encrypt 证书：
``` new URL("https://helloworld.letsencrypt.org").openConnection().connect(); ```

然后查看是否抛出以下异常即可：

```
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException
[... 以下输出省略 ...]
```

##### 1.1.2.使用 SSLPing 进行测试
如您还没有测试程序，您可以使用 ping 测试程序：SSLPing（可测试任何 SSL/TLS 端口，不仅是 HTTPS）。下面将使用预先编译的 SSLPing.jar 进行测试（阅读源码后自行编译也非常容易）：

在命令行输入以下内容以克隆 SSLPing 这个项目（请确保您已安装 git）或点击链接下载 [SSLPing.jar](https://github.com/dimalinux/SSLPing/raw/master/dist/SSLPing.jar)

```git
git clone https://github.com/dimalinux/SSLPing.git
```

成功后进行测试：

```git
java -jar SSLPing/dist/SSLPing.jar helloworld.letsencrypt.org 443
```

如您的 JDK/JRE 版本是否在以下范围（命令行输入 ``` java -version ```）：

-  Java 7 < 7u111
-  Java 8 < 8u101

Let's Encrypt 证书将不会被信任而抛出以下异常：

```java
About to connect to 'helloworld.letsencrypt.org' on port 443
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException
[... 以下输出省略 ...]
```

*关于 Let's Encrypt 证书完整兼容性情况参见 [https://letsencrypt.org/certificates/](https://letsencrypt.org/certificates/)*

#### 2.问题解决
当您出现以上问题时，您可以参考以下方式进行解决（任选其一）：

#### 2.1.往 JRE 中导入 Let's Encrypt 证书（无需修改代码，推荐）
您可以将 Let's Encrypt 证书加入您 JRE 的信任证书，这种方式无需修改您的代码，简单快捷，推荐使用。
##### 2.1.1.您的操作系统为 Mac OS X 或 Linux
- 检查 __JAVA_HOME__ 已经正确配置

在终端上输入以下内容：

```
echo $JAVA_HOME 
```

出现以下类似回应即为正确配置：

```
/Library/Java/JavaVirtualMachines/jdk1.8.0_92.jdk/Contents/Home/
```

如未配置请自行搜索配置 Java 环境变量即可。

- 下载  Let's Encrypt 中间证书

```
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem
```

或直接复制上述链接下载即可

- 导入证书

```
keytool -trustcacerts -keystore "$JAVA_HOME/jre/lib/security/cacerts" -storepass changeit -noprompt -importcert -alias lets-encrypt-x3-cross-signed -file "lets-encrypt-x3-cross-signed.pem"
```

出现 ``` Certificate was added to keystore ``` 即可

**（注：当出现 java.io.FileNotFoundException... 时您可能要检查相关文件路径是否正确）**

##### 2.1.2.您的操作系统为 Windows
- 检查 __JAVA_HOME__ 已经正确配置

在命令行上输入以下内容：

```
echo %JAVA_HOME%
```
出现以下类似回应即为正确配置：

```
C:\Program Files (x86)\Java\jdk1.8.0_92
```

如未配置请自行搜索配置 Java 环境变量即可。

- 下载  Let's Encrypt 中间证书

```
wget https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem
```

或直接复制上述链接下载即可

- 导入证书

```
cd %JAVA_HOME%\bin
keytool -trustcacerts -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -storepass changeit -noprompt -importcert -alias lets-encrypt-x3-cross-signed -file "lets-encrypt-x3-cross-signed.pem"
```

出现 ``` Certificate was added to keystore ``` 即可

**（注：当出现 java.io.FileNotFoundException... 时您可能要检查相关文件路径是否正确）**

该方法至此已经完成，请检测是否成功。

---------

#### 2.2.程序运行时添加 Let's Encrypt 证书为信任证书
您也可以在您的程序初始化或网络初始化时将 Let's Encrypt 证书添加进信任证书。

使用火狐浏览器访问 [https://helloworld.letsencrypt.org](https://helloworld.letsencrypt.org) ，然后将 ``` Let's Encrypt Authority X3 ``` 导出为 ``` .cer ``` 文件，或点击下载 [Let's Encrypt Authority X3.cer](https://lets-encrypt.pek3a.qingstor.com/Let's%20Encrypt%20Authority%20X3.cer)

将上述文件地址替换下述文件地址： ``` "Let's Encrypt Authority X3.cer" ```

具体请参考以下示例添加:

```java
/**
 * Created by chengww on 2018/9/18.
 */

import java.io.IOException;
import java.net.URL;
import java.net.URLConnection;

import javax.net.ssl.SSLHandshakeException;

public class SSLExample {

    private static void initTrustManager() {
        // Enter the path of the file named 'Let's Encrypt Authority X3.cer'
        System.setProperty("javax.net.ssl.trustStore", "Let's Encrypt Authority X3.cer");
    }

    public static void main(String[] args) throws IOException {
        initTrustManager();
        
        // signed by default trusted CAs.
        testUrl(new URL("https://www.thawte.com"));

        // signed by letsencrypt
        testUrl(new URL("https://helloworld.letsencrypt.org"));
        // signed by LE's cross-sign CA
        testUrl(new URL("https://letsencrypt.org"));
        // qingstorage
        testUrl(new URL("https://stor.qingstorage.com"));
        // qingcloud
        testUrl(new URL("https://www.qingcloud.com/"));
        // self-signed
        testUrl(new URL("https://www.pcwebshop.co.uk/"));


    }

    static void testUrl(URL url) throws IOException {
        URLConnection connection = url.openConnection();
        try {
            connection.connect();
            System.out.println("Headers of " + url + " => "
                    + connection.getHeaderFields());
        } catch (SSLHandshakeException e) {
            System.out.println("Untrusted: " + url);
        }
    }

}
```

该方法至此已经完成，请检测是否成功。

---------
#### 2.3.升级 JDK/JRE 版本
您可以直接升级您的 JDK/JRE update 版本，7u111 及 8u101 之后已将 Let's Encrypt 证书加入信任。

前往 [https://www.oracle.com/technetwork/java/javase/downloads/index.html](https://www.oracle.com/technetwork/java/javase/downloads/index.html) 下拉到最后一项 Java Archive，点击**DOWNLOAD**

![Java_Archive.png](https://lets-encrypt.pek3a.qingstor.com/screenShots/Java_Archive.png)

选择 Accept License Agreement，下载您对应的 JDK/JRE 版本后安装即可

![Accept_License_Agreement.png](https://lets-encrypt.pek3a.qingstor.com/screenShots/Accept_License_Agreement.png)

该方法至此已经完成，请检测是否成功。

---------
### 2.4.检查是否成功
重复操作上述 **1.1.检查您的 java 环境是否支持 Let's Encrypt 证书** 的内容即可。

如您对上述检查还心存疑虑，您可以使用以下配置来检测是否成功（不抛出异常即可）：

```
// 最新版 qingstor-sdk-java 已将 EvnContext 更名为 EnvContext，原名称保留，使用原名称也没有任何业务上的影响
EnvContext context = new EnvContext("ZYRDZVLWBNDRRGQCDDCP", "Z01aX1QymRC3FGqqOZAaZNjoTMr22vJGg91A80ET");
context.setProtocol("https");
context.setHost("stor.qingstorage.com");
String zoneKey = "example1";
String bucketName = "demobucket3";
Bucket bucket = new Bucket(context, zoneKey, bucketName);
Bucket.PutObjectInput input = new Bucket.PutObjectInput();
input.setContentType("application/x-directory");
try {
    Bucket.PutObjectOutput output = bucket.putObject("your folder name", input);
    if (output.getStatueCode() == 200 || output.getStatueCode() == 201) {
        System.out.println("Create a directory success.");
    } else {
        System.out.println("Create a directory failed.");
        System.out.println("StatusCode = " + output.getStatueCode());
        System.out.println("RequestId = " + output.getRequestId());
        System.out.println("Code = " + output.getCode());
        System.out.println("Message = " + output.getMessage());
        System.out.println("Url = " + output.getUrl());
    }
} catch (QSException e) {
    e.printStackTrace();
}
```

在您的使用过程中有任何问题欢迎随时提工单和我们进行联系，祝您生活愉快，谢谢!
