# Why
### Sometimes there's a need to have a middleware or filter that runs after controller or router.
## Case 1
I have a need to read the response from controller to handle error. So an [[Error Handler Middleware]] is required. Then what's the problem? Just read the documentation right? Well not so fast brother. It requires me a hour to search google, scouring the documentation, asking chatgpt with zero working solutions or even a goddamn clue. They all pinpoit to beego.AfterExec or beego.FinishRouter in the InsertFilter config and it doesn't work. So here's my digging:

```go
// so the framework will runs this
func (p *ControllerRegister) execFilter(context *beecontext.Context, urlPath string, pos int) (started bool) {
var preFilterParams map[string]string 
for _, filterR := range p.filters[pos] { 
	b, done := filterR.filter(context, urlPath, preFilterParams) 
	if done { 
		return b 
		} 
	} 
	return false 
}

func (f *FilterRouter) filter(ctx *context.Context, urlPath string, preFilterParams map[string]string) (bool, bool) {
    //we need this conditions to be not reached, means returnOnOutput and ResponseWriter.Started both is false
    if f.returnOnOutput && ctx.ResponseWriter.Started {
        return true, true
    }

    if f.resetParams {
        preFilterParams = ctx.Input.Params()
    }

    if ok := f.ValidRouter(urlPath, ctx); ok {
        //we need to reach this for the middleware to run
        f.filterFunc(ctx)
        if f.resetParams {
            ctx.Input.ResetParams()
            for k, v := range preFilterParams {
                ctx.Input.SetParam(k, v)
            }
        }

    } else if f.next != nil {
        return f.next.filter(ctx, urlPath, preFilterParams)
    }

    if f.returnOnOutput && ctx.ResponseWriter.Started {
        return true, true
    }
    return false, false

}
// Now since it was after controller logic, and I want to catch the response already written, the Started is already set as true because of this

// Write writes the data to the connection as part of a HTTP reply,

// and sets Started to true.

// Started:  if true, the response was already sent

func (r *Response) Write(p []byte) (int, error) {
    r.Started = true
    return r.ResponseWriter.Write(p)
}

// The only way I found is this:

beego.InsertFilter("/*", beego.FinishRouter, middleware.LogInterceptor, beego.WithReturnOnOutput(false))

type filterOpts struct {
    returnOnOutput      bool
    resetParams         bool
    routerCaseSensitive bool
}

type FilterOpt func(opts *filterOpts)

func WithReturnOnOutput(ret bool) FilterOpt {
    return func(opts *filterOpts) {
        opts.returnOnOutput = ret
    }
}

// If I remember correctly, bool default value is true, so I only need to set the returnOnOutput as false.

// Yeah if only it was documented. Well then case closed, solution found

// But wait, default bool is false, so there's a code that sets the bool to true inside the framework. Yeah let's digging a bit more

// So we will runs the InsertFilter to set the middleware right


// InsertFilter Add a FilterFunc with pattern rule and action constant.
// params is for:
//  1. setting the returnOnOutput value (false allows multiple filters to execute)
//  2. determining whether or not params need to be reset.

func (p *ControllerRegister) InsertFilter(pattern string, pos int, filter FilterFunc, opts ...FilterOpt) error {
    opts = append(opts, WithCaseSensitive(p.cfg.RouterCaseSensitive))
    mr := newFilterRouter(pattern, filter, opts...)
    return p.insertFilterRouter(pos, mr)
}

// params is for:
//  1. setting the returnOnOutput value (false allows multiple filters to execute)
//  2. determining whether or not params need to be reset.

func newFilterRouter(pattern string, filter FilterFunc, opts ...FilterOpt) *FilterRouter {
    mr := &FilterRouter{
        tree:       NewTree(),
        pattern:    pattern,
        filterFunc: filter,
    }
    // lookie what we found here
    fos := &filterOpts{
        returnOnOutput: true,
    }

    // beego.WithReturnOnOutput(false) will be useful here
    for _, o := range opts {
        o(fos)
    }

    if !fos.routerCaseSensitive {
        mr.pattern = strings.ToLower(pattern)
    }

    mr.returnOnOutput = fos.returnOnOutput
    mr.resetParams = fos.resetParams
    mr.tree.AddRouter(pattern, true)
    return mr
}
```
So the conclusion is, the beego framework is a goddamn headache. For one line of code to run, you either can read the docs for 5 minutes then able to execute it, or taking like only hours to realize there's no answer provided and have to digs the source code directly.
It was worth it tho.