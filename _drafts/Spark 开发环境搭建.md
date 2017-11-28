## Spark 开发环境搭建

###　安装java开发工具包JDK

```
    sudo apt-get install openjdk-
```

### 设置ssh无密码登录

### 下载安装Hadoop

```
    sudo mv hadoop-X.Y.Z /usr/local/hadoop
```

+ 配置Hadoop环境变量

```
    sudo gedit ~/.bashrc
```

+ 修改Hadoop配置文件

+ 创建格式化HDFS目录

+ 启动Hadoop

### 安装scala

```
    sudo mv scala-X.Y.Z /usr/local/scala
```

+ 设置scala环境变量

```
    sudo gedit ~/.bashrc

    export SCALA_HOME=/usr/local/scala
    export PATH=$PATH:$SCALA_HOME/bin
```


### 安装Spark

```
    sudo rm spark-X.Y.Z-bin-hadoopX.Y.Z /usr/loca/spark
```

+ 编辑Spark环境变量

+ 设置Spark显示信息为WARN

###　安装SBT

SBT目录结构

```
proj/
├── build.sbt
├── project
│   ├── build.properties
│   ├── Build.scala
│   └── plugins.sbt
└── src
    ├── main
    │   ├── java
    │   ├── resources
    │   └── scala
    └── test
        ├── java
        ├── resources
        └── scala
```

#### src 目录

+ src/main/java 目录存放Java源代码文件
+ src/main/resources 目录存放相应的资源文件
+ src/main/scala 目录存放Scala源代码文件
+ src/test/java 目录存放Java语言书写的测试代码文件
+ src/test/resources 目录存放测试起见使用到的资源文件
+ src/test/scala  目录存放scala语言书写的测试代码文件

####　build.sbt

build定义文件,放在项目根目录下,是一种简化的build定义.

```
  name := "hello"      // 项目名称   
  organization := "xxx.xxx.xxx"  // 组织名称   
  version := "0.0.1-SNAPSHOT"  // 版本号   
  scalaVersion := "2.9.2"   // 使用的Scala版本号
```

在build.sbt中添加项目依赖:

```
libraryDependencies += "ch.qos.logback" % "logback-core" % "1.0.0"   
libraryDependencies += "ch.qos.logback" % "logback-classic" % "1.0.0"
```

或者

```
libraryDependencies ++= Seq(                              
  "ch.qos.logback" % "logback-core" % "1.0.0",                             
  "ch.qos.logback" % "logback-classic" % "1.0.0",                             
  ...                             
  )
```

#### project目录

+ build.properties 声明使用的要使用哪个版本的SBT来编译当前项目,高版本的sbt boot launcher可以兼容编译低版本.

+ plugins.sbt 声明当前项目使用的插件,如assembly功能,清理ivy local cache功能等.

```
  resolvers += Resolver.url("git://github.com/jrudolph/sbt-dependency-graph.git")   
  resolvers += "sbt-idea-repo" at "http://mpeltonen.github.com/maven/"   
  addSbtPlugin("com.github.mpeltonen" % "sbt-idea" % "1.1.0")   
  addSbtPlugin("net.virtual-void" % "sbt-dependency-graph" % "0.6.0")
```


#### Target 目录

以上目录和文件通常是在创建项目的时候需要我们创建的，实际上， SBT还会在编译或者运行期间自动生成某些相应的目录和文件，比如SBT会在项目的根目录下和project目录下自动生成相应的target目录，并将编译结果或者某些缓存的信息置于其中，一般情况下，我们不希望将这些目录和文件记录到版本控制系统中，所以，通常会将这些目录和文件排除在版本管理之外。  

比如，如果我们使用git来做版本控制，那么就可以在.gitignore中添加一行"target/"来排除项目根目录下和project目录下的target目录及其相关文件。


###　安装IntellJ

+ 安装scala插件

+ 使用sbt创建项目

+ 添加library

将spark/jars中的jar包加入library中,

+ 配置log4j

在src/main目录下创建log4j.properties,内容从spark根目录下conf目录中拷出来.

将第一行的log4j.rootCategory=INFO, console改成log4j.rootCategory=ERROR, console，只显示ERROR级别的日志。

+ 运行测试程序sparkPi.scala

点击“Run”–> “Run Configurations”，在弹出的框中对应栏中 Program argument 填写“local”，表示将该参数传递给main函数，如下图所示，之后点击“Run”–> “Run”运行程序即可。

```scala
  import scala.math.random
  import org.apache.spark._

  object sparkPi{
    def main(args:Array[String]){
      val conf = new SparkConf().setAppName("Simple Application").setMaster("local")
      val sc = new SparkContext(conf)
      val slices = 2
      val n = 100000 * slices
      val count = sc.parallelize(1 to n, slices).map{i=>
        val x = random*2 -1
        val y = random * 2 -1
        if (x*x + y*y <1) 1 else 0}.reduce(_+_)
      println("Pi is roughly" + 4.0 * count /n)
    }
  }
```

+ 将程序打jar包

1. 建立artifacts

依次选择“File”–> “Project Structure” –> “Artifact”，选择“+”–> “Jar” –> “From Modules with dependencies”，选择main函数，并在弹出框中选择输出jar位置，并选择“OK”。

![PNG]

最后依次选择“Build”–> “Build Artifact”编译生成jar包.
