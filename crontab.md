Crontab：
service crond restart
service crond status

crontab -l #列出定时任务
crontab -e #打开
  */1 * * * * date >> /tmp/date.tmp #每分钟

  #说明：*(分钟0-59) *(小时0-23) *(日期1-31) *(月份1-12) *(星期0-7)

30   21    *      *    *     #每晚21:30do
45   4  1,10,22   *    *     #每月1 10 22的4:45do
45   4  1-10      *    *     #每月1-10号do
*/2  *     *      *    *     #每2分钟do
1-59/2 *   *      *    *     #每2分钟（在奇数）do
0  23-7/1  *      *    *     #晚上11-早上7点 每小时do
0,30 18-23 *      *    *     #每天18-23 每隔30分钟do
0-59/30 18-23 *   *    *     #每天18-23 每隔30分钟do

crontab -e -u user           #用户添加定时
tail -f /var/log/cron        #执行过的定时任务 (日志信息)

#配置文件 /etc/crontab
#系统任务 /etc/crontab/cron.d/* 都被载入到系统任务中
#用户任务 /var/spool/cron/root  (crontab -e)
#发送的日志 /var/spool/mail      能被root查看


#常见错误
#环境变量不会被加载
#第三个和第五个域 执行的是或操作

59 1 1-7 4 0 /root/a.sh  #表示4月的1-7号 1:59分 或是 4月的星期日1:59分

59 1 1-7 4 * test `date +\%w` = 0 && /root/a.sh #表示4月第一个星期日 1:59分

#计划任务中的% 要是变成\%

#每半分钟执行任务

*/1 * * * * date >> /tmp/date.conf && sleep 30s && date >> /tmp/date.conf
