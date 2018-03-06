//wamp msql设置：
01.打开 Mysql控制台
02.查看 general_log 状态：SHOW VARIABLES LIKE '%general_log%';确认 log记录档在何处。
03.打开 general_log 监看：SET GLOBAL general_log = 'ON';执行网站 ，看 log记录档，执行了那些SQL语句。
04.关闭 general_log 取消监看:SET GLOBAL general_log = 'OFF';
