![timg (2).jpeg](https://upload-images.jianshu.io/upload_images/18406403-19dfb490c1dcc92c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在天朝使用jcenter、mavenCentral及google三个远程仓库，Gradle Sync会很慢，goole仓库甚至需要[科学上网](https://github.com/hugetiny/awesome-vpn)才能访问。为了加快Gradle Sync速度，一招教你优先用 [阿里云仓库服务](https://maven.aliyun.com/mvn/view) 的仓库作为下载源。

### Maven仓库列表
|  仓库名 | 简介  | 实际地址 | 使用地址 |
|  ----  | ----  | ----  | ----  |
| jcenter | JFrog公司提供的仓库 | http://jcenter.bintray.com | https://maven.aliyun.com/repository/jcenter 
https://maven.aliyun.com/nexus/content/repositories/jcenter |
| mavenLocal | 本台电脑上的仓库 | {USER_HOME}/.m2/repository | C:/Users/liyujiang/.m2/repository (Windows) 
/home/liyujiang/.m2/repository (Linux) |
| mavenCentral | Sonatype公司提供的中央库 | http://central.maven.org/maven2 | https://maven.aliyun.com/repository/central 
https://maven.aliyun.com/nexus/content/repositories/central |
| google | Google公司提供的仓库 | https://maven.google.com | https://maven.aliyun.com/repository/google 
https://maven.aliyun.com/nexus/content/repositories/google
https://dl.google.com/dl/android/maven2 |
| jitpack | JitPack提供的仓库 | https://jitpack.io | https://jitpack.io |
| public | jcenter和mavenCentral的聚合仓库 | https://maven.aliyun.com/repository/public 
https://maven.aliyun.com/nexus/content/groups/public |
| gradle-plugin | Gradle插件仓库 | https://plugins.gradle.org/m2 | https://maven.aliyun.com/repository/gradle-plugin 
https://maven.aliyun.com/nexus/content/repositories/gradle-plugin |

### 阿里云代理仓库配置

在**项目根目录**下的`build.gradle`的`buildscript.repositories`及`allprojects.repositories`闭包内的最前面（Gradle是从上往下寻找的，故要放到jcenter()及google()的前面），添加阿里云仓库服务的代理仓库地址，示例如下：

```
buildscript {
    repositories {
        maven {
            url 'https://maven.aliyun.com/repository/jcenter'
        }
        maven {
            url 'https://maven.aliyun.com/repository/google'
        }
        jcenter()
        google()
    }
}

allprojects {
    repositories {
        maven {
            url 'https://maven.aliyun.com/repository/jcenter'
        }
        maven {
            url 'https://maven.aliyun.com/repository/central'
        }
        maven {
            url 'https://maven.aliyun.com/repository/google'
        }
        jcenter()
        mavenCentral()
        google()
    }
}
```
