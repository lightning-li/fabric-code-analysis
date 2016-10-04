### peer 服务的工作流程

**1. 启动（CLI）**

peer 服务可以通过命令行的方式来启动，比如，`peer node start`就可以启动一个peer服务，相应的配置可以从peer/core.yaml中读取。接下来分析一下代码中使用的 `cobra` 库和 `viper` 库\(均出自spf13大神之手\)。

`cobra` 既是一个用来创建强大的现代化的 CLI 应用的库，也同时可以用来生成应用程序和命令文件。

`viper` 为 `Go` 应用程序提供了一个完整的配置解决方案。它被设计用来和应用程序一起工作，能够处理所有类型的配置，详细内容见 [viper](viper.md)。

`fabric peer`

