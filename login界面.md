# login.html
## 流程
1. 输入用户名和密码
2. xml异步请求接口 /login?username=&password=
3. 登录成功，获得接口返回值，设置token，页面跳转
4. 登录失败，alert错误提示信息，也是由接口返回的
``` go
func OperationsLogin(f operationsLoginInterface) {
	httpUrl := "/operations/login/*interface"
	Router.GET(httpUrl, func(c *gin.Context) {
		t_key2 := c.Param("interface")
		t_key2 = strings.Trim(t_key2, "/")
		name := c.Query("name")         
		password := c.Query("password") 
		if t_key2 == "login" {
			result := f(name, password, "login")
			if result == VMESSAGE2 {
				var user UserClaims
				user.Name = name
				user.Password = password
				str := GenerateToken(&user)
				c.String(http.StatusOK, str)
				return
			}
			c.String(401, result)
```
``` go
if cmd == "login" {
		if name == "" || password == "" {
            //Incorrect user name or password
			return core.VMESSAGE1
		}
		sqlCmd := selectSaltSql(name)
		salt := core.CmdSqlString(sqlCmd)
		if salt == "" {
			return core.VMESSAGE1
		}
        //密码加密
		m5 := md5.New()
		m5.Write([]byte(password))
		m5.Write([]byte(string(salt)))
		code := m5.Sum(nil)
		password = hex.EncodeToString(code)
        //查询user表是否由匹配的记录
		sqlCmd = selectPasswordSql(name)
		result := core.CmdSqlString(sqlCmd)
		if result != password {
			return core.VMESSAGE1
		}
        //Login successful
		return core.VMESSAGE2
```