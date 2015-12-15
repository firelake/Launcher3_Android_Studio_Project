#Gradle Study Notes
##Chapter 1
1. Build, project, task
```
BUILD
  |
  |
  1..* --> PROJECT
	  			|
	  			|
	  			1..* --> TASK
```
2. Gradle的构建系统是由以下几个文件组成
	1. build.gradle 我们称这个文件为一个构建脚本，这个脚本定义了一个模块和编译用的tasks，它一般是放在项目的模块中，也可以放在项目的根目录用来作为编译结构全局设置，它是必须的
	2. settings.gradle 它描述了哪一个模块需要参与构建。每一个多模块的构建都必须在项目结构的根目录中加入这个设置文件，它也是必须的
	3. gradle.properties 用来配置构建属性，这个不是必须的
3. Task 定义，doFirst, doLast,  task依赖
4. Plugin 插件：Gradle的设计理念是，所有有用的特性都由Gradle插件提供，例如编写一个Java项目时，需要使用到 Java 插件， 它会将许多任务自动的加入到你项目里。Gradle本身提供了一系列的标准插件，无需多余配置只需要在你的build.gradle文件中加入 apply plugin: 'java'
5. 依赖:Firstly, Gradle needs to know about the things that your project needs to build or run, in order to find them. We call these incoming files the dependencies of the project. Secondly, Gradle needs to build and upload the things that your project produces. We call these outgoing files the publications of the project.
	1. 外部依赖：依赖一个外部的jar
		1. 指定一个仓库的地址
		```groovy
		repositories {
		mavenCentral()
		}
		```
		2. 引用仓库中的外部依赖。引用一个外部依赖需要指定使用的group, name 和 version 属性，三者缺一不可。那从哪里得知JAR包的这三个属性呢？我们可以从mvnrepository中搜索到。
		```groovy
		dependencies {
		compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
		// 简化写法
		// compile 'commons-collections:commons-collections:3.2'
		}
		```
	2. 本地依赖：Gradle也可以从本地目录中引入JAR包依赖，可以单一引入指定的某一JAR包，也可以引入某目录下所有的JAR包
		```groovy
		dependencies {
		compile files('dir/file.jar')
		compile fileTree(dir: 'libs', include: '*.jar')
		}
		```
	3. 项目依赖：往往一个完整的项目由多个子项目构成。在Gradle中，使用文件settings.gradle定义当前项目的子项目。默认情况下，每个子项目的名称对应着当前操作系统目录下的一个子目录。
		1. settings.gradle
					include 'sub-project1', 'sub-project2', 'sub-project3'
		2. 如sub-project1依赖sub-project2，则在sub-project1的build.gradle中加入以下配置即可：
		```groovy
		ependencies {
		compile project(':sub-project2')
		}
		```
6. Shortcut notations: a convenient notation for accessing an existing task. Each task is available as a property of the build script
7. Default tasks: Gradle allows you to define one or more default tasks that are executed if no other tasks are specified. 
```groovy
defaultTasks 'task1', 'task2'
```
8. DAG:Gradle has a configuration phase and an execution phase. After the configuration phase, Gradle knows all tasks that should be executed. Gradle offers you a hook to make use of this information. 
```groovy
task distribution << {
    println "We build the zip with version=$version"
}

task release(dependsOn: 'distribution') << {
    println 'We release now'
}

gradle.taskGraph.whenReady {taskGraph ->
    if (taskGraph.hasTask(release)) {
        version = '1.0'
    } else {
        version = '1.0-SNAPSHOT'
    }
}
```
9. gradle java plugin
```groovy
apply plugin: 'java'
```
	1. src/main/java/
	2. src/test/java/
	3. src/main/resources/
	4. src/test/resources/
	5. build/
10. multi-project build
	1. basic hierarchy
		**projects**
		```
		multiproject/
		  api/
		  services/webservice/
		  shared/
		  services/shared/
		```
		**settings.gradle**
		```groovy
		include 'shared', 'api', 'services:webservice', 'services:shared'
		```
	2. common configuration

11. Closures as the last parameter in a method
	1. When the last parameter of a method is a closure, you can place the closure after the method call
	```
	// the following 3 forms is has the same result
	repositories {
    	println "in a closure"
	}
	repositories() { println "in a closure" }
	repositories({ println "in a closure" })
	```
12. Closure delegate: each closure has a delegate object, which Groovy uses to look up variable and method references which are not local variables or parameters of the closure.
##Chapter 14 - More About Tasks

##Chapter 18 - Gradle Daemon
1. When to use Daemon: it is recommended that the Daemon is used in all developer environments. It is recommend to not enable the Daemon for Continuous Integration and build server environments.
##Chapter 19 - Gradle Build Configure
1. settings like JVM memory settings, Java home, daemon on/off can be more useful if they can be versioned with the project in your VCS so that the entire team can work with a consistent environment. 
2. ways of configuring:
	1. local environment variables: GRADLE_OPTS or JAVA_OPTS
	2. gradle.properties **(BETTER ONE)**
2. The configuration is applied in following order (if an option is configured in multiple locations the last one wins):
	1. from gradle.properties in project build dir.
	2. from gradle.properties in gradle user home.
	3. from system properties, e.g. when -Dsome.property is set on the command line.