# GoCrawler

完成时间：1天
***
### 1. 学习Go的基础语法

   包括基本语法结构，基本数据类型，函数的调用，~~协程的使用~~等（前期实验用到，但是协程开多了网炸了，就没有再继续实验，取消了协程的使用）

### 2. 学习Go语言中html以及Goquery的使用

   通过调用Goquery中的相关函数实现网页内容的读取（以GET的方式），再使用Goquery的selector解析html代码，获取所需数据

   ```
   //获取Vtuber的属性，并用json传回
func get_vtuber(selection *goquery.Selection) []byte {
	vUrl, _ := selection.Attr("href")                        //Vtuber直播的地址
	vLiveName := selection.Find("h3").Text()                 //获取Vtuber直播间名字
	vVtuberName := selection.Find("span+span").Prev().Text() //获取Vtuber名字
	v := vtuber{url + vUrl, vLiveName, vVtuberName}          //构造Vtuber
	res, err := json.Marshal(v)                              //编码成json
	checkErr(err)                                            //错误判断
	return res                                               //返回json
}
   ```

   ```
   //从网页获取html信息，并返回所有的Vtuber，用json切片（二维byte类型切片）保存返回
func html_get(url string) [][]byte {
	doc, err := goquery.NewDocument(url)  //获取链接的html
	checkErr(err)                         //错误判断
	find := doc.Find(".room-card-item>a") //获取包含Vtuber信息的div中的a
	vtubers := [][]byte{}                 //建立二维byte类型切片用于保存Vtuber信息，内容为json格式
	//遍历当前页面中正在直播的Vtuber（kksk）
	find.Each(func(i int, selection *goquery.Selection) {
		vtubers = append(vtubers, get_vtuber(selection)) //调用get_vtuber函数返回Vtuber信息
	})
	return vtubers //返回Vtubers信息
}
   ```
### 3. json解析

   模拟数据传输，将获取的数据使用Golang自带的json编码，再使用其解码，模拟学习json的使用（相关代码参见4）
### 4. Go连接MySQL数据库

   模拟数据存储，设计增改查函数，并做数据判断，以数据链接为查询条件，若已存在，则进行修改操作，若不存在，则进行插入操作（数据解析使用json，参见3）
   ```
   //初始化mysql连接，并返回数据库
func init_sql() *sql.DB {
	db, err := sql.Open("mysql", database) //连接mysql数据库
	checkErr(err)                          //错误判断
	return db                              //返回数据库
}
   ```
   ```
   //查询函数，并返回是否有查询结果
func query(db *sql.DB, vtu vtuber) bool {
	sql_str := "select * from dd where Vtuber_Url='" + vtu.Vtuber_Url + "';" //要执行的sql语句
	rows, err := db.Query(sql_str)                                           //执行语句并获取返回值
	checkErr(err)                                                            //错误判断
	return rows.Next()                                                       //返回是否有查询结果
}
   ```
   ```
   //插入函数
func insert(db *sql.DB, vtu vtuber) {
	sql_str := "insert into dd (Live_Name, Vtuber_Name, Vtuber_Url) value ('" + vtu.Live_Name + "','" + vtu.Vtuber_Name + "','" + vtu.Vtuber_Url + "')" //要执行的语句
	_, err := db.Exec(sql_str)                                                                                                                          //执行语句
	checkErr(err)                                                                                                                                       //错误判断
	fmt.Println("insert")                                                                                                                               //若插入成功输出（复杂情况下可用Log）
}
   ```
   ```
   //修改函数
func update(db *sql.DB, vtu vtuber) {
	sql_str := "update dd set Live_Name='" + vtu.Live_Name + "',Vtuber_Name ='" + vtu.Vtuber_Name + "' where Vtuber_Url='" + vtu.Vtuber_Url + "'" //要执行的语句
	_, err := db.Exec(sql_str)                                                                                                                    //执行语句
	checkErr(err)                                                                                                                                 //错误判断
	fmt.Println("update")                                                                                                                         //若修改成功输出（复杂情况下可用Log）
}
   ```
   ```
   //保存Vtuber数据
func saveData(db *sql.DB, vtubers [][]byte) {
	for _, v := range vtubers { //遍历获取的Vtuber
		var vtu = vtuber{}             //设置接收解析json的Vtuber
		err := json.Unmarshal(v, &vtu) //解析得到的json
		checkErr(err)                  //错误判断
		//查询是否已有该Vtuber，通过直播间地址判断
		if query(db, vtu) { //若查询成功则调用修改函数
			update(db, vtu) //修改
		} else { //若查询失败则调用插入函数
			insert(db, vtu) //插入
		}
	}
}
   ```
#### 附录：
   使用数据结构：
   ```
   //Vtuber属性
type vtuber struct {
	Vtuber_Url  string //Vtuber直播的地址
	Live_Name   string //Vtuber直播间名字
	Vtuber_Name string //Vtuber名字（kksk）
}
   ```
   
