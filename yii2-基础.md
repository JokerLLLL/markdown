路由重写.htaccess文件：
# use mod_rewrite for pretty URL support
RewriteEngine on
# if a directory or a file exists, use the request directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# otherwise forward the request to index.php
RewriteRule . index.php

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
####################
#                  #
#                  #
#                  #
#                  #
#                  #
#                  #
####################


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
  getSql();
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

 ActiveRecord  QueryBuilder  modle类和查询类

 $rows = (new \yii\db\Query())
    ->select(['id', 'email'])
    ->from('user')
    ->where(['last_name' => 'Smith'])
    ->limit(10)
    ->all();