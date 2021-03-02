项目管理工具



# build.gradle

## 生命周期

在构建初始化阶段，Gradle根据`build.gradle`组装工程的大致流程如下：

- 1、创建一个`Settings`实例，如果Gradle工程下的`settings.gradle`文件存在，则该文件将被解析成一个`Settings`实例
- 2、根据`settings.gradle`文件对工程进行配置
- 3、使用`Settings`实例创建`Project`实例的结构
- 4、最后，通过对项目执行build.gradle文件（如果存在）来评估每个项目

## 内部属性

在`build.gradle`文件中采用`DSL`进行配置，下面是你可以在`build.gradle`文件中配置的内容：

- `Task`

  `Task`是一种任务，任务是Gradle中的基本组件，`Task`的概念也是收到了`Ant`的启发而设计的

- `Dependencies`

  依赖管理

- `Plugins`

  插件，就像我们在Maven的`pom.xml`文件中增加插件一样，Gradle也可以添加插件，常见的插件有`java`，`eclipse`,我们可以使用`apply plugin: 'java'`加载插件

- `Properties`

  属性配置

- `Methods`

  配置一个方法，纳尼，配置文件可以配置一个方法，你特么在逗我！没错，这就是`DSL`的魅力，`DSL`内置支`Groovy`这种脚本语言，因此，我们可以在`build.gradle`文件中编写相关符合`groovy`语法的代码，因此，换句话说，`build.gradle`文件已经超出了配置文件的范畴，看起来更像是一个脚本文件，比配置文件更加强大。

- `Script Blocks`

  脚本块是个很强大的特性，也是Gradle特有的。上面说到其实`build.gradle`文件相当于一个脚本文件，因此脚本块的出现也就不足为奇了。常用的脚本块有以下几种：

  - `allprojects`

    在多项目工程中对所有项目（包括子项目）的通用配置

  - `ant`

    对ant提供支持

  - `artifacts`

    对生成构件的支持，如生产一个jar包

  - `buildscript`

    执行`gradle build`命令的相关配置，主要配置构建的输出目录，引用的仓库等信息

  - `configurations`

    提供对依赖管理的支持

  - `dependencies`

    依赖的声明和配置

  - `repositories`

    仓库配置

  - `subprojects`

    在多项目中对所有子项目的通用配置