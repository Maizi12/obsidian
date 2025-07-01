Like the Error Handler, I also create a function to handles panic across repositories. Which was also a terrible decision considering we runs microservices means I would need to add it on each repositories. So that also got me thinking. I would need to make a Panic Handler Middleware. 
Read [[[Panic Handler]]] for more technical details. 
Let's talk about the implementation, so the Panic Handler needs to provide enough context like which function the panic happens, which line. How to do it? Well alhamdulillah we got runtime library. The PanicHandler didn't actually need to provide error to the response, just give 400 or 500, we only need to stop it from crashing when panic happens.
```go
// router.go
func init(){
// ----
beego.BConfig.RecoverFunc = middleware.RecoverMiddleware
// ----
}
//middleware.go
func RecoverMiddleware(ctx *context.Context, cfg *beego.Config) {  
    var funcname string  
    fmt.Println("recover middleware")  
    if err := recover(); err != nil {  
       for i := 1; i < 5; i++ {  
          pc, _, _, ok := runtime.Caller(i)  
          if ok {  
             funcname = runtime.FuncForPC(pc).Name()  
             if strings.Contains(funcname, conf.Konfig.Server.AppName) {  
                _, line := runtime.FuncForPC(pc).FileLine(pc)  
                funcname += " Line " + strconv.Itoa(line)  
                break  
             }  
          }  
       }  
       // You can also access ctx.Input or ctx.Output here if needed  
       fmt.Printf("[PANIC] UserID: %v | Error: %v | In: %v\n", ctx.Input.GetData("user_id"), err, funcname)  
       // You could write error response or log here too
       pkg.CreateLog(0, pkg.BeegoBodyLogMiddleware(ctx), pkg.BeegoBodyLogStatusCode(ctx), *ctx, funcname, "500")
       ctx.Output.SetStatus(500)  
       ctx.Output.Body([]byte("Internal Server Error"))  
    }  
}
```
So here we scans the runtime, why 1 to 5? Because that the sufficient range, if we use like beego's default panic handler, it returns all the error stack which we don't actually need.

So the test case is similar to error handler, purposely creates a panic inside the repository which we didn't set to catch panic in there but in the router. And it works! the response is Internal Server Error as intended. Aight see you next note.