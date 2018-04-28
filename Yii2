<?php

/**
 * yii事务的方法
 * @var [type]
 */
$connection = Yii::$app->db;
$connection = Modle::getDb();
$transaction = Yii::$app->db->beginTransaction();
try{
  #逻辑处理#
  $transaction ->commit();
}catch(\Exception $e){
  $transaction ->rollBack();
}


/**
 * 原生sql查询
 * @var string
 */
$sql = 'select * from table where 1=1 ';
$sql .= ' and id in (1,3,5)';
$sql .= ' order by id desc';
$sql .= ' limit 3,5';
$command = Yii::$app ->db->createCommand($sql);
$rows = $command ->queryAll();
$num = $rows ->count();#条数
$data = $rows ->readAll();#数据


/**
 * 表单模型数据验证规则
 * @var Modle
 */

$modle = new ModleForm;
if($modle ->load(Yii::$app ->request->post() && $modle ->validate()) {
  Yii::$app->user->login($this->getUser(), 3600 * 24 * 30);
}

class ModleForm extends \yii\base\Modle
{
  #rules 方法做规则验证
  public function rules() {
    return [
      [['username', 'password'], 'required'],
      ['rememberMe', 'boolean'],
      ['password', 'validatePassword'],
    ];
  }
  #方法规则验证
  public function validatePassword($attribute, $params) {
      return Yii::$app->security->validatePassword($password, $userModle->password_hash);
  }
}



/**
 * 两表关联，数据查询
 * @var [type]
 */
$models = Post::find() ->where(['>','id',6]) ->all();
foreach ($models as $model) {
  echo $modle ->tital;
  foreach($model ->Commnents->contents as $v){
    echo $v;
  }
  echo $model ->Author->name;
}
class Post extends \yii\db\ActiveRecord
{
  #关联评论表
  public function getComments() {
      return $this->hasMany(Comment::className(), ['post_id' => 'id']);
  }
  #关联作者表
  public function getAuthor() {
      return $this->hasOne(Adminuser::className(), ['id' => 'author_id']);
  }
}

/**
 * with 的关联查询例子  Modle中要有hasOne 或hasMany 相关联
 * @var [type]
 */
eg1：
$result = self::find()
    ->select('id, business_id')
    ->where(['id' => $id])
    ->with(['bussiness' => function ($query) {
        $query->select('id, business_name, company_province');
       }])
    ->asArray()
    ->one();
#打印结果
"id": "13",
"business_id": "13",
"bussiness": {
    "id": "13",
    "business_name": "大霞",
    "company_province": "2"
}

eg2：
$result = self::find()
           ->select('id, business_id')
           ->where(['id' => $id])
           ->with(['bussiness' => function ($query) {
               $query->select('id, business_name, company_province')
               ->with(['province' => function ($query) {
                       $query->select('id, name');
                      }]);
             }])
           ->asArray()
           ->one();
#打印结果
"id": "13",
"business_id": "13",
"bussiness": {
    "id": "13",
    "business_name": "大霞",
    "company_province": "2",
    "province": {
        "id": "2",
        "name": "北京市"
    }
}
eg3：
$result = self::find()->
    select('id, business_id')->
    where(['id' => $id])->
    with('bussiness.province')-> #把关联表的关联表 查询出来
    asArray()->
    one();
#打印结果
"id": "13",
"business_id": "13",
"bussiness": {
    "id": "13",
    "business_name": "大霞",
    "mobile": "xxxxxx",
    "weixin": "xxxxx",
    "province": {
        "id": "2",
        "name": "北京市",
        "level": "2",
        "usetype": "0",
        "upid": "3225",
        "displayorder": "0"
    }
}


/**
 * AR类的事务重写周期示例
 */
class Modle extends \yii\db\ActiveRecord
{
   public function beforeSave($insert) {
     if(parent::beforeSave($inset)) {
       if($insert) {
         #有插入数据时候的事务逻辑
       }else{
         #更新数据插入的事务逻辑
       }
     }else{
       return false;
     }
   }
}

/**
 * yii2框架查询数据必须带id(因为在要update sql是以id去记录表单的)
 * 并且含有脏数据必须传入true值不让其验证
 */

 $modle = new BankAccount;
 $modle->attributes = array(
     "shop_id"=>$shop_id,
     "identify"=>$identify,
     "phone"=>$phone,
     "create_time"=>$create_time,
 );
$modle->save(true,array("shop_id","identify","phone","create_time"));
**



//console日志文件配置
'components' => [
    'log' => [
        'targets' => [
            [
                'class' => 'yii\log\FileTarget',
                'levels' => ['error', 'warning'],
            ],
        ],
    ],
],
日志位置:
console/runtime/logs/app.log

/**
 * yii2 AR类删除数据
 * @var [type]
 */
User::deleteAll('age>:age AND sex=:sex', [':age' => '20', ':sex' => '1']);
User::findOne($id)->delete();
User::find()->where(['id' => $id])->one()->delete();
