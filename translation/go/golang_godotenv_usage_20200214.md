# GoDotEnv 使用手册

GoDotEnv是Go（golang）版的Ruby dotenv 项目，它从`.env`文件中加载变量。

Ruby dotenv如此描述：
> 在环境中存储配置是12-factor app[^1]的原则之一，不管怎么样，对部署的环境修改，比如数据库资源或者外部服务的证书，需要从代码中抽取出来，放入环境变量中。但不应在开发环境设置环境变量机器或持续集成服务机器运行多个项目上进行修改。`DotEnv`在项目启动的时候将`.env`文件中的变量加载到`ENV`中。

`GoDotEnv`可以作为库（后台进程导入env）或者 `bin`命令。

## 如何安装

1. 作为库 : `go get github.com/joho/godotenv`
2. 作为命令：`go get github.com/joho/godotenv/cmd/godotenv`

## 使用手册

### 直接使用

在项目的**根目录**创建`.env`，里面放入项目中要用到的配置变量。

```shell
S3_BUCKET=YOURS3BUCKET
SECRET_KEY=YOURSECRETKEYGOESHERE
```

测试代码如下：

```golang
package main

import (
	"log"
	"os"

	"github.com/joho/godotenv"
)

func main() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}

	s3Bucket := os.Getenv("S3_BUCKET")
	secretKey := os.Getenv("SECRET_KEY")

	log.Println("s3Bucket:" + s3Bucket + " secretKey:" + secretKey)
}
```

### 偷懒方式

按照如下方式，会自动将`.env`导入

`import _ "github.com/joho/godotenv/autoload"`

还有下面两种方式也是合法的：

```golang
_ = godotenv.Load("somerandomfile")
_ = godotenv.Load("filenumberone.env", "filenumbertwo.env")

```

### `.env`文件内容格式

1. `key=value`格式

```shell
# I am a comment and that is OK
SOME_VAR=someval
FOO=BAR # comments at line end are OK too
export BAR=BAZ
```

2.  YAML格式

```yaml
FOO: bar
BAR: baz
```

### 读取变量

1. `os.Getenv(key)`
2. 读入map

```golang
var myEnv map[string]string
myEnv, err := godotenv.Read()

s3Bucket := myEnv["S3_BUCKET"]
```

或者 `io.Reader`

```golang
reader := getRemoteFile()
myEnv, err := godotenv.Parse(reader)
```

或者来源于一个 `string`

```golang
content := getRemoteFileContent()
myEnv, err := godotenv.Unmarshal(content)
```

## 优先级和习惯用法

已经存在环境变量的优先级高于后加载的，也就是说越早加载的越高。可以利用此规则来
管理多个环境（比如，开发、测试、生产）。创建一个名为 `{YOURAPP}_ENV`的环境变量
，加载的顺序如下：

```golang
env := os.Getenv("FOO_ENV")
if "" == env {
    env = "developement"
}
godotenv.Load(".env." + env + ".local")
if "test" != env {
    godotenv.Load(".env.local")
}

godotenv.Load(".env." + env)
godotenv.Load()  // The Original .env

```

但可以通过`godotenv.Overload()`改变这种优先级。


## 命令模式

需要`$PATH`配置 `$GOPATH/bin `, `-f` 指定`.env`的文件路径，缺省条件是 `$PWD/.env`

`godotenv -f /some/path/to/.env some_command with some args`

## 写 Env 文件

`Godotenv` 也能将存储在map环境变量写入文件中：

```golang
env, err := godotenv.Unmarshal("KEY=value")
err := godotenv.Write(env, "./.env")
```

或者写入字符串中

```golang
env, err := godotenv.Unmarshal("KEY=value")
content, err := godotenv.Marshal(env)
```

[^1]: https://12factor.net/