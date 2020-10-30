# Gradle



## 一、项目结构

![image-20201029165007022](Gradle.assets/image-20201029165007022.png)

1. 包装文件的生成文件夹

2. Gradle包装器启动脚本

3. 设置文件以定义构建名称和子项目

   ```groovy
   rootProject.name = 'demo'  // 根项目名称
   include('app', 'list', 'utilities')  // 包含的子项目，也可以分开按行写，同下两行
   // include 'app'
   // lude 'list'
   ```

4. buildSrc的构建脚本以配置构建逻辑的依赖关系

   ```groovy
   %  buildSrc /build.gradle
   
   plugins {
       id 'groovy-gradle-plugin'  // 通过这个插件，支持编写约定插件
   }
   
   repositories {
       gradlePluginPortal()  // 添加依赖库
   }
   ```

5. 用Groovy或Kotlin DSL编写的约定插件的源文件夹

   ```groovy
   %  buildSrc /src/main/groovy/demo.java-common-conventions.gradle
   
   plugins {
       id 'java'  // Java插件，用于构建Java项目的功能
   }
   
   repositories {
       jcenter()  // 存储库，作为外部依赖项的源
   }
   
   dependencies {  // 标准依赖项
       testImplementation 'org.junit.jupiter:junit-jupiter-api:5.6.2' 
       testRuntimeOnly 'org.junit.jupiter:junit-jupiter-engine' 
   }
   
   tasks.named('test') {
       useJUnitPlatform() 
   }
   ```

   ```groovy
   %  buildSrc /src/main/groovy/demo.java-library-conventions.gradle
   
   plugins {
       id 'demo.java-common-conventions'  // 引用上面定义的common插件，共享通用配置
       id 'java-library'  // 更细的分类插件引用
   }
   ```

   ```groovy
   %  buildSrc /src/main/groovy/demo.java-application-conventions.gradle
   
   plugins {
       id 'demo.java-common-conventions' 
       id 'application' 
   }
   ```

6. 三个子项目的构建脚本app，list以及utilities

   ```groovy
   %  app/build.gradle
   
   plugins {  // 每个构建脚本都应有一个plugins{}块来应用插件
       id 'demo.java-application-conventions'
   }
   
   dependencies { // 项目依赖
       implementation project(':utilities')
   }
   
   application {
       mainClass = 'demo.app.App'  // 插件可能有一个或多个配置块。仅当它们为一个项目配置特定的东西时，才应直接在构建脚本中使用它们。否则，此类配置也属于约定插件。在这个例子中，我们使用的application {}块，这是特定的application插件，设置mainClass在我们的app项目demo.app.App
   }
   ```

7. 每个子项目中的Java源文件夹

8. 子项目中的Java测试源文件夹

### 1.1 Java项目结构





## 二、Gradle构建基础

### 2.1 Hello world!

```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

```bash
# 执行
> gradle -q hello
Hello world!
```

gradle脚本支持所有Groovy功能。

#### 2.1.1 任务依赖性

```groovy
task hello {
    doLast {
        println 'Hello world!'
    }
}
task intro {
    dependsOn hello // 可以在hello定义声明前依赖
    doLast {
        println "I'm Gradle"
    }
}
```

```bash
# 执行
> gradle -q intro
Hello world!
I'm gradle
```

#### 2.1.2 动态任务

```groovy
4.times { counter ->
    task "task$counter" {
        doLast {
            println "I'm task number $counter"
        }
    }
}
```

```bash
# 执行
>gradle -q task1
I'm task number 1
```

#### 2.1.3 也可以处理现有任务

```groovy
// 复用2.1.2内容
task0.dependsOn task2, task3
```

```bash
# 执行
> gradle -q task0
I'm task number 2
I'm task number 3
I'm task number 0
```

#### 2.1.4 通过API访问任务，添加行为

```gr
task hello {
    doLast {
        println 'Hello Earth'
    }
}
hello.doFirst {
    println 'Hello Venus'
}
hello.configure {
    doLast {
        println 'Hello Mars'
    }
}
hello.configure {
    doLast {
        println 'Hello Jupiter'
    }
}
```

```bash
# 执行
> gradle -q hello
Hello Venus
Hello Earth
Hello Mars
Hello Jupiter
```

#### 2.1.5 额外任务属性

https://docs.gradle.org/current/userguide/writing_build_scripts.html#sec:extra_properties

```groovy
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties {
    doLast {
        println myTask.myProperty
    }
}
```

```bash
# 执行
> gradle -q printTaskProperties
myValue
```

#### 2.1.6 默认任务

gradle时不指定任务，则执行定义的一个或多个默认任务

```groovy
defaultTasks 'clean', 'run'

task clean {
    doLast {
        println 'Default Cleaning!'
    }
}

task run {
    doLast {
        println 'Default Running!'
    }
}

task other {
    doLast {
        println "I'm not a default task!"
    }
}
```

```bash
# 执行
> gradle -q
Default Cleaning!
Default Running!
```

#### 2.1.7 构建脚本的外部依赖关系

`buildscript()`方法配置外部依赖项

```groovy
import org.apache.commons.codec.binary.Base64

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'commons-codec', name: 'commons-codec', version: '1.2'
    }
}

task encode {
    doLast {
        def byte[] encodedString = new Base64().encode('hello world\n'.getBytes())
        println new String(encodedString)
    }
}
```





















