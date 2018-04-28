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
没安装则安装admin(rbac 可视化管理)
composer require mdmsoft/yii2-admin "2.x-dev"

# PHP 7.1 进行开发，而我本地是 PHP 7.0, 于是悲剧发生了
删除 composer.lock 文件，重新执行 composer install，这样就能重新生成 composer.lock 文件了。

#依赖composer.lock文件 可以通过 composer update 进行更新


