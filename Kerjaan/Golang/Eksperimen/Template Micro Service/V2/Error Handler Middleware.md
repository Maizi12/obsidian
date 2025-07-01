|                             |                       |
| --------------------------- | --------------------- |
| What                        | Why                   |
| Error Handler di middleware | Easier Standarization |
So for Error Handler, you can either sets a dedicated function, and applies it on each 
```if err!=nil ``` which is what I do before, terrible I just realize it now. 

So that got me thinking, "what if I can make it to a middleware and read if the response is error, will runs function to create log etc". Bright Idea, right? Well the reality isn't that easy. Because that means the middleware needs to run after controller or router.

For more details on how to set a middleware executed after controller or router, you can read the [[AfterExec And FinishRouter Filter Execution Point]]. 

```go
// router
func init(){
// -------
beego.InsertFilter("/*", beego.BeforeExec, middleware.ResponseInterceptor)
// -------
}

//middleware.go
type responseRecorder struct {  
    http.ResponseWriter  
    body   *bytes.Buffer  
    status int  
}  
  
func (r *responseRecorder) Write(b []byte) (int, error) {  
    r.body.Write(b)                  // Capture response body  
    return r.ResponseWriter.Write(b) // Still write to client  
}  
  
func (r *responseRecorder) WriteHeader(statusCode int) {  
    r.status = statusCode  
    r.ResponseWriter.WriteHeader(statusCode)  
}  
  
func ResponseInterceptor(ctx *context.Context) {  
    fmt.Println("response interceptor")  
    // Wrap the actual writer  
    buf := &bytes.Buffer{}  
    rec := &responseRecorder{  
       ResponseWriter: ctx.ResponseWriter.ResponseWriter,  
       body:           buf,  
       status:         200, // default  
    }  
  
    // Replace Beego's internal writer with our wrapper  
    ctx.ResponseWriter.ResponseWriter = rec  
  
    // After controller logic runs, body has been written  
    ctx.Input.SetData("respRecorder", rec) // optional: for later access  
}  
  
func LogInterceptor(ctx *context.Context) {  
    if val := ctx.Input.GetData("respRecorder"); val != nil {  
       if rec, ok := val.(*responseRecorder); ok {  
          if rec.status >= 400 {  
             pkg.CreateLog(0, pkg.BeegoBodyLogMiddleware(ctx), pkg.BeegoBodyLogStatusCode(ctx), *ctx, rec.body.String(), strconv.Itoa(rec.status))  
          }
       }  
    }
}
```

With that out of the way, let's test it. I was testing it with concurrent rest api request. With the function inside repositories deliberately sets to return error.

command : ```ab -n 1000 -c 100 "http://localhost:4032/v1/user/coa" ```

The results magnificent, for 1000 request and 100 concurrent requests, we get the result of total 2.396 seconds. With one longest request takes 708ms and the mean of requests per second 417 and the mean time taken per request is 239ms.

