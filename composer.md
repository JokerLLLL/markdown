#Yii2-admin安装
linux:
php composer.phar require mdmsoft/yii2-admin "~2.0"
php composer.phar update
windows:
composer require mdmsoft/yii2-admin "~2.0"
composer update

#yii2的composer更新
composer update yiisoft/yii2-composer

#composer自更新
composer self-update composer update

#版本更新
composer self-update --no-plugins

#版本回滚
composer self-update --rollback


#安装设置中国镜像源：
composer config -g repo.packagist composer https://packagist.phpcomposer.com

#查看详情
composer -v

#yii2 后台模板(-vvv 显示安装过程)
composer require dmstr/yii2-adminlte-asset "2.*" -vvv
