# index.html
## 退出登录
>点击退出按钮，界面跳转到login界面，token的value初始化，不可通过url跳转到index界面，调用了/login/接口
``` go 
//url = operations/login/*interface
//OperationsLogin
else if key2 == "" {
    if true {
        t_key2 = "login.html"
    }
    c.HTML(http.StatusOK, t_key2, nil)
}
```
