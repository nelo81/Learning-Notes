## java9 jigsaw module

#### 基本概念

1. java9 引入了模块系统，同时自身的 jdk 也模块化了，其 modules 在 jmods 文件夹中，属于 module-path

2. unnamed modules:
  - 在 classpath 的所有jar (不管是否模块化)共同组成一个 unnamed module
  - unnamed modules 会声明依赖所有的 named module，且 exports 自己的所有包
  - 但是一个 named module 不能声明依赖 unnamed module
  - 如果一个package 在 named 和 unnamed 模块中都有定义，则使用 named 中的 package

3. named module: 
  - 有 module-info.java 的模块，是 java9 正规的 module
  - 没有 module-info.java 的 jar 包，如果放在 module-path 下，java9 模块系统自动将其变成 automatic module (其中如果 jar 包中 MANIFEST.MF 文件有 Automatic-Module-Name 属性，取其值作为模块名，没有的话，将 jar 包文件名根据一定规则提取模块名，如果提取不成功则无法转变为 automatic module)

4. automatic module: 
  - 默认 exports/open 自己所有的 package
  - transitive 依赖 jdk，其他 automatic module 及自己的 module 等其他所有存在的模块
  - 可以访问 classpath 那些 unnamed module 的类
  - jlink 不支持链接 automatic modules

5. main module: 包含 main 方法的 module，通过 `--module` 指定

6. root module: 模块系统解析的根模块，从根模块解析(在编译时期，而不是运行时)模块依赖，可以通过 `--add-modules mod1,mod2` 来将除 JDK 默认 root modules 外的模块添加到模块依赖解析中,可以通过扫描模块描述符把相关依赖的模块也同时解析了。

7. layers: 
  - layer 之于 module，就相当于 classloader 之于 class，layer 是 module 的一个加载和实例化的机制。通常用来在运行时动态加载 modules
  - boot layer 是 java 模块系统首先使用的 layer。它包括了 bootstrap loader，platform loader，application loader。java runtime 会根据 `--add-modules` 指定的 root modules 来构建模块依赖图，这一层就是 boot layer。

#### 模块访问限制

- 直接调用访问: 在 java9 之前，只要是 public 的方法，类等，外部类都能导入和调用，但是模块化之后 named module 中只有 module-info.java 中 export 的包中的类和方法并且是 public 才能被直接调用，否则编译时会出错。

- 反射调用访问: named module 中只有 module-info.java 中 open 的包中的类和方法才能被外部类反射调用。

- unnamed module 默认和 java8 以前的访问权限一样，包括反射，不过可以设置 `--illegal-access=deny` (默认是 permit) 来禁止对 unnamed module 的所有反射调用。

- automatic module 所有调用（包括反射）和 java8 以前的访问权限一样