# Go编译时参数

Go编译时，可以通过传入参数来定制编译过程的特征

具体参数可以通过 **go help build** 来进行查看

下面挑选一些常用的参数列举：

## 标准参数

| 参数 | 说明 |
| --- | --- |
| -o | 指定构建完成输出路径和文件名 |
| -i | 已废弃，安装指定的包来完成编译，编译需要的包会自动缓存下来 |
|  | 其他参数都被归类为build flags |

## build flags

build flags在**build,clean,get,install,list,run,test**等命令下都是通用的

下面记录一些常用的参数

名称                     | 描述                          |
| ---------------------- | --------------------------------------- |
| -tags ‘tag list’       | （常用） 构建出带tag的版本.  |
| -gcflags ‘arg list’    | （常用）编译参数go tool compile --help查看所有可用的参数 .    |
| -ldflags ‘flag list’   | （常用）链接参数go tool link --help查看可用可用的参数   |
| -mod                   |  （常用）readonly,vendor,mod 1.14版本以后，如果在mod文件里面有指定vendor，则默认使用vendor，否者设置为readonly|
| -race                  | （重要）同时检测数据竞争状态，只支持 linux/amd64, freebsd/amd64, darwin/amd64 和 windows/amd64. |
| -trimpath             | （重要）删除编译包含的固定路径信息，如 -trimpath=$GOPATH，报错信息打印时只会包含文件的相对路径|
| -modfile             |  指定使用的modfile文件，但go.mod文件依然是需要的，用于确认编译包的根目录，gosum依然是需要的，如传入的xx.mod, 则需要或对应生成为 xx.sum|
| -n                     | 仅打印输出build需要的命令，不执行build动作（少用）。      |
| -p n                   | 开多少核cpu来并行编译，默认为本机CPU核数（少用）。      |
| -v                     | 打印出被编译的包名（少用）.            |
| -work                  | 打印临时工作目录的名称，并在退出时不删除它（少用）。          |
| -x                     | 同时打印输出执行的命令名（-n）（少用）.               |

## gcflag

编译参数，我们通过**go tool compile -help**包含哪些

下面列举一些常用的参数

名称                     | 描述                          |
| ---------------------- | --------------------------------------- |
| -m      | （常用）打印优化信息 |
| -N      | 禁用优化 (debug时用到） |
| -l      | 禁止内联优化 (debug时用到） |
| -c      | 指定编译时的的并发数，默认为1 |
| -L      | 错误信息中打印文件全名  |

gcflag传入的方式为： -gcflag="pattern= args",其中pattern代表取值分别为 main,all,std,...,用于指定编译参数作用的范围，args则为对应的编译参数

### gcflag 中的 pattern

pattern 是选择包的模式，它可以有以下几种定义:

- `main`: 表示 main 函数所在的顶级包路径

- `all`: 表示 GOPATH 中的所有包。如果在 modules 模式下，则表示主模块和它所有的依赖，包括 test 文件的依赖

- `std`: 表示 Go 标准库中的所有包

- `...`: `...` 是一个通配符，可以匹配任意字符串(包括空字符串)。例如:

    - 例如: `net/...` 表示 net 模块和它的所有子模块

    - `./...` 表示当前主模块和所有子模块

    - **注意:**  如果 pattern 中包含了 `/` 和 `...`，那么就不会匹配 `vendor` 目录

        - 例如: `./...` 不会匹配 `./vendor` 目录。可以使用 `./vendor/...` 匹配 vendor 目录和它的子模块

举例如下：

```
go build -gcflags="main=-N -l" .
```

## ldflag

链接参数,我们通过**go tool link --help** 查看可用的参数

下面列举一些常用的参数

名称                     | 描述                          |
| ---------------------- | --------------------------------------- |
| -X     | 注入变量，通常用于版本信息注入 |

举例如下

```
go run -ldflags="-X main.who handsomeboy" main.go
```





