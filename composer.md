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
