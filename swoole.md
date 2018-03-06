#C编写的一个通信扩展
  2.php的cli模式(Command Line Interface)
   php -v
   php -i
   php -l 1.php  (检测1.php脚本语法错误)
   php -r "echo 'hello world';" (直接调用后面的语法)
   php -m (查看加载模块)


  3.进程和线程
   Linux的fork()函数通过系统调用即可实现创建一个与原进程几乎相同的进程。对于多任务，通常我们会设计Master-Worker模式，即一个Master进程负责分配任务，多个Worker进程负责执行任务。同理，如果是多线程，Master就是主线程，Worker就是子线程。
   单进程多线程
   多进程多线程  apache多进程
   多进程单线程  php为单线程单线程
   nginx/apache/php-fpm 是多进程的
   php-fpm 多进程单线程


  4.IO模型
   同步IO 等待IO操作结果会堵塞当前进程
   异步IO 只发出IO指令(执行其他指令)返回结果后在处理


  5.通信协议
   TCP/IP


  6.安装


  7.异步多线程服务器
    socket编程 套接字 是用来与另一个进程进行跨网络通信的文件(函数库)

    #7.1 swoole_server只能运行在cli模式
    `<?php
    $ser = new swoole_server('127.0.0.1',9501);

    //worker进程数 cpu的1-4倍
    $ser ->set([
      'worker_num'=>2,
      ]);

    //有新的客户端连接时，worker进程内会触发该回调
    $serv->on('Connect', function ($serv, $fd) {
        echo "new client connected." . PHP_EOL;
    });

    //server接收到客户端的数据后，worker进程内触发该回调
    $serv->on('Receive', function ($serv, $fd, $fromId, $data) {
        // 收到数据后发送给客户端
        $serv->send($fd, 'hello server'. $data);
    });

    //客户端断开连接或者server主动关闭连接时 worker进程内调用
    $serv->on('Close', function ($serv, $fd) {
        echo "Client close." . PHP_EOL;
    });

    // 启动server
    $serv->start();
    ?>`
    *Connect 和 Receive两个回调，都强调了是在worker进程内触发的
    **

    #7.2 swoole_client模拟客户端（运行在cli） 是面向web应用

    `<?php
    // 创建一个同步阻塞的tcp socket
    // 第一个参数是表示socket的类型，有下面四种类型选择，这里选则tcp socket就好
    /**
     *     SWOOLE_SOCK_TCP 创建tcp socket
     *     SWOOLE_SOCK_TCP6 创建tcp ipv6 socket
     *     SWOOLE_SOCK_UDP 创建udp socket
     *     SWOOLE_SOCK_UDP6 创建udp ipv6 socket
     */
    // 第二个参数是同步还是异步
    /**
     *     SWOOLE_SOCK_SYNC 同步客户端
     *     SWOOLE_SOCK_ASYNC 异步客户端
     */
    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC);

    // 随后建立连接，连接失败直接退出并打印错误码
    $client->connect('127.0.0.1', 9501) || exit("connect failed. Error: {$client->errCode}\n");
    // 向服务端发送数据
    $client->send("hello server.");
    // 从服务端接收数据
    $response = $client->recv();
    // 输出接受到的数据
    echo $response . PHP_EOL;
    // 关闭连接
    $client->close()
    ?>`

  8.task(异步任务)
    #7.1 ASync Task(异步任务)
    我们可以利用AsyncTask将一个耗时的任务投递到队列中，由进程池异步去执行。
    task进程其实是要在worker进程内发起的，通过worker进程投递到task进程中去处理
    swoole_server->task函数是非阻塞函数，任务投递到task进程中后会立即返回，不会影响到worker进程的其他操作。task进程却是阻塞的，task过少，塞满缓冲区，就会导致worker进程阻塞
    `<?php
    $serv = new swoole_server("127.0.0.1", 9501);
    //2、开启task功能
    //task功能默认是关闭的，开启task功能需要满足两个条件

    //配置task进程的数量
    //注册task的回调函数onTask和onFinish
    //配置task进程的数量，即配置task_worker_num这个配置项。比如我们开启一个task进程
    $serv->set([
      'task_worker_num' => 1,
    ]);

    $serv->on('Connect', function ($serv, $fd) {
        echo "new client connected." . PHP_EOL;
    });
    $serv->on('Receive', function ($serv, $fd, $fromId, $data) {
        echo "worker received data: {$data}" . PHP_EOL;

        // 投递一个任务到task进程中
        $serv->task($data);

        // 通知客户端server收到数据了
        $serv->send($fd, 'This is a message from server.');

        // 为了校验task是否是异步的，这里和task进程内都输出内容，看看谁先输出
        echo "worker continue run."  . PHP_EOL;
    });

    //task回调注册
    /**
     * $serv swoole_server
     * $taskId投递的任务id,因为task进程是由worker进程发起，所以多worker多task下，该值可能会相同
     * $fromId 来自那个worker进程的id
     * $data 要投递的任务数据
     */
    $serv->on('Task', function ($serv, $taskId, $fromId, $data) {
        echo "task start. --- from worker id: {$fromId}." . PHP_EOL;
        for ($i=0; $i < 5; $i++) {
            sleep(1);
            echo "task runing. --- {$i}" . PHP_EOL;
        }
        echo "task end." . PHP_EOL;
    });

    /**
     * 只有在task进程中调用了finish方法或者return了结果，才会触发finish
     */
    $serv->on('Finish', function ($serv, $taskId, $data) {
        echo "finish received data '{$data}'" . PHP_EOL;
    });

    ?>`
    1-没有耗时任务的情况下，worker直接运行，无需开启task
    2-对于耗时的任务，可以在worker内调用task函数，把异步任务投递给task进程进行处理，task进程的数量取决于task_worker_num的配置
    3-task进程内可以选择调用finish方法或者return，来通知worker进程此任务已完成，worker进程会在onFinish回调中对task的执行结果进一步处理。如果worker进程不关心任务的结果，finish就不需要了


  9.运行流程和进程模型
     `<?php
     $serv = new swoole_server('127.0.0.1', 9501);
     $serv->set([
         'worker_num' => 2,
         'task_worker_num' => 1,
     ]);
     $serv->on('Connect', function ($serv, $fd) {
     });
     $serv->on('Receive', function ($serv, $fd, $fromId, $data) {
     });
     $serv->on('Close', function ($serv, $fd) {
     });
     $serv->on('Task', function ($serv, $taskId, $fromId, $data) {
     });
     $serv->on('Finish', function ($serv, $taskId, $data) {
     });

     $serv->start();
     ?>`

     以上脚本：swoole的进程模型可以用Master-Manager-Worker
     (1个Master进程+1个Manager进程+2个Worker进程+1个Task进程）
     --Master进程，Master进程就是我们的主进程。Master进程，包括主线程，多个Reactor线程等。
       ·主线程用于Accept,信号处理等操作，而Reactor线程是处理tcp连接，处理网络IO，收发数据的线程
       ·主线程处理完新的连接后，会将这个连接分配给固定的Reactor线程，并且这个Reactor线程会一直负责监听此socke。
       ·socket可读时，会读取数据，并将该请求分配给worker进程，worker进程内的回调onReceive的第三个参数$fromId；socket可写时，会把数据发送给tcp客户端。
     --Manager进程,Manager进程就专门负责worker/task进程的fork操作和管理。
       ·真正实现业务逻辑，是在worker/task进程内完成的

       Master进程：
           启动：onStart
           关闭：onShutdown
       Manager进程：
           启动：onManagerStart
           关闭：onManagerStop
       Worker进程：
           启动：onWorkerStart
           关闭：onWorkerStop
       提醒：task_worker也会触发onWorkerStart回调

       `<?php
       $serv->on("start", function ($serv){
           swoole_set_process_name('server-process: master');
       });
       // 以下回调发生在Manager进程
       $serv->on('ManagerStart', function ($serv){
           swoole_set_process_name('server-process: manager');
       });
       $serv->on('WorkerStart', function ($serv, $workerId){
           if($workerId >= $serv->setting['worker_num']) {
               swoole_set_process_name("server-process: task");
           } else {
               swoole_set_process_name("server-process: worker");
           }
       });
       ?>`
       0~worker_num表示worker进程的标识，包括0但不包括worker_num
       worker_num~worker_num+task_worker_num是task进程的标识。


  10.常驻内存,内存泄漏

    ###但是，对开发人员的要求也更高了。因为这些资源常驻内存，并不会像web模式下，在请求结束之后会释放内存和资源。也就是说我们在操作中一旦没有处理好，就会发生内存泄漏，久而久之就可能会发生内存溢出。？？？

    ###server一开始就把我们的代码加载到内存中了，无论后期我们怎么修改本地磁盘上的代码，客户端再次发起请求的时候，永远都是内存中的代码在生效，所以我们只能终止server，释放内存然后再重启server，重新把新的代码加载到内存中，如此，明白否？？？

    ###尽量避免使用全局变量！woole还提供了max_request机制，我们可以配置max_request和task_max_request这两个参数来避免内存溢出

    `$serv->set([
          'worker_num' => 1,
          'task_worker_num' => 1,
          'max_request' => 3,
          'task_max_request' => 4,
      ]);`


  11.守护进程、信号、平滑重启

    --woole官方也为我们提供了配置选项daemonize，默认不启用守护进程，若要开启守护进程，daemonize设置为true即可。

    --信号是软件中断，每一个信号都有一个名字。通常，信号的名字都以“SIG”开头，比如我们最熟悉的Ctrl+C就是一个名字叫“SIGINT”的信号，意味着“终端中断”。

    --平滑重启，也叫“热重启”。
      SIGTERM，一种优雅的终止信号，会待进程执行完当前程序之后中断，而不是直接干掉进程
      SIGUSR1，将平稳的重启所有的Worker进程
      SIGUSR2，将平稳的重启所有的Task进程

    ●在swoole中，重启只能针对Worker进程启动之后载入的文件有效！只有在onWorkerStart回调之后加载的文件，重启才有意义。在Worker进程启动之前就已经加载到内存中的文件，只能server再重启

    ●发送SIGUSR1信号:
      ps aux|grep Reload.php
      kill -USR1 5953

    #不生效实例：新的程序代码只针对在onWorkerStart回调之后才加载进来的php文件才能生效，Test.php是在class定义之前就加载进来了，所以肯定不生效。
    `<?php
    class Test
    {
        public function run($data)
        {
            echo $data;
        }
    }
    ?>`

    `<?php
     require_once("Test.php");
    class NoReload
    {
        private $_serv;
        private $_test;

        /**
         * init
         */
        public function __construct()
        {
            $this->_serv = new Swoole\Server("127.0.0.1", 9501);
            $this->_serv->set([
                'worker_num' => 1,
            ]);
            $this->_serv->on('Receive', [$this, 'onReceive']);

            $this->_test = new Test;
        }
        /**
         * start server
         */
        public function start()
        {
            $this->_serv->start();
        }
        public function onReceive($serv, $fd, $fromId, $data)
        {
            $this->_test->run($data);
        }
    }
    $noReload = new NoReload;
    $noReload->start();
    ?>`

    #生效写法
    `<?php
    class Reload
    {
        private $_serv;
        private $_test;

        /**
         * init
         */
        public function __construct()
        {
            $this->_serv = new Swoole\Server("127.0.0.1", 9501);
            $this->_serv->set([
                'worker_num' => 1,
            ]);
            $this->_serv->on('Receive', [$this, 'onReceive']);
            $this->_serv->on('WorkerStart', [$this, 'onWorkerStart']);
        }
        /**
         * start server
         */
        public function start()
        {
            $this->_serv->start();
        }
        public function onWorkerStart($serv, $workerId)
        {
            require_once("Test.php");
            $this->_test = new Test;
        }
        public function onReceive($serv, $fd, $fromId, $data)
        {
            $this->_test->run($data);
        }
    }

    $reload = new Reload;
    $reload->start();
    ?>`

    #进程守护配置  (./server.log 重定向输出位置)
    `$this->_serv->set([
        'worker_num' => 1,
        'daemonize' => true,
        'log_file' => __DIR__ . '/server.log',
    ]);
    `


  12.swoole之定时器
    ▲永久性定时器
        int swoole_timer_tick(int $ms, callable $callback, mixed $params);
          $ms 指时间，单位毫秒
          $callback 回调函数，定时器创建后会调用该函数
          $params 传递给回调函数的参数

       swoole_timer_tick(1000, function () {
            echo "This is a tick.\n";
        });

       bool swoole_timer_clear(int $timerId)

       <?php
          $i = 0;

          swoole_timer_tick(1000, function ($timeId, $params) use (&$i) {
              $i ++;
              echo "hello, {$params} --- {$i}\n";
              if ($i >= 5) {
                  swoole_timer_clear($timeId);
              }
          }, 'world');
       ?>

          # server_swoole 面向对象写法

          $serv->set([
              'worker_num' => 2,
          ]);
          $serv->on('WorkerStart', function ($serv, $workerId){
              if ($workerId == 0) {
                  $i = 0;
                  $params = 'world';
                  $serv->tick(1000, function ($timeId) use ($serv, &$i, $params) {
                      $i ++;
                      echo "hello, {$params} --- {$i}\n";
                      if ($i >= 5) {
                          $serv->clearTimer($timeId);
                      }
                  });
              }
          });



           代码总体上就不分析了，只看一点，为什么在onWorkerStart回调内判断了$workerId是否等于0？
          注意到我们开启了两个Worker进程，如果不判断，那么就会在两个Worker进程内各注册一个定时器，实际上也就是我们注册了两个相同的定时器，这是没有必要的。

    ▲一次性定时器
     #全局
      swoole_timer_after(3000, function () {
        echo "only once.\n";
      });
     #回调
      $serv->on('Receive', function ($serv, $fd, $fromId, $data) {
          $serv->after(3000, function () {
              echo "only once.\n";
          });
      });


  13.swoole之粘贴包
    socket的buffer缓冲 导致onReceive()回调不能保证完整性。
    #方法1：EOF结束协议
    `//服务端配置 自行分包
    $this->_serv->set([
        'worker_num' => 1,
        'open_eof_check' => true, //打开EOF检测
        'package_eof' => "\r\n", //设置EOF
    ]);
    //客户端消息最后加标识符
    for ($i = 0; $i < 3; $i++) {
    $client->send("Just a test.\r\n");
    }
    `
     //swoole提供了open_eof_split配置参数 自动分包
    `$this->_serv->set([
        'worker_num' => 1,
        'open_eof_check' => true, //打开EOF检测
        'package_eof' => "\r\n", //设置EOF
        'open_eof_split' => true,
    ]);
    public function onReceive($serv, $fd, $fromId, $data)
    {
         echo "Server received data: {$data}" . PHP_EOL;
    }
    `

    #方法二：固定包头+包体协议
     //服务端代码
    `<?php
    class ServerPack
    {
        private $_serv;

        /**
         * init
         */
        public function __construct()
        {
            $this->_serv = new Swoole\Server("127.0.0.1", 9501);
            $this->_serv->set([
                'worker_num' => 1,
                'open_length_check'     => true,      // 开启协议解析
                'package_length_type'   => 'N',     // 长度字段的类型
                'package_length_offset' => 0,       //第几个字节是包长度的值
                'package_body_offset'   => 4,       //第几个字节开始计算长度
                'package_max_length'    => 81920,  //协议最大长度
            ]);
            $this->_serv->on('Receive', [$this, 'onReceive']);
        }
        public function onReceive($serv, $fd, $fromId, $data)
        {
            $info = unpack('N', $data);
            $len = $info[1];
            $body = substr($data, - $len);
            echo "server received data: {$body}\n";
        }
        /**
         * start server
         */
        public function start()
        {
            $this->_serv->start();
        }
    }

    $reload = new ServerPack;
    $reload->start();
    ?>`
    //客户端代码
    `<?php

    $client = new swoole_client(SWOOLE_SOCK_TCP, SWOOLE_SOCK_SYNC);
    $client->connect('127.0.0.1', 9501) || exit("connect failed. Error: {$client->errCode}\n");

    // 向服务端发送数据
    for ($i = 0; $i < 3; $i++) {
        $data = "Just a test.";
        $data = pack('N', strlen($data)) . $data;
        $client->send($data);
    }

    $client->close();
    ?>
    `
    1、首先，在server端我们配置了open_length_check，该参数表明我们要开启固定包头协议解析

    2、package_length_type配置，表明包头长度的类型，这个类型跟客户端使用pack打包包头的类型一致，一般设置为N或者n，N表示4个字节，n表示2个字节

    3、我们看下客户端的代码 pack('N', strlen($data)) . $data，这句话就是包头+包体的意思，包头是pack函数打包的二进制数据，内容便是真实数据的长度 strlen($data)。

    在内存中，整数一般占用4个字节，所以我们看到，在这段数据中0-4字节表示的是包头，剩余的就是真实的数据。但是server不知道呀，怎么告诉server这一事实呢？

    看配置package_length_offset和package_body_offset，前者就是告诉server，从第几个字节开始是长度，后者就是从第几个字节开始计算长度。

    4、既然如此，我们就可以在onReceive回调对数据解包，然后从包头中取出包体长度，再从接收到的数据中截取真正的包体。


  14.异步发送邮件案例

    ##发送邮件类
     `<?php
     class Mailer
     {
         public $transport;
         public $mailer;
         /**
          * 发送邮件类 参数 $data 需要三个必填项 包括 邮件主题$data['subject']、接收邮件的    人$data['to']和邮件内容 $data['content']
          * @param Array $data
          * @return bool $result 发送成功 or 失败
          */
         public function send($data)
         {
             $this->transport = (new Swift_SmtpTransport('smtp.qq.com', 25))
                 ->setEncryption('tls')
                 ->setUsername('bailangzhan@qq.com')
                 ->setPassword('aslqcbarmwikbjab');
             $this->mailer = new Swift_Mailer($this->transport);

             $message = (new Swift_Message($data['subject']))
                 ->setFrom(array('bailangzhan@qq.com' => '白狼栈'))
                 ->setTo(array($data['to']))
                 ->setBody($data['content']);

             $result = $this->mailer->send($message);

             // 释放
             $this->destroy();
             return $result;
         }
         public function destroy()
         {
             $this->transport = null;
             $this->mailer = null;
         }
     }
     ?>
     `
     ##使用(测试邮件能否发送)
     `require_once __DIR__ . "/task/Mailer.php";
     $data = [
         'to' => '422744***@qq.com',
         'subject' => 'just a test',
         'content' => 'This is just a test.',
     ];
     $mailer = new Mailer;
     $mailer->send($data);
     `
     ##swoole server
     `<?php?
     class TaskServer
     {
         private $_serv;
         private $_run;
         /**
         * init
         */
         public function __construct()
         {
             $this->_serv = new Swoole\Server("127.0.0.1", 9501);
             $this->_serv->set([
                 'worker_num' => 2,
                 'daemonize' => false,
                 'log_file' => __DIR__ . '/server.log',
                 'task_worker_num' => 2,
                 'max_request' => 5000,
                 'task_max_request' => 5000,
                 'open_eof_check' => true, //打开EOF检测
                 'package_eof' => "\r\n", //设置EOF
                 'open_eof_split' => true, // 自动分包
             ]);
             $this->_serv->on('Connect', [$this, 'onConnect']);
             $this->_serv->on('Receive', [$this, 'onReceive']);
             $this->_serv->on('WorkerStart', [$this, 'onWorkerStart']);
             $this->_serv->on('Task', [$this, 'onTask']);
             $this->_serv->on('Finish', [$this, 'onFinish']);
             $this->_serv->on('Close', [$this, 'onClose']);
         }
         public function onConnect($serv, $fd, $fromId)
         {
         }
         public function onWorkerStart($serv, $workerId)
         {
             require_once __DIR__ . "/TaskRun.php";
             $this->_run = new TaskRun;
         }
         public function onReceive($serv, $fd, $fromId, $data)
         {
             $data = $this->unpack($data);
             $this->_run->receive($serv, $fd, $fromId, $data);
             // 投递一个任务到task进程中
             if (!empty($data['event'])) {
                 $serv->task(array_merge($data , ['fd' => $fd]));
             }
         }
         public function onTask($serv, $taskId, $fromId, $data)
         {
             $this->_run->task($serv, $taskId, $fromId, $data);
         }
         public function onFinish($serv, $taskId, $data)
         {
             $this->_run->finish($serv, $taskId, $data);
         }
         public function onClose($serv, $fd, $fromId)
         {
         }
         /**
         * 对数据包单独处理，数据包经过`json_decode`处理之后，只能是数组
         * @param $data
         * @return bool|mixed
         */
         public function unpack($data)
         {
             $data = str_replace("\r\n", '', $data);
             if (!$data) {
                 return false;
             }
             $data = json_decode($data, true);
             if (!$data || !is_array($data)) {
                 return false;
             }
             return $data;
         }
         public function start()
         {
             $this->_serv->start();
         }
     }
     $reload = new TaskServer;
     $reload->start();
     ?>`

     ##业务逻辑
    `<?php
    require_once ('./TaskClient.php');
    require_once ('./Mailer.php');
    class TaskRun
    {
        public function receive($serv, $fd, $fromId, $data)
        {
        }
        public function task($serv, $taskId, $fromId, $data)
        {
            try {
                switch ($data['event']) {
                    case TaskClient::EVENT_TYPE_SEND_MAIL:
                        $mailer = new Mailer;
                        $result = $mailer->send($data);
                        break;
                    default:
                        break;
                }
                return $result;
            } catch (\Exception $e) {
                throw new \Exception('task exception :' . $e->getMessage());
            }
        }
        public function finish($serv, $taskId, $data)
        {
            return true;
        }
    }
    ?>`

    ##swoole_client
    `<?php
    class TaskClient
    {
        private $client;
        const EVENT_TYPE_SEND_MAIL = 'send-mail';
        public function __construct ()
        {
            $this->client = new Swoole\Client(SWOOLE_SOCK_TCP);
            if (!$this->client->connect('127.0.0.1', 9501)) {
                $msg = 'swoole client connect failed.';
                throw new \Exception("Error: {$msg}.");
            }
        }
        /**
         * @param $data Array
         * send data
         */
        public function sendData ($data)
        {
            $data = $this->togetherDataByEof($data);
            $this->client->send($data);
        }
        /**
         * 数据末尾拼接EOF标记
         * @param Array $data 要处理的数据
         * @return String json_encode($data) . EOF
         */
        public function togetherDataByEof($data)
        {
            if (!is_array($data)) {
                return false;
            }
            return json_encode($data) . "\r\n";
        }
    }
    ?>`

    ##web可访问位置index.php
    `<?php
    require_once ("./swoole-practice/TaskClient.php");
    $data = [
        'event' => TaskClient::EVENT_TYPE_SEND_MAIL,
        'to' => '422744***@qq.com',
        'subject' => 'just a test',
        'content' => 'This just a test.',
    ];
    $client = new TaskClient();
    $client->sendData($data);
    ?>`


  15.swoole之http服务器
    #swoole_http_server
    swoole_http_server继承自swoole_server，废除了onConnect/onReceive回调，新增了onRequest回调。
    `<?php
    $http = new swoole_http_server("127.0.0.1", 8000);
    $http->on('request', function (swoole_http_request $request, swoole_http_response $response) {
        $response->status(200);
        $response->end('hello world.');
    });
    $http->start();
    ?>`

    #swoole_http_request
    `$http->on('request', function (swoole_http_request $request, swoole_http_response    $response) {

        print_r($request);

        $response->status(200);
        $response->end('hello world.');
    });
    `
    打印内容：
    fd 客户端标识
    header 请求的头信息，你可以对比下浏览器实际发出的request header，结果是一样的
    server 请求相关的服务器信息
    还有一项data，HTTP请求的详细信息

    #swoole_http_response
    负责处理HTTP响应信息，包括响应的头信息header\响应状态等。
    比如上例中 $response->status() 设置响应状态码，$response->end() 发送响应体并结束请求

    #nginx代理 (设置)
    `server {
        listen       80;
        root /var/www/test/;
        server_name  swoole.example.com;
        index index.php index.html;

        location / {
            if (!-e $request_filename) {
                proxy_pass http://127.0.0.1:8000;
            }
        }
    }
    `


  16.swoole之websocket协议
    相对于Http这种非持久连接而言，websocket协议是一种持久化连接，它是一种独立的，基于TCP的协议。基于websocket，我们可以实现客户端和服务端双向通信。
    swoole内置的websocket服务器，异步非阻塞多进程，牛逼的swoole！

    #创建websocket服务器
    `<?php

    class WebSocketServer
    {
        private $_serv;

        public function __construct()
        {
            $this->_serv = new swoole_websocket_server("127.0.0.1", 9501);
            $this->_serv->set([
                'worker_num' => 1,
            ]);
            $this->_serv->on('open', [$this, 'onOpen']);
            $this->_serv->on('message', [$this, 'onMessage']);
            $this->_serv->on('close', [$this, 'onClose']);
        }

        /**
         * @param $serv
         * @param $request
         */
        public function onOpen($serv, $request)
        {
            echo "server: handshake success with fd{$request->fd}.\n";
        }

        /**
         * @param $serv
         * @param $frame
         */
        public function onMessage($serv, $frame)
        {
            $serv->push($frame->fd, "server received data :{$frame->data}");
        }
        public function onClose($serv, $fd)
        {
            echo "client {$fd} closed.\n";
        }

        public function start()
        {
            $this->_serv->start();
        }
    }

    $server = new WebSocketServer;
    $server->start();
    ?>`

    #回调
    --onOpen回调：客户端与服务端建立连接的时候将触发该回调，回调的第二个参数是swoole_http_request对象，包括了http握手的一些信息，比如GET\COOKIE等

    --onMessage回调：这个是服务端收到客户端信息后回调，在该回调内我们调用了swoole_websocket_server::push方法向客户端推送了数据，注意哦，push的第一个参数只能是websocket客户端的标识

    客户端js写法：
    `<script>
      var ws = new WebSocket('ws://127.0.0.1:9501');
      ws.onopen = function(event) {
          // 发送消息
          ws.send('This is websocket client.');
      };

      // 监听消息
      ws.onmessage = function(event) {
          console.log('Client received a message: ', event.data);
      };
      ws.onclose = function(event) {
          console.log('Client has closed.\n', event);
      };
      </script>`

      --客户端先发送给server : This is websocket client.
      --server收到消息后，在onMessage回调内向客户端推送了 server received data :This is websocket client.
      --客户端在onmessage回调内收到server的信息，并在server发过来的消息的前面增加了 Client received a message: ，这才有了控制台上面展示的信息

      #改法
      <div>
      <textarea name="content" id="content" cols="30" rows="10"></textarea>
          <button onclick="send();">发送</button>
      </div>
      <div class="list" style="border: solid 1px #ccc; margin-top: 10px;">
          <ul id="ul">
          </ul>
      </div>
      <script>
      var ws = new WebSocket('ws://127.0.0.1:9501');
      ws.onopen = function(event) {
          ws.send('This is websocket client.');
      };
      ws.onmessage = function(event) {
          var data = event.data;
          var ul = document.getElementById('ul');
          var li = document.createElement('li');
          li.innerHTML = data;
          ul.appendChild(li);
      };
      ws.onclose = function(event) {
          console.log('Client has closed.\n', event);
      };

      function send() {
          var obj = document.getElementById('content');
          var content = obj.value;
          ws.send(content);
      }
      </script>

      #server更改
      `<?php
      public function onMessage($serv, $frame)
      {
          // 循环当前的所有连接，并把接收到的客户端信息全部发送
          foreach ($serv->connections as $fd) {
              $serv->push($fd, $frame->data);
          }
      }
      ?>`


  17.常见的websocket问题
      ·心跳检测的必要性
          $serv->set([
            'heartbeat_check_interval' => N,
            'heartbeat_idle_time' => M,
          ]);
          含义就是N秒检查一次，看看哪些连接M秒内没有活动的，server就会主动关闭这个无效的连接。

      ·校验客户端连接的有效性
          以client 连接server时携带token为例：
          #服务端：
          `<?php
          class WebSocketServerValid
          {
              private $_serv;
              public $key = '^manks.top&swoole$';

              public function __construct()
              {
                  $this->_serv = new swoole_websocket_server("127.0.0.1", 9501);
                  $this->_serv->set([
                      'worker_num' => 1,
                      'heartbeat_check_interval' => 30,
                      'heartbeat_idle_time' => 62,
                  ]);
                  $this->_serv->on('open', [$this, 'onOpen']);
                  $this->_serv->on('message', [$this, 'onMessage']);
                  $this->_serv->on('close', [$this, 'onClose']);
              }

              /**
               * @param $serv
               * @param $request
               */
              public function onOpen($serv, $request)
              {
                  $this->checkAccess($serv, $request);
              }

              /**
               * @param $serv
               * @param $frame
               */
              public function onMessage($serv, $frame)
              {
                  $this->_serv->push($frame->fd, 'Server: ' . $frame->data);
              }
              public function onClose($serv, $fd)
              {
                  echo "client {$fd} closed.\n";
              }

              /**
               * 校验客户端连接的合法性,无效的连接不允许连接
               * @param $serv
               * @param $request
               * @return mixed
               */
              public function checkAccess($serv, $request)
              {
                  // get不存在或者uid和token有一项不存在，关闭当前连接
                  if (!isset($request->get) || !isset($request->get['uid']) || !isset($request->get['token'])) {
                      $this->_serv->close($request->fd);
                      return false;
                  }
                  $uid = $request->get['uid'];
                  $token = $request->get['token'];
                  // 校验token是否正确,无效关闭连接
                  if (md5(md5($uid) . $this->key) != $token) {
                      $this->_serv->close($request->fd);
                      return false;
                  }
              }

              public function start()
              {
                  $this->_serv->start();
              }
          }

          $server = new WebSocketServerValid;
          $server->start();
          ?>`
      ·客户端的重连机制
        js重连；


  18.通知案例以及多端口复合
    swoole_websocket_server:
    `<?php

    class CommentServer
    {
        private $_serv;
        public $key = '^manks.top&swoole$';
        // 用户id和fd对应的映射,key => value,key是用户的uid,value是用户的fd
        public $user2fd = [];

        public function __construct()
        {
            $this->_serv = new swoole_websocket_server("127.0.0.1", 9501);
            $this->_serv->set([
                'worker_num' => 1,
                'heartbeat_check_interval' => 60,
                'heartbeat_idle_time' => 125,
            ]);
            $this->_serv->on('open', [$this, 'onOpen']);
            $this->_serv->on('message', [$this, 'onMessage']);
            $this->_serv->on('close', [$this, 'onClose']);
        }

        /**
         * @param $serv
         * @param $request
         * @return mixed
         */
        public function onOpen($serv, $request)
        {
            // 连接授权
            $accessResult = $this->checkAccess($serv, $request);
            if (!$accessResult) {
                return false;
            }
            // 始终把用户最新的fd跟uid映射在一起
            if (array_key_exists($request->get['uid'], $this->user2fd)) {
                $existFd = $this->user2fd[$request->get['uid']];
                $this->close($existFd, 'uid exists.');
                $this->user2fd[$request->get['uid']] = $request->fd;
                return false;
            } else {
                $this->user2fd[$request->get['uid']] = $request->fd;
            }
        }

        /**
         * @param $serv
         * @param $frame
         * @return mixed
         */
        public function onMessage($serv, $frame)
        {
            // 校验数据的有效性，我们认为数据被`json_decode`处理之后是数组并且数组的`event`项非空才是有效数据
            // 非有效数据，关闭该连接
            $data = $frame->data;
            $data = json_decode($data, true);
            if (!$data || !is_array($data) || empty($data['event'])) {
                $this->close($frame->fd, 'data format invalidate.');
                return false;
            }
            // 根据数据的`event`项，判断要做什么,`event`映射到当前类具体的某一个方法，方法不存在则关闭连接
            $method = $data['event'];
            if (!method_exists($this, $method)) {
                $this->close($frame->fd, 'event is not exists.');
                return false;
            }
            $this->$method($frame->fd, $data);
        }
        public function onClose($serv, $fd)
        {
            echo "client {$fd} closed.\n";
        }

        /**
         * 校验客户端连接的合法性,无效的连接不允许连接
         * @param $serv
         * @param $request
         * @return mixed
         */
        public function checkAccess($serv, $request)
        {
            // get不存在或者uid和token有一项不存在，关闭当前连接
            if (!isset($request->get) || !isset($request->get['uid']) || !isset($request->get['token'])) {
                $this->close($request->fd, 'access faild.');
                return false;
            }
            $uid = $request->get['uid'];
            $token = $request->get['token'];
            // 校验token是否正确,无效关闭连接
            if (md5(md5($uid) . $this->key) != $token) {
                $this->close($request->fd, 'token invalidate.');
                return false;
            }
            return true;
        }

        /**
         * @param $fd
         * @param $message
         * 关闭$fd的连接，并删除该用户的映射
         */
        public function close($fd, $message = '')
        {
            // 关闭连接
            $this->_serv->close($fd);
            // 删除映射关系
            if ($uid = array_search($fd, $this->user2fd)) {
                unset($this->user2fd[$uid]);
            }
        }

        public function alertTip($fd, $data)
        {
            // 推送目标用户的uid非真或者该uid尚无保存的映射fd，关闭连接
            if (empty($data['toUid']) || !array_key_exists($data['toUid'], $this->user2fd)) {
                $this->close($fd);
                return false;
            }
            $this->push($this->user2fd[$data['toUid']], ['event' => $data['event'], 'msg' => '收到一条新的回复.']);
        }
        /**
         * @param $fd
         * @param $message
         */
        public function push($fd, $message)
        {
            if (!is_array($message)) {
                $message = [$message];
            }
            $message = json_encode($message);
            // push失败，close
            if ($this->_serv->push($fd, $message) == false) {
                $this->close($fd);
            }
        }

        public function start()
        {
            $this->_serv->start();
        }
    }

    $server = new CommentServer;
    $server->start();
    ?>`

    客户端：
    <div>
        发送内容：<textarea name="content" id="content" cols="30" rows="10"></textarea><br>
        发送给谁：<input type="text" name="toUid" value="" id="toUid"><br>
        <button onclick="send();">发送</button>
    </div>

    <script>
        var ws = new WebSocket("ws://127.0.0.1:9501?uid=<?php echo $uid ?>&token=<?php echo $token; ?>");
        ws.onopen = function(event) {
        };
        ws.onmessage = function(event) {
            var data = event.data;
            data = eval("("+data+")");
            if (data.event == 'alertTip') {
                alert(data.msg);
            }
        };
        ws.onclose = function(event) {
            console.log('Client has closed.\n');
        };

        function send() {
            var obj = document.getElementById('content');
            var content = obj.value;
            var toUid = document.getElementById('toUid').value;
            ws.send('{"event":"alertTip", "toUid": '+toUid+'}');
        }
    </script>


  19.
