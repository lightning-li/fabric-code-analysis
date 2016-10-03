### peer 服务的工作流程

**1. 启动（CLI）**

peer 服务可以通过命令行的方式来启动，比如，`peer node start`就可以启动一个peer服务，相应的配置可以从peer/core.yaml中读取。接下来分析一下代码中使用的 `cobra` 库和 `viper` 库(均出自spf13大神之手)。