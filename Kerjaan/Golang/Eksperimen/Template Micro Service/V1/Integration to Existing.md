|                                           |      |           |         |       |        |          |       |        |
| ----------------------------------------- | ---- | --------- | ------- | ----- | ------ | -------- | ----- | ------ |
|                                           | user | transaksi | finance | proto | master | generate | crons | report |
| Baca Conf dijadiin variable               | ✅    | ✅         | ✅       | ✅     | ✅      |          |       | ✅      |
| Custom Logger GORM                        | ✅    | ✅         | ✅       | ✅     | ✅      |          |       | ✅      |
| Auto Rekonek DB                           | ✅    | ✅         | ✅       | ✅     | ✅      |          |       | ✅      |
| Query DB nunggu selesai rekonek DB        | ✅    | ✅         | ✅       | ✅     | ✅      |          |       | ✅      |
| Rekonek DB dihold sampai Query DB selesai | ✅    | ✅         | ✅       | ✅     | ✅      |          |       | ✅      |
| Response Template                         | ✅    | ✅         | ✅       | ❌     | ✅      |          |       | ✅      |
| Error Handler Middleware                  | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
| Panic Recover Middleware                  | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
| idUser taro di context                    | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
| Log Query                                 | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
| Log Activity                              | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
| Log Error Middleware                      | ✅    | ✅         | ✅       |       | ✅      |          |       | ✅      |
Jangan push/pull ke prod sebelum staging aman

|                     |      |           |         |       |        |          |       |        |
| ------------------- | ---- | --------- | ------- | ----- | ------ | -------- | ----- | ------ |
|                     | user | transaksi | finance | proto | master | generate | crons | report |
| Integration Updated | ✅    | ✅         | ✅       | ✅     | ✅      | ✅        |       | ✅      |