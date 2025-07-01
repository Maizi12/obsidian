Welcome back to beego's hellhole where things isn't as easy as using Gin. Where a single line of code doesn't work and there's no tutorial or documentation about it. Where you are so cooked you start to digging the code yourself and give back to community. Who needs tutorial when you can make 1 right? Let's start

In Golang there's a built-in function ``` recover() ``` which is set to recover from ```panic()```.
So it should be easy right? Only need do thing like this: ``` defer func() { if err := recover(); err != nil { } }```
defer func is there to ensure the recover() is executed. Well then easy peasy right? We only need to do something like this ``` //beego.InsertFilter("/*", beego.AfterExec, middleware.RecoverMiddleware) ``` and ohh-- it didn't work. 
Why wouldn't it? Just when I think it would only take 5 minutes. Well then time to do some digging again. 
Let's try ask chatgpt first, it says the problem is ```defer recover()``` only recovers panics happen during the filter. So perhaps we need to wrap the entire beego HTTP Handler. Is there no easier way? 
Oh right, beego has a default RecoverFunc. I just have to replace it with my own. 
### Default Beego Panic Handler
```go
func init() {
    BConfig = newBConfig()
}

func newBConfig() *Config {
    res := &Config{
        // ----------------------------
    }
    res.RecoverFunc = defaultRecoverPanic
    return res
}

func defaultRecoverPanic(ctx *context.Context, cfg *Config) {
    if err := recover(); err != nil {
    // -------------------
    }
}
```
Alright since we already found it, let's set the middleware to fit in

```go
// router.go
beego.BConfig.RecoverFunc = middleware.RecoverMiddleware

//middleware.go
func RecoverMiddleware(ctx *context.Context, cfg *beego.Config) {  
    var funcname string  
    defer func() {  
       if err := recover(); err != nil {
       fmt.Println("recover middleware")
       }
    }
}
```
Well let's test it again, oh damn it why it doesn't work? Time to dig again. Let's find where the ```
RecoverFunc``` is called. Oh it was called in this:

```go
func (p *ControllerRegister) serveHttp(ctx *beecontext.Context) {
// ----------------
if p.cfg.RecoverFunc != nil {  
    defer p.cfg.RecoverFunc(ctx, p.cfg)  
}
//-----------
```
So the RecoverFunc does called, but why doesn't mine executed? Could it because double defer? Inside ```serveHttp```, ```RecoverFunc``` already called using defer, and inside the ```RecoverMiddleware
``` was executed using ```defer``` too. Let's try change it to:

```go
/// defer block deleted
if err := recover(); err != nil {
}
```
Aight time to test it again, and yes it works! Another feature completed.