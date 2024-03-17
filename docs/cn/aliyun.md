添加阿里云镜像作为gradle下载源
================
在根目录下的build.gradle文件中的`repositories`标签内添加如下内容 
```
repositories {
        maven {
            url = 'https://maven.aliyun.com/repository/public'
        }
        gradlePluginPortal {
            url = 'https://maven.aliyun.com/repository/gradle-plugin'
        }
    }
```
