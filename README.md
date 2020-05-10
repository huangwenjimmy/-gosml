# gosml

 gosml ��Ϊ [![sml](https://github.com/huangwenjimmy/sml) GOLANGʵ�֣������˴󲿷ֹ����ࣨHttps,SqlTemplate,ManagedQueue,Cron,Scheduler,gdbc�����ɻ������ÿ��ٿ���restful�ӿ�
 
 ## Features
 
 ** SqlTemplate ���ٷ�������ݲ�ѯģ�壬֧�����ô�����ݿ�,�򱾵��ļ�Զ���ļ���
 
 ** Https ��ʽ��񣬶Ը����ѯ������[url��������ͷ������body�������ļ���]�Ƚ��з�װ����ͳһ����
 
 ** Cron cron���ʾ��ʵ��95%���ϣ��������ӷ��Ϲ��˵���д��������趨
 
 ** ManagedQueue �̶߳��й�������֧��ָ��Э���������д�С������������ͳ�ơ������·���
 
 ** Scheduler ��������֧��cron���������뼶��ȷ������֧��������������ȣ�֧�����ڵ��ȣ���������ȣ���ʱ����
 
 ** Gdbc ��JDBC������֧�ֻص������ṩ���ò�ѯapi
 
 ## Usage
 
 ### Gdbc
 
 ����ʹ�� gddc
 
 ```go
 package main

import (
	_ "github.com/go-sql-driver/mysql"
	"github.com/huangwenjimmy/gosml"
	"fmt"
)


func main1(){
	//init sml sqltemplate
	stq:=gosml.Init(gosml.Ds{Id:"defJt",Url:"root:rooxxx@tcp(192.168.1.xxx:3306)/xxxx",DbType:"mysql"})
	stq.Cache(gosml.NewDefaultCacheManager()) //��ӻ���
	result_map,_:=stq.Gdbcs["defJt"].QueryForMap("select 1 as a where 1=?",1)
	fmt.Println(result_map) //return map[string]interface{}
	result_list,_:=stq.Gdbcs["defJt"].QueryForList("select * from (select 1 as a union all select 2 as a) t where a=$1",2)
	fmt.Println(result_list)//result []map[string]interface{}
	stq.Gdbcs["defJt"].QueryForCall("select * from sml_if where limit ?",func(row map[string]interface{}) (bool,error){
			//�����ڵ�������ʽ����,����bool error���ϼ��ж�
			return true,nil
		},10000)
	//sml template api֧������sql�ο�java-api
	result_multi,_:=stq.Query("test-if",map[string]string{"id":"as","b":"hw","age":"30","time":"2020-04-22T10:06:04Z","dd":"1,2,3","times":"201904,201908"})
	fmt.Println(result_multi)//result    map list page
}
 ```
  ### Cron
  
  crontab���ʾʵ�֣�֧���뼶���´�ִ��ʱ�䣬���ʾ����
* ss mi HH day-of-mon Mon day-of-w Year[�ɲ�ѡ]
*  0 0/5 * * * ?          //��0�㣬ÿ��5����ִ��  
*  0 0,30 9-20 * * ?      //ÿ��9�㵽20��,0�ֺ�30��ִ��
*  0 0 9 1 * ?            //ÿ��1��9����ִ��
*  0 40 23 L * ?          //ÿ�����һ��23:40��ִ��
*  0 0 9 ? * MON          //ÿ��һ9����ִ�� 
*  0 0 9 ? * FRIL         //ÿ�����һ��������9����ִ��
*  0 0 9 1 JUN-DEC *      //7����12��ÿ1��9����ִ��

```go
package main

import (
	"github.com/huangwenjimmy/gosml"
	"github.com/huangwenjimmy/gosml/queue"
	"fmt"
)

func main(){
	cp:=queue.NewCronParser("0 1 9 * * ?")
	flag:=cp.Valid(gosml.ConvertStrToDate("20200428090100"))
	fmt.Println(flag) //true
	//trigger
	tri:=queue.NewTrigger("0 1 9 * * ?")
	fmt.Println(tri.GetNext())//��ǰʱ���´�ִ��ʱ��
	fmt.Println(tri.GetNextDate(gosml.ConvertStrToDate("20200428090100")))//ָ��ʱ���´�ִ��ʱ��
}
```

### ManagedQueue|Scheduler

������+�̹߳��������ʹ�ã���Ч��ת���ڲ������������

```go
package main

import (
	"github.com/huangwenjimmy/gosml/queue"
	"fmt"
	"time"
)

func main(){
	ds:=queue.NewScheduler("defaultScheduler",10,10000)//����������ָ��10��Э�̣�10000���������
	ds.InitScheduler()//��ʼ��
	//�·���������
	ds.SchedulerJob("3/5 * * * * ?",&DJ{"job1"})//��ӽṹ������  implements  queue.Job method ToString|Run
	//�·���������
	ds.SchedulerFuncJob("0 0/5 * * * ?","funcJob1",func(){
		fmt.Println("funcJob1",time.Now())
	})
	//ִ��һ��������
	//ds.ExecuteTask(task)//task implements queue.Task
	time.Sleep(time.Hour)
}
type DJ struct{
	name string
}
func (dj *DJ) ToString() string{
	return dj.name
}
func (dj *DJ) Run() error{
	fmt.Println(dj.name,time.Now())
	return nil
}
```

### Https   
 ֧�ֳ�������

```go
package main

import (
	"github.com/huangwenjimmy/gosml"
	"fmt"
	"time"
	"strings"
)

func main(){
	//��ӱ�ͷ  basic ��֤   ָ�����Ӷ�д��ʱʱ��
	https:=gosml.NewGetHttps("https://localhost:16003/bus/sml/query/if-cfg-interMngLike").
	Header("content-type","application/json").
	BasicAuth("user:passwd").
	ConnectTimeout(time.Second).RWTimeout(time.Second).
	Param("ids","10001").Param("name","����")
	https.Execute()
	//֧�ֶԷ��ؽ�����ж��ֲ���,��ȡ���������л������ص�
	fmt.Println(https.GetBodyString())//https.GetBodyBytes()|https.GetBodyTo(writer)|https.GetBodyToFile(*File)
	//post-body����body֧��   string []byte  io.reader
	https=gosml.NewPostBodyHttps("https://localhost:16003/bus/sml/query/if-cfg-interMngLike").Body(`{"ids":"1001","name":"����"}`)
	https.Execute()
	fmt.Println(https.GetBodyString())
	//post-upload�ϴ�  ��Ҫָ��Multipart  ������Param url����������ƴ����url  ͨ��gosml.UpFile  ����Ӷ���ļ�
	https=gosml.NewPostFormHttps("https://localhost:16003/bus/sml/web/file/upload").Multipart().
	Param("filepath","../logs").
	UpFile(&gosml.UpFile{FormName:"file",FileName:"hello1.txt",Input:strings.NewReader("helloworld234�찡\n�����ϴ�������")})
	https.Execute()
	fmt.Println(https.Response.Status,https.GetBodyString())
}
```



 
  
 