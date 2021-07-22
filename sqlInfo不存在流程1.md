# **Kylin-Desktop 10.1**
## **main.go**
``` go
//传递函数
core.SetFuncValidationConformIP(validation.ConformIP)
```
引用core包下的SetFuncValidationConformIP函数
``` go
//将全局变量validationConformIP赋给v，也就是validation.ConformIP
func SetFuncValidationConformIP(v validationInterface){
	validationConformIP = v
}
```
将validation.ConformIP传给全局变量core包下的validationConformIP
```go
//过滤IP
type validationInterface func(string) (bool) 
```
validationConformIP的类型是validationInterface函数，返回值是bool类型，需要传入一个string类型的参数
```go
//判断运维端身份
func ConformIP(ipstr string) bool {
	for i := 0; i < len(People_operations); i++ {
		if ipstr == People_operations[i] {
			return true
		}
	}
	return false
}
```
将validation包下的ConformIP函数赋值给core包下的全局变量validationConformIP
***
```go
//初始化服务器
core.ServerInit()
```
引用core包下的ServerInit函数
``` go
func ServerInit(){
	Router.Use(cors())
	Router.Use(Is_connect_interrupt)
	Router.Use(JwtVerify) 
	htmlpath := InstallPath + "html"
	if pathExists("./html"){
		Router.LoadHTMLGlob("./html/*/*")
		Router.StaticFile("/favicon.ico", "./favicon.ico")
		Router.StaticFS("/operations/public/layui", http.Dir("./layui"))
	}else if pathExists(htmlpath){
		Router.LoadHTMLGlob(htmlpath + "/*/*")
		Router.StaticFile("/favicon.ico", InstallPath+"favicon.ico")
		Router.StaticFS("/operations/public/layui", http.Dir(InstallPath+"layui"))
	}else{
		fmt.Println("加载资源失败")
	}
}
```
1. 该函数主要作用是首先判断目录下的静态资源是否存在，如果存在则进行静态资源加载操作，如果不存在则打印"加载资源失败"  
2. Router.Use(Is_connect_interrupt)  
Router.Use(JwtVerify)  
引用了core包下定义的两个函数：Is_connect_interrupt，JwtVerify
```go
func Is_connect_interrupt(c *gin.Context){
	if Connect_interrupt==-1{
		
		return
	}
	str := strconv.Itoa(Connect_interrupt)+"秒后重连..."
	if Connect_interrupt==0{
		str = "正在重连..."
	}
	c.String(202,MMESSAGE0+"已尝试重连"+strconv.FormatUint(ReconnectMysqlNum,10)+"次，"+str)
	c.Abort()
	return 
}
```
Is_connect_interrupt函数是服务器初始化时，输出数据库连接状态
```go
//验证token
func JwtVerify(c *gin.Context) {
	if validationConformIP(c.ClientIP()) {
		return
	}
	if isContainArr(noVerify, c.Request.RequestURI) {
		return
	}
	token := c.GetHeader("AccessToken")
	if !parseToken(token) {
		fmt.Println(token)
		c.AbortWithStatus(401)
	}
}
```
***
``` go
success := client.GetDefultTbn()
```
获取ssmap表下serial-number为88888888的记录，如果记录存在则返回true，否则返回false
``` go
if !success { 
		fmt.Println("-------------------------------------")
		fmt.Println("连接数据库失败，可能是网络问题，请稍后重试")
		fmt.Println(core.OperationFile(success))
		fmt.Println("-------------------------------------")
		core.CloseSql()
		return
	}
```
如果success为false，即表中没有对应记录，则调用core.OperationFile(success)
``` go
func OperationFile(success bool) string {
	//homePath  = "/etc"
	//filename  = "kylin-sourceinfo-server/data/sqlInfo"
	//filenameEncryption string = "encryption"
	os.Remove(homePath + filename)
	os.Remove(homePath + filenameEncryption)
	content := t_ipAddress + "=" + sqlInfo.ipAddress + "\n"
	content += t_dbName + "=" + sqlInfo.DbName + "\n"
	content += t_userName + "=" + sqlInfo.UserName + "\n"
	content += t_password + "=" + sqlInfo.Password + "\n"
	content += t_port + "=" + sqlInfo.Port + "\n"
	content += t_charset + "=" + sqlInfo.Charset + "\n"
	content += t_tokenAging + "=" + sqlInfo.TokenAging
	createFileWithDir(homePath, filename, content)
	return "也可能是数据库地址信息错误，\n确认 " + homePath + filename + " 中信息是否正确"
}
```
创建sqlInfo文件，将连接数据库信息写入sqlInfo文件中
``` go
func createFileWithDir(path string, name string, content string) {
	os.MkdirAll(path, os.ModePerm)
	file, _ := os.OpenFile(path+name, os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0777)

	defer file.Close()
	file.WriteString(content)
}
```
这一步走完直接return，再次重启服务连接数据库，**注意homePath和fileName的设置**
