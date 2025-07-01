Title: Lack of documentation for intercepting responses with global middleware in Beego v2 (AfterExec & returnOnOutput)

Type: 🛠️ Optimization Request
Beego Version: v2.3.7

### Description
I was attempting to create a global middleware to intercept and log the response after it has been written, by using beego.InsertFilter with AfterExec. However, I noticed that neither AfterExec nor FinishRouter filters were triggering my middleware as expected when the response was already sent.

After tracing the call stack using a debug helper:

```go
beego.InsertFilter("/*", beego.BeforeExec, middleware.ResponseInterceptor)

func ResponseInterceptor(ctx *context.Context) {
	for i := 0; i < 10; i++ {
		pc, _, _, ok := runtime.Caller(i)
		if ok {
			fn := runtime.FuncForPC(pc)
			_, line := fn.FileLine(pc)
			fmt.Println(fn.Name() + " Line " + strconv.Itoa(line))
		}
	}
}
```

I found that the chain stops inside the FilterRouter.filter() function due to this early return:
```go
if f.returnOnOutput && ctx.ResponseWriter.Started {
	return true, true
}
```
### Workaround
By inspecting InsertFilter, I realized that this returnOnOutput flag is set to true by default. So I manually disabled it:
```go
beego.InsertFilter("/*", beego.AfterExec, middleware.LogInterceptor, beego.WithReturnOnOutput(false))
```

This change allows my `AfterExec` middleware to run even after a response has already started.

### Concern

While this works, it was **non-obvious** and **undocumented**. There’s no clear mention in the official Beego docs that:

- `returnOnOutput` must be set to `false` to execute filters after the response is started.
    
- `AfterExec` can be used for post-response logging with the correct configuration.

### Recommendation

1. Please **add documentation or examples** showing how to intercept response after it's written.
    
2. Consider **exposing a dedicated helper** or config flag (e.g., `InsertPostResponseFilter(...)`) to simplify this setup.
    
3. If possible, update the behavior so `AfterExec` runs by default even if response is started — or document the current design tradeoffs clearly.

| What                                        | Why                                                                                                                                | Version |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------- |
| [[RedEnv to variable]]                      | Kalau nggak, bisa jadi nanti ketika udah jadi binary ketika file conf nya ilang jadi error. Dan hemat resources ga buka file terus | V1      |
| Custom Logger GORM                          | Readable                                                                                                                           | V1      |
| [[Auto Rekonek DB And Wait Query Finished]] | Biar kalo server DB nya ke restart, gaperlu restart docker, dan ga tabrakan dengan query DB yang sedang berjalan                   | V1      |
| [[Query DB wait reconnect DB finished]]     | Tabrakan dengan Rekonek DB kalo nggak                                                                                              | V1      |
| [[Error Handler Middleware]]                | Customizable errror handler for response and logging                                                                               | V2      |
| [[Panic Handler]]                           | Stops from application crashing and easier to trace while also integrated with error handler for uniformed error responses         | V2      |
| Input Sanitation                            | For Date range field, to ensure the format of data inputted is correct                                                             | V2      |
| Activity Log                                | Helping to trace user's activity                                                                                                   | V3      |
| Query Log                                   | Also helping to trace user's activity when the needs arise                                                                         | V3      |