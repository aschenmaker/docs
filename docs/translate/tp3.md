# 【翻译计划3】Handling 1 Million Requests per Minute with Golang

Created: Mar 14, 2021 3:41 PM
Year: 2017
link: https://medium.com/smsjunk/handling-1-million-requests-per-minute-with-golang-f70ac505fcaa
writer: Marcio Castilho
标签: golang, 技术

【About Project】翻译计划是旨在于分享一些外文的技术、或者生活内容。这是本系列的第五篇翻译文章。

【About Article】如何用Golang处理每分钟百万级的请求？一个协程池，就够了！

![https://dist.lyneee.com/blog/2021-04-19-1-article-pic.jpeg](https://dist.lyneee.com/blog/2021-04-19-1-article-pic.jpeg)

以下是本文的正式内容：

---

# 如何用golang处理百万请求

我在几家不同的公司从事发垃圾邮件、反病毒和反恶意软件工作已经超过15年了。因为我们每天要处理大量的数据，我知道这些系统最终会有多么复杂。

目前，我是smsjunk.com的首席执行官以及KnowBe4的首席架构师(Chief Architect Officer)，都是从事网络安全行业的公司。

有趣的是，在过去十年的时间里，作为一名软件工程师，我所参与的所有web后端开发大部分都是用Ruby on Rails完成的。不要误会我的意思，我喜欢Ruby on Rails，我相信这是一个非常棒的环境，但是在你开始用Ruby的方式去思考和设计系统后，你可能忘了去使用多线程、并行化、快速执行以及小的内存开销来使软件架构高效和简单。多年来，我一直是一名C/C++、Delphi和C#的开发人员，我才开始意识到，使用合适的工具，能够让事情变得更加简单。

> 我不是很理解互联网总是在争论各种语言和框架优劣。我认为，*高效、生产力强和代码可维护性主要取决于构建方案的简单程度。*

## 问题所在 The Problem

在开发一个匿名的遥测和分析系统时，我们的目标是能够处理来自数百万个Endpoint的大量POST请求。Web handler能收到一个JSON格式的文档，其中可能包含有要写入到Amazon S3中的有效内容，以便于我们的map-reduce系统以后能够对这些数据进行操作。

一般情况，我们会考虑创建一个worker层级的架构，利用一下的技术：

- Sidekiq
- Resque Reque
- DelayedJob
- Elasticbeanstalk Woker tier
- RabbitMQ
- and so on ……

设置了两个不同的集群，一个用于web前端，一个用于工作节点，这样我们就可以扩大我们后台的工作量。

但是从一开始，我们的团队就知道我们要用Go语言来做他，因为在讨论阶段我们已经意识到这可能是一个非常大的流量系统。我已经使用了Go两年了，我们在这里开发了一些系统，但是没有一个可以承载这么大的流量。

我们首先创建了一些结构来定义Post请求要接受的web请求的内容，并创建了一个将其上传到我们S3 bucket的方法。

```golang
type PayloadCollection struct {
	WindowsVersion  string    `json:"version"`
	Token           string    `json:"token"`
	Payloads        []Payload `json:"data"`
}

type Payload struct {
    // [redacted]
}

func (p *Payload) UploadToS3() error {
	// the storageFolder method ensures that there are no name collision in
	// case we get same timestamp in the key name
	storage_path := fmt.Sprintf("%v/%v", p.storageFolder, time.Now().UnixNano())

	bucket := S3Bucket

	b := new(bytes.Buffer)
	encodeErr := json.NewEncoder(b).Encode(payload)
	if encodeErr != nil {
		return encodeErr
	}

	// Everything we post to the S3 bucket should be marked 'private'
	var acl = s3.Private
	var contentType = "application/octet-stream"

	return bucket.PutReader(storage_path, b, int64(b.Len()), contentType, acl, s3.Options{})
}
```

## 原生方法使用Go routines

Native approach to Go routines

最初我们使用了一个非常简单的POST处理程序实现了，只是试图将任务并行化的一个简单goroutine：

```go
func payloadHandler(w http.ResponseWriter, r *http.Request) {

	if r.Method != "POST" {
		w.WriteHeader(http.StatusMethodNotAllowed)
		return
	}

	// Read the body into a string for json decoding
	var content = &PayloadCollection{}
	err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
	if err != nil {
		w.Header().Set("Content-Type", "application/json; charset=UTF-8")
		w.WriteHeader(http.StatusBadRequest)
		return
	}
	
	// Go through each payload and queue items individually to be posted to S3
	for _, payload := range content.Payloads {
		go payload.UploadToS3()   // <----- DON'T DO THIS
	}

	w.WriteHeader(http.StatusOK)
}
```

对于中等温和的负载，这对大多数人来说是可行的，但是这很快被证明在大规模使用的情况下不是很好。我们预料到有很多请求，但是我们将第一个版本部署到生产环境中时，数量并没有达到我们开始后看到的数量级。我们完全低估了请求量。

上买呢方法在几个不同的方面都很糟糕，我们完全没有办法控制我们产生了多少个Go routines。所以这段代码很快就崩溃了。

## 再试一次Trying again

我们需要找到一种不同的方法。因为从一开始，我们就开始讨论如何使请求处理的生命周期非常短，并在后台产生处理。当然，这是在Ruby on Rails的世界里必须要做的事情，否则我们会阻塞所有可以工作的web处理器，无论我们使用的是puma、unicorn还是passenger（请不要讨论JRuby）。那么我们需要利用常见的方案去做到这点，比如Resque、Sidekip、SQS等等。这样的方案还有很多，因为很多方法可以达到这个目的。

因此第二次迭代是创建一个缓冲通道，我们可以创建一个buffer channel，在其中排队一些任务，然后上传到S3上，因为我们能够控制队列最大数量，并且有足够的内存来排队的任务，因此我们认为可以只在channel queue里缓存任务。

```go
var Queue chan Payload

func init() {
    Queue = make(chan Payload, MAX_QUEUE)
}

func payloadHandler(w http.ResponseWriter, r *http.Request) {
    ...
    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {
        Queue <- payload
    }
    ...
}
```

然后，为了实际地接触任务队列并消除他们，我们使用了类似的东西：

```go
func StartProcessor() {
    for {
        select {
        case job := <-Queue:
            job.payload.UploadToS3()  // <-- STILL NOT GOOD
        }
    }
}
```

老实说，我不知道我们那时候到底在想什么。这一定是一个用红牛填满的夜晚。这种方法没有给我们带阿里任何好处，我们用一个缓冲队列来交换有缺陷的并发性，只是简单的推迟了问题的解决。我们的同步处理器一次只上传一个有效内容到S3上，而传入速率远远大于了单个处理器上传到S3的能力，我们的缓冲通道很快到达了极限，阻塞了Request handler去把更多的任务加入到队列。

我们只是在回避这个问题，然后开始倒计时，知道我们系统最后死亡。在我们部署这个有缺小的版本后，延迟率以一个恒定的速度不断增加。

![https://dist.lyneee.com/blog/2021-04-19-2-million-requests.png](https://dist.lyneee.com/blog/2021-04-19-2-million-requests.png)

## 更好的解决方案

我们已经决定在使用Go channels的时使用一个通用模式，以便于创建一个两层的channel系统，一个用于排队作业，一个用于控制在JobQueue上的并发操作的woker数量。

我们的想法是将上传到S3的操作并行化到一个可持续的速率，这个速率不会使得机器瘫痪，但是也不会从S3产生连接错误。因此，我们选择了一个Job/Worker的模式。对于那些熟悉Java、C#的人来说，可以把这个看作是实现了一个通过channel实现的worker线程池的Golang方法。

```go
var (
	MaxWorker = os.Getenv("MAX_WORKERS")
	MaxQueue  = os.Getenv("MAX_QUEUE")
)

// Job represents the job to be run
type Job struct {
	Payload Payload
}

// A buffered channel that we can send work requests on.
var JobQueue chan Job

// Worker represents the worker that executes the job
type Worker struct {
	WorkerPool  chan chan Job
	JobChannel  chan Job
	quit    	  chan bool
}

func NewWorker(workerPool chan chan Job) Worker {
	return Worker{
		WorkerPool: workerPool,
		JobChannel: make(chan Job),
		quit:       make(chan bool)}
}

// Start method starts the run loop for the worker, listening for a quit channel in
// case we need to stop it
func (w Worker) Start() {
	go func() {
		for {
			// register the current worker into the worker queue.
			w.WorkerPool <- w.JobChannel

			select {
			case job := <-w.JobChannel:
				// we have received a work request.
				if err := job.Payload.UploadToS3(); err != nil {
					log.Errorf("Error uploading to S3: %s", err.Error())
				}

			case <-w.quit:
				// we have received a signal to stop
				return
			}
		}
	}()
}

// Stop signals the worker to stop listening for work requests.
func (w Worker) Stop() {
	go func() {
		w.quit <- true
	}()
}

```

我们修改了web请求的handler，使用有效负载创建Job struct的实力，并将其发送到Job Queue通道，以便于worker进行提取

```go
func payloadHandler(w http.ResponseWriter, r *http.Request) {

	if r.Method != "POST" {
				w.WriteHeader(http.StatusMethodNotAllowed)
				return
	}

   // Read the body into a string for json decoding
	var content = &PayloadCollection{}
	err := json.NewDecoder(io.LimitReader(r.Body, MaxLength)).Decode(&content)
  if err != nil {
			w.Header().Set("Content-Type", "application/json; charset=UTF-8")
			w.WriteHeader(http.StatusBadRequest)
			return
	}

    // Go through each payload and queue items individually to be posted to S3
    for _, payload := range content.Payloads {
        // let's create a job with the payload
        work := Job{Payload: payload}
        // Push the work onto the queue.
        JobQueue <- work
    }
	    w.WriteHeader(http.StatusOK)
}
```

在web服务器初始化期间，我们穿件了一个Dispatcher并调度Run()函数来创建workerpool，并开始侦听出现在Job Queue中的任务。

```go
dispatcher := NewDispatcher(MaxWorker) 
dispatcher.Run()
```

下面是我们的dispatcher的实现代码：

```go
type Dispatcher struct {
	// A pool of workers channels that are registered with the dispatcher
	WorkerPool chan chan Job
}

func NewDispatcher(maxWorkers int) *Dispatcher {
	pool := make(chan chan Job, maxWorkers)
	return &Dispatcher{WorkerPool: pool}
}

func (d *Dispatcher) Run() {
    // starting n number of workers
	for i := 0; i < d.maxWorkers; i++ {
		worker := NewWorker(d.pool)
		worker.Start()
	}

	go d.dispatch()
}

func (d *Dispatcher) dispatch() {
	for {
		select {
		case job := <-JobQueue:
			// a job request has been received
			go func(job Job) {
				// try to obtain a worker job channel that is available.
				// this will block until a worker is idle
				jobChannel := <-d.WorkerPool

				// dispatch the job to the worker job channel
				jobChannel <- job
			}(job)
		}
	}
}
```

注意到，我们提供了要实例化并添加到worker pool中最大worker数量。因为我们使用了在Amazon Elastic beanstalk为这个项目使用了容器化的Go环境，我们也试图去遵循[十二因素](https://12factor.net/zh_cn/)的方法来在生产环境中配置我们的系统，所以我们从环境变量中读取出这些值。这样我们就能够控制任务队列和worker的数量，这样我们就可以在不需要重新部署集群的情况下快速调整这些值的大小。

```go

var ( 
  MaxWorker = os.Getenv("MAX_WORKERS") 
  MaxQueue  = os.Getenv("MAX_QUEUE") 
)
```

在我们部署它之后，我们发现所有的延迟率都又了显著的降低，我们处理请求的能力快速上升。

![https://dist.lyneee.com/blog/2021-04-19-3-ability-to-handle-requests.png](https://dist.lyneee.com/blog/2021-04-19-3-ability-to-handle-requests.png)

在弹性负载均衡器(Elastic Load Balancer)完全准备好的几分钟后，我们看到Elastic Bean stalk上，每分钟处理了接近一百万个请求。我们通常在早上的几个小时内，我们的流量会达到每分钟一百万以上。

当我们重新部署了新的代码，服务器的数量就从一百台大幅下降到了20台左右。

![https://dist.lyneee.com/blog/2021-04-19-4-after-EAS.png](https://dist.lyneee.com/blog/2021-04-19-4-after-EAS.png)

在我们恰当的配置好集群和自动缩放设置之后，我们能够将它降低到只有4x EC2 c4。如果CPU连续五分钟超过了90%，弹性自动伸缩则会添加一个新的实例。

## 总结

在我看来，简洁总是占上风的。(Simplicity always wins in my book)。我们本可以设计一个复杂的系统，包括许多的队列、后台worker、复杂的部署，但是我们决定利用Elasticbeanstalk自动圣所的能力，以及golang提供给我们的高效、简单的并发方法。

工作总有合适的工具。有时候当你的Ruby on Rails系统需要一个分场强大的web处理程序的时候，你可以考虑一下Ruby生态以外的更简单而且更强大的替代方案。



## 参考

[1] 十二因素：软件即服务(SaaS)的交付准则；https://12factor.net/zh_cn/


