| 模块          | 用途     | 安装                                                     |
| ------------- | -------- | -------------------------------------------------------- |
| json-iterator | json解析 | go get github.com/json-iterator/go                       |
| zap           | 日志     | go get -u go.uber.org/zap                                |
| logrus        | 日志     | go get github.com/sirupsen/logrus                        |
| validator     | 参数校验 | github.com/go-playground/validator/v10                   |
| gorm          | orm      | go get -u gorm.io/gorm<br>go get -u gorm.io/driver/mysql |



# OS

```go
os.Getenv("GOPATH")				// 获取系统环境变量
os.Getwd()						// 获取当前路径
```

# Time

```go
t := time.Now()										// 输出当前时间
fmt.Println(t.Format("2006-01-02 15:04:05.000"))	// 格式化输出
```

### 美化json输出

```go
// PrettyJson
// 美化输出json
// @data json.Marshal()输出
func PrettyJson(data []byte) string {
	var out bytes.Buffer
	_ = json.Indent(&out, data, "", "  ")
	return out.String()
}

```



### 获取当前执行文件路径

只验证了main函数时的情况

```go
func getCurrentFilePath() string {
	// 0:		上溯的栈帧数, 0表示Caller的调用者(Caller所在的调用栈)
	// pc:		调用栈标识符
	// filePath:文件绝对路径
	// line:	该调用在文件中的行号
	// ok:		如果读取失败会返回false
	pc, filePath, line, ok := runtime.Caller(0)
	fmt.Println(pc)
	fmt.Println(filePath)
	fmt.Println(line)
	fmt.Println(ok)
	return filePath
}
```



### 获取当前执行文件目录路径

只验证了main函数时的情况

```go
func getCurrentDirPath() string {
	_, filePath, _, _ := runtime.Caller(0)
	return filepath.Dir(filePath)
}
```

