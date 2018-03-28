路由重写.htaccess文件(隐藏index.php)：
# use mod_rewrite for pretty URL support
RewriteEngine on
# if a directory or a file exists, use the request directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# otherwise forward the request to index.php
RewriteRule . index.php

advanced目录下的路由.htaccess(重定向)：
# prevent directory listings
Options -Indexes
# follow symbolic links
Options FollowSymlinks
RewriteEngine on
RewriteCond %{REQUEST_URI} ^/admin/$
RewriteRule ^(admin)/$ /$1 [R=301,L]
RewriteCond %{REQUEST_URI} ^/admin
RewriteRule ^admin(/.+)?$ /backend/web/$1 [L,PT]
RewriteCond %{REQUEST_URI} ^.*$
RewriteRule ^(.*)$ /frontend/web/$1



url美化：
'urlManager' => [
        'enablePrettyUrl' => true,  //是否启用严格解析，如启用严格解析，要求当前请求应至少匹配1个路由规则
	      'enableStrictParsing' => false, //是否在URL中显示入口脚本。是对美化功能的进一步补充。
        'showScriptName' => false,
        //'suffix'=>'.html',  //结尾
        'rules' => [
    				"<controller:\w+>/<action:\w+>"=>"<controller>/<action>"
        ],
],

session + cookie :
  Yii::$app->session->set('name',$data);
  Yii::$app->session['name'];
  Yii::$app->session->destroy();

日志配置位置：components->log->targets：
日志位置：frontend/runtime/logs/app.log
  'targets' => [
      [
          'class' => 'yii\log\DbTarget',
          'levels' => ['error', 'warning'],
          'logVars' => ['_SERVER'], //记录_变量类型
      ],
      [
          'class' => 'yii\log\EmailTarget',
          'levels' => ['error'],
          'categories' => ['yii\db\*',], //记录类型
          'except'=>['yii\db\Command::query'] //不记录类型
          'message' => [
             'from' => ['log@example.com'],
             'to' => ['admin@example.com', 'developer@example.com'],
             'subject' => 'Database errors at example.com',
          ],
      ],
  ],
  手动设置：
      $filename = 'test.log';
      $msg = '测试测试测试数据';
      $log = new yii\log\FileTarget;
      //$log->logFile = Yii::$app->getRuntimePath() . '/logs/'.$filename;
      $log->logFile = Yii::getAlias('@runtime')."/logs/".$filename;
      $log->messages[] = [$msg,1,'application',microtime(true)];
      $log->export();

      // Yii::warning('a message waring something.....');
      // Yii::error('a message error something.....');


组件；属性；事件；行为；
<!-- ###############
#                  #
#                  #
#                  #
#                  #
#                  #
#                  #
################ -->


#配置：

//初始一个对象配置
$config = [
  'class'=>'yii\db\Connetion',
  'dsn'=>'mysql:host=127.0.0.1;dbname=demo',
  'username'=>'root',
  'password'=>'',
  'charset'=>'utf8',
];
$db = Yii::createObject($config);

//重设已存在对象
Yii::configure($object,$config);

yii\web\Applicatin::component 配置：
defined('YII_ENV') or define('YII_ENV','dev');
if(YII_ENV){
   $config['bootstrap'][] = 'debug'
   //....
}
return $config;

别名：
Yii::setAlias('@foo', '/path/to/foo');


#数据访问DAO：

\yii\db\Connectionp;配置
$db = \Yii::$app->db; 或 new \yii\db\Connection($conf);

$command = $db ->createCommand($sql);
  getRawSql();
  bindValues(':id',$id);
  bindValues([':id'=>$id]);
  bindParam([':id'=>$id]);
  batchInsert('ts_name',['name','vip_num'],array(['xxx1',1],['xxx2',2],['xxx3',3]));
  queryAll();
  queryOne();
  execute();

$transaction = $db->beginTransaction;
try{
    $transaction->commit();
}catch(\Exception $e) {
    $transaction->rollBack();
}


 查询构建器：yii\db\Query;

 $rows = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->limit(10)
    ->all();

    all()：将返回一个由行组成的数组，每一行是一个由名称和值构成的关联数组（译者注：省略键的数组称为索引数组）。
    one()：返回结果集的第一行。
    column()：返回结果集的第一列。
    scalar()：返回结果集的第一行第一列的标量值。
    exists()：返回一个表示该查询是否包结果集的值。
    count()：返回 COUNT 查询的结果。
    average()
    batch()
    createCommand()
    distinct()
    from()
    groupBy()
    having()
    innerJoin()
    join()
    max()
    min()
    one()
    params()
    select()
    union()
    where()

 AR模型：yii\db\ActiveQuery;
    yii\db\ActiveRecord::find() 返回查询构建器
    findOne()
    findAll()
    findOne(1)->updateCounters(['count' => 1]);  // UPDATE `post` SET `count` = `count` + 1 WHERE `id` = 1;

    脏属性：
    yii\db\ActiveRecord::save() 保存 AR 实例时，只有 脏属性 被保存。 (===判读)
    ->getDirtyAttributes()  //脏属性
    ->getOldAttributes()    //原属性

    updateAll()
    updateAllCounters(['age'=>1]);  //UPDATE `customer` SET `age` = `age` + 1;

    ->delete()
    ->deleteAll()

  AR周期：
    实例化：
    init()

    查询：
    init()
    afterFind()

    保存数据：
    save()
    beforeValidate()
    afterValidate()
    beforeSave()
    afterSave()

    删除数据delete()：
    beforeDelete()
    afterDelete()

    注(以下方法直接执行sql 不会触发生命周期)：
    updateAll() deleteAll() updateCounters() updateAllCounters()

    try{

      }catch(\Exception $e){
        $transaction ->rollBack();
      }catch(\Throwable $e){
        $transaction ->rollBack();
      }

  乐观锁：
   了解一下yii\db\ActiveRecord::optimisticLock()


  关联数据：
    public function getOrders()
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }
    ①关联数据后可用with查询构建器 但查询字段要带上关联条件：
    ->with(['orders'=>function($query){
         $query ->select('customer_id,xxx')
      }])

    ②直接使用属性：
    $obj = Modle::findOne(1);
    $obj->orders;         //返回AR实例
    $obj->getOrders();    //返回条件构建 AQ模型(查询构建器)

    with的延迟加载(减少sql)：
    // SELECT * FROM `customer` LIMIT 100;
    // SELECT * FROM `orders` WHERE `customer_id` IN (...)
    $customers = Customer::find()
        ->with('orders')
        ->limit(100)
        ->all();

    foreach ($customers as $customer) {
        // 没有任何的 SQL 执行
        $orders = $customer->orders;
    }

    //选择额外的字段  (下面的方法都可以写到AR模型中  使用__get __set 方法去构建属性访问)
    $rooms = Room::find()
    ->select([
        '{{room}}.*', // select all columns
        '([[length]] * [[width]] * [[height]]) AS volume', // 计算体积
    ])
    ->orderBy('volume DESC') // 使用排序
    ->all();

    //AR类构建 然后直接访问volume属性
    private $_volume;
    public function setVolume($volume)
    {
        $this->_volume = (float) $volume;
    }
    public function getVolume()
    {
        if (empty($this->length) || empty($this->width) || empty($this->height)) {
            return null;
        }

        if ($this->_volume === null) {
            $this->setVolume(
                $this->length * $this->width * $this->height
            );
        }

        return $this->_volume;
    }

    $customers = Customer::find()
        ->select([
            '{{customer}}.*', // select customer 表所有的字段
            'COUNT({{order}}.id) AS ordersCount' // 计算订单总数
        ])
        ->joinWith('orders') // 连接表
        ->groupBy('{{customer}}.id') // 分组查询，以确保聚合函数生效
        ->all();
