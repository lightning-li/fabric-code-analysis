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
------------------------------
#### 设置默认值
例子：

```
viper.setDefault("ContentDir", "content")
viper.setDefault("Student", map[string]string{"name" : "lk", "age" : 12})
```

#### 读取配置文件

当前，一个 Viper 的实例只支持一个配置文件，Viper 并没有默认配置文件的搜索路径，所以应用程序需要告诉 Viper 实例配置文件的路径，这里有一个如何使用 Viper 搜索并读取配置文件的例子：
```
viper.SetConfigName("config")          //配置文件的名称(不需要扩展名)
viper.AddConfigPath(".")              //将当前目录加入到 viper 的搜索路径中
viper.AddConfigPath("$HOME/somedir")  //可多次调用AddConfigPath函数，注意：若多个搜索路径中均包含指定的配置文件，默认读取的是一个添加的搜索路径里的配置文件
err := viper.ReadInConfig()          //搜索并且读取配置文件
if err != nil {
    panic(fmt.Errorf("Fatal error config file : %s \n", err))
}
fmt.Println(viper.Get("name"))      //假设配置文件中包含name属性，那么 Get 可以得到该属性对应的值
```

#### 观察并且重新读取配置文件(Watching and re-reading config files)

viper 支持你的应用程序运行时观察到并读取配置文件的改变，重启服务才能让你的配置文件生效已经成为过去，viper 可以很轻松的帮助你读取到更新过后的文件，告诉 viper 实例观察配置文件(WatchConfig)，你还可以传进去一个函数来告诉 viper 实例当配置文件改变时应该做什么，例如
```
viper.WatchConfig()
viper.OnConfigChange(fun (e fsnotify.Event) {
    fmt.Println("config files changed, afer change, the name become : ", viper.Get("name"))
})
```

#### Reading Config from io.Reader

viper 预定义了很多配置源，例如文件、环境变量、flags 和远程 key/value 存储，但是你不会被局限于此，你可以定义自己的配置文件源，并将其送给 viper， 例如
```
viper.SetConfigType("yaml")
var yaml_ex = []byte(`
name : kk
age : 23
wife :
    name : ss           //注意：yaml文件只支持前面是空格，不支持tab键，所以确保属性的前面是空格而不是tab键
    age : 23
subject :
    - math
    - computer
`)
viper.ReadConfig(bytes.NewBuffer(yaml_ex))
fmt.Println("name ", viper.Get("name"))
```

#### 注册并使用别名(Registering and Using Aliases)
Aliases允许一个值多个键引用，例如
```
viper.RegisterAlias("Loud", "Verbose")
viper.Set("loud", "haha")                      //viper 不区分大小写
fmt.Println("Loud ", viper.Get("Loud"))        //输出：Loud haha
fmt.Println("verbose ", viper.Get("verbose"))  //输出：verbose haha
```

#### viper 使用环境变量
viper 使用以下四种方法与环境变量打交道：
- `AutomaticEnv()`
- `BindEnv(string...) : error`
- `SetEnvPrefix(string)`
- `SetEnvKeyReplacer(*strings.Replacer)`

需要认识到的是，viper 与环境变量打交道时，是大小写敏感的。

viper 提供了一种机制来保证环境变量是唯一的，通过使用`SetEnvPrefix(string)`告诉viper 实例读取环境变量的时候自动加上前缀，格式形为：`prefix + '_' + env`；`AutomaticEnv` 和 `BindEnv(string...)` 都会使用该前缀。

`BindEnv` 函数需要一个或两个参数，第一个参数为 key，第二个参数为环境变量的名称，当只有一个参数的时候，viper 会自动去环境变量中匹配第一个 key 参数(注意，此时 key 会被 viper 实例自动转换为大写)；当有两个参数的时候，viper 会将该 key 与指定的环境变量名称绑定。注意：如果显示指定了两个参数，则 `BindEnv` 不会自动加上前缀。

需要认识到的是，每一次 viper 实例访问环境变量的时候，都会重新读取，`BindEnv` 并不会将 key 与值相固定

`AutomaticEnv` 与 `SetEnvPrefix` 结合的时候相当强大，`AutomaticEnv` 被调用后，每一次调用`viper.Get`，viper 将会检查环境变量。viper 实例会自动将前缀加在 key (自动被转换为大写字母)上，如果 `SetEnvPrefix` 被调用的话。

`SetEnvKeyReplacer` 允许你使用 `strings.Replacer` 对象在某种程度上重写环境变量，比如你想在 `viper.Get` 调用中使用 `-` 或者别的间隔符，而在环境变量中使用 `_` 间隔符的时候，`SetEnvKeyReplacer` 是很有用的。 