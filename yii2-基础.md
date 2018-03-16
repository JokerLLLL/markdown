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
        'suffix'=>'.html',  //结尾
        'rules' => [
    				"<controller:\w+>/<action:\w+>"=>"<controller>/<action>"
        ],
],

session + cookie :
  Yii::$app->session->set('name',$data);
  Yii::$app->session['name'];
  Yii::$app->session->destroy();
