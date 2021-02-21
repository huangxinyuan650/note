## GO真的是门不错的语言，简洁
##### 基础命令
- go build：go build [*.go] [-o exe.path]，测试编译，若不指定待编译文件则编译当前目录下所有go文件，同时可指定输出编译后的exe文件
- go clean：移除当前源码包中编译生成的文件
- go fmt：格式化代码文件（go fmt *.go｜ go fmt -w src）
- go get：动态获取远程代码包（下载源码然后执行go install）
- go install：安装源码包（生成结果文件（可执行文件或者.a文件），然后将文件加到GOPATH下的bin或者pkg文件夹下）
- go test：自动读取源文件下的*_test.go文件，然后生成并运行测试用的可执行文件
- go doc：查看package响应的文档

##### Go程序启动过程
- 1、起始于main包，导入main包中的其他包
- 2、main包中导入其他包时会同时将包级常量和变量初始化，以此类推（顺序为堆栈）
- 3、其他包导入完成后会初始化main包中的包级常量和变量
- 4、执行init方法
- 5、执行main方法

#### Tips
- interface{}作为函数参数则函数可接受任意类型参数，做为返回值则函数可返回任意类型的值
