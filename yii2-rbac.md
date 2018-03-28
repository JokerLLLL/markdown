后台管理权限：
composer下载：
dmstr/yii2-adminlte-asset
mdmsoft/yii2-admin

配置后台的config的main.php

    //模块
    'modules' => [
        'admin' => [
            'class' => 'mdm\admin\Module',
            // 'layout' => 'left-menu',//yii2-admin的导航菜单
        ]
    ],
    //rbac组件
    components =>[
    	'authManager'=> [
    		'class' => 'yii\rbac\DbManager',
    	]
    ]
    //acf配置(权限高于rbac)
    'as access' => [
		'class' => 'mdm\admin\components\AccessControl',
		'allowActions' => [
			'site/*',
			'admin/*',
		]
	],
