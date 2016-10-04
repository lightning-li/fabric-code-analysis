### viper

** viper 是什么？**

`viper` 为 `Go` 应用程序提供了一个完整的配置解决方案。它被设计用来和应用程序一起工作，能够处理所有类型的配置，它支持：

* 设置默认值
* 从 `JSON`、`TOML`、`YAML`、`HCL` 和 `Java` 特性配置文件中读取配置
* 可一直观察配置文件的改变并且重新读取配置文件
* 从环境变量中读取配置
* 从远程配置系统\(`etcd`、`Consul`\)中读取配置，并且一直观察配置的改变
* 从命令行的 `flags` 中读取配置
* 从 `buffer` 中读取配置
* 设置明确的值

`viper` 可以被认为是一个你的应用程序所需要的所有配置的注册中心。

** 为何使用viper？ **

当构建一个现代的应用程序的时候，你不想担心配置文件的格式、类型；你想要聚焦在构建令人惊叹的软件，`viper` 可以帮你处理配置的工作：

1. 寻找，载入，并且反序列化配置文件，该文件的格式可以是 JSON、TOML、YAML、HCL 或者 java properties 格式。
2. 提供一种机制，对于你不同的配置选项设置默认值。
3. 提供一种机制，通过命令行的 flags 覆盖配置选项的值。
4. 提供一种别名系统，让你很轻易的不用打破现有的代码来重命名参数。
5. 当使用者提供了命令行或者配置文件，与默认值相同的时候，很轻易的分辨出区别

`viper` 使用以下优先级，每一个条目的优先级都比它下面的条目的优先级高

- `explicit call to Set`
- `flag`
- `env`
- `config`
- `key/value store`
- `default`

### Putting Values into Viper
#### 设置默认值
例子：

```
viper.setDefault("ContentDir", "content")
viper.setDefault("Student", map[string]string{"name" : "lk", "age" : 12})
```