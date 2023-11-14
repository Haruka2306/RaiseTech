# 第6回課題
## 最後にAWSを利用した日の記録をCloudTrailのイベントから探し出す
ConsoleLogin：AWS Management Consoleにブラウザ経由でログインした時の記録
![ct_console-login](/images/lecture06/ct_console-login.png)
* イベント名：ConsoleLogin
* イベント時間：November 12, 2023, 22:07:31 (UTC+09:00)
* イベントソース：signin.amazonaws.com
* イベントID：a7b95e0d-e8ee-4bda-85d0-707bbefe9e7d

## CloudWatchを使用してALBアラームとアクションを設定
* AmazonSNS
![](/images/lecture06/cw_AmazonSNS.png)
* CloudWatchのアクション設定
![](/images/lecture06/cw_action.png)
### Action: Alarm
* ターゲットグループのヘルスステータス：Unhealthy
![cw_unhealthy-tg](/images/lecture06/cw_unhealthy-tg.png)
* CloudWatchのアクション：アラーム状態
![cw_alarm-UnHealthyHostCount](/images/lecture06/cw_alarm-UnHealthyHostCount.png)
* メール通知
![](/images/lecture06/cw_mail-alarm.png)
### Action: OK
* ターゲットグループのヘルスステータス：Healthy
![](/images/lecture06/cw_healthy-tg.png)
* CloudWatchのアクション：OK
![](/images/lecture06/cw_ok-UnHealthyHostCount.png)
* メール通知
![](/images/lecture06/cw_mail-ok.png)

## AWSのコスト管理
* AWS利用料の見積<br>
[リソース内容の見積](https://calculator.aws/#/estimate?id=aa0c54fbffaa6dd9efb471bc0f63339613f3032d)
* 現在の利用料金
![](/images/lecture06/cost_Nov.png)
* 先月の利用料金及びEC2の料金
![](/images/lecture06/cost_Oct.png)
![](/images/lecture06/cost_ec2.png)


