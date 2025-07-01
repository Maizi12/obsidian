
| What                        | Why                                                                                                                                | Test Case                                                         |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Baca Conf dijadiin variable | Kalau nggak, bisa jadi nanti ketika udah jadi binary ketika file conf nya ilang jadi error. Dan hemat resources ga buka file terus | Compare ketika running antara old dan new dengan delete conf file |
Code
```go
func main() {
var wg sync.WaitGroup  
wg.Add(2)  
conf.ReadEnv()  
go func() {  
for i := 0; i < 10000; i++ {  
dbUser := conf.ReadEnvOld().DB.User  
fmt.Println("dbUser Old ENV", dbUser)  
}  
defer wg.Done()  
}()  
go func() {  
for j := 0; j < 10000; j++ {  
dbUser := conf.Konfig.DB.User  
fmt.Println("dbUser New ENV", dbUser)  
}  
defer wg.Done()  
}()  
wg.Wait()  
} 
// db module
var Konfig Env  

type EnvServer struct {  
AppName string  
HttpPort string  
RunMode string  
AutoRender bool  
CopyRequestBody bool  
EnableDocs bool  
GRPCPort string  
}  

type EnvDB struct {  
Driver string  
User string  
Password string  
Host string  
Name string  
MaxLifetime int  
MaxOpenConns int  
MaxIdleConns int  
}  
  
type Env struct {  
Server EnvServer  
DB EnvDB  
}  
  
func ReadEnv() *Env {  
var result Env  
conf, err := config.NewConfig("ini", "conf/app.conf")  
if err != nil {  
fmt.Println("Error reading config file:", err)  
return nil  
}  
result.DB.Driver, _ = conf.String("db.driver")  
result.DB.User, _ = conf.String("db.user")  
result.DB.Password, _ = conf.String("db.password")  
result.DB.Host, _ = conf.String("db.url")  
result.DB.Name, _ = conf.String("db.name")  
  
result.DB.MaxLifetime, _ = conf.Int("db.maxLifetime")  
result.DB.MaxOpenConns, _ = conf.Int("db.maxOpenConns")  
result.DB.MaxIdleConns, _ = conf.Int("db.maxIdleConns")  
  
result.Server.AppName, _ = conf.String("appname")  
result.Server.HttpPort, _ = conf.String("httpport")  
result.Server.GRPCPort, _ = conf.String("GRPCPort")  
result.Server.RunMode, _ = conf.String("runmode")  
Konfig = result  
return &result  
}  
ReadEnvOld sama tapi gaada Konfig=result
```

Result: Panic Error dari ReadEnvOld ketika file dihapus