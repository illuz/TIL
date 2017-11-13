# Gradle, Go



## 用 Gradle build Java

### 介绍

Gradle 是 和 Ant, Maven 类似的构建工具，不过它不是用 XML 而是用 Groovy DSL 来编写构建语法，并能支持多个项目的构建，以及增量构建。

目前 Gradle 主要致力于 Java, Scala, Groovy 的项目构建，后面可能会支持其它语言，官方已经写了是支持 C++ 的。
它还是 Android 的官方钦定构建工具。

Gradle 的特性：

- Groovy DSL
- 支持 C/C++ 构建，并可扩展成支持其它语言
- 多个项目构建
- 增量构建
- Gradle Deamon，可以用一个守护进程监视项目变动，实现内存中的热更新
- 兼容 Maven 的 POM 配置
- 其它看 https://gradle.org/features/

User Guide: https://docs.gradle.org/current/userguide/userguide.html


### 使用

#### 命令

用 homebrew 装 gradle。

- `gradle tasks` 查看现在可以做的任务
- `gradle dist` 可以直接构建 dist 项目，用 `di` 缩写也可以
- `-b, --build-file` 指定构建文件
- `-p, --project-dir`  指定构建的项目目录
- `gradle dependencies` 可以看项目依赖
- `gradle properties` 盾项目属性

#### 使用 wrapper 来实现不同项目不同 gradle 版本

方法用 `gradle wrapper --gradle-version 2.0` 来生成 wrapper，之后只需要用目录下的 `gradlew` 来运行命令就行了。

#### 版本管理

```groovy
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

**dependencies** 
可以用 `compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'` 
也可以简单点： `compile 'org.hibernate:hibernate-core:3.6.7.Final'`

**repositories** 
Maven central: `mavenCentral()`
Bintray’s JCenter: `jcenter()`
其它 maven 库: `maven { url "http://repo.mycompany.com/maven2" }`
其它 ivy 库：`ivy { url "http://repo.mycompany.com/repo" }`
本地 ivy 库: `ivy { url "../local-repo" }`

#### 打包与发包（Java）

**打包**的话，就直接用 java 插件的 jar 任务配置 menifest 就行了，

```groovy
sourceCompatibility = 1.7
version = '1.0'
jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart',
                   'Implementation-Version': version,
                   'Main-Class': 'xxx.appMainClass',
                   'Class-Path': 'my-lib.jar'
    }
}
```

关于 jar 的配置选项看 oracle 官方文档：https://docs.oracle.com/javase/8/docs/technotes/guides/jar/jar.html

或者也可以用 spring-boot 插件打包成 fat-jar，具体看这文章：https://segmentfault.com/a/1190000008422142

```groovy
// from https://spring.io/guides/gs/gradle/#_declare_dependencies
apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'application'

mainClassName = 'hello.HelloWorld'

// tag::repositories[]
repositories {
    mavenCentral()
}
// end::repositories[]

// tag::jar[]
jar {
    baseName = 'gs-gradle'
    version =  '0.1.0'
}
// end::jar[]

// tag::dependencies[]
sourceCompatibility = 1.8
targetCompatibility = 1.8

dependencies {
    compile "joda-time:joda-time:2.2"
    testCompile "junit:junit:4.12"
}
// end::dependencies[]

// tag::wrapper[]
// end::wrapper[]
```



gradle **发包**会比 maven 麻烦一点，如果要像 maven 一样打包的话，要用 maven 插件，要为各个动作配 task：

```
uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
            pom.version = '1.0Maven'
            pom.artifactId = 'myMavenName'
        }
    }
}
install {
    repositories.mavenInstaller {
        pom.version = '1.0Maven'
        pom.artifactId = 'myName'
    }
}
// 更多看文档 https://docs.gradle.org/current/userguide/maven_plugin.html
```

#### 插件

plugins 分两种，一种是 script 一种是 binary。

script 一般是用 `apply from: 'xxx.gradle'`，binary 一般是用 `apply plugin: xxx`。

DSL  玩法：

```groovy
plugins {
  id "org.gradle.sample.hello" version "1.0.0" apply false
  id "org.gradle.sample.goodbye" version "1.0.0" apply false
}

subprojects { subproject ->
    if (subproject.name.startsWith("hello")) {
        apply plugin: 'org.gradle.sample.hello'
    }
    if (subproject.name.startsWith("goodbye")) {
        apply plugin: 'org.gradle.sample.goodbye'
    }
}
```

#### 总结

Gradle 十分灵活，基本上就是引入 plugin，然后配置参数或者重写 task，来使用插件。

现在社区也有不少别人编写的实用的 plugin，自己写一个的成本也不是很高。

所以理论上来说，它可以构建任何的语言，而且如果花心思能兼容很多东西，比如 maven。

### 其它一些东西

- [Kotlin-based DSL](https://github.com/gradle/kotlin-dsl) 
- [build scans](https://gradle.com/build-scans)：web 形式可交互的构建界面。

## Go 笔记

这次主要是从写算法题上来看 go 语法的。

#### 输入输出

##### fmt 包

- 输入：`fmt.Scanf` 和 C 的 scanf 差不多，`fmt.Sscanf` 和 C 的 sscanf 差不多
- 输出：`fmt.Println` 就是 Println

输出和各种语言的占位符差不多，go 则是增加了一些更详细的语法

```
%v	the value in a default format
	when printing structs, the plus flag (%+v) adds field names
%#v	a Go-syntax representation of the value
%T	a Go-syntax representation of the type of the value
%%	a literal percent sign; consumes no value
...
```

Go 的输入输出比 C/C++ 更方便的是，可以直接用 `fmt.Scan` 和 `fmt.Print` 来按变量的默认格式输入输出。

