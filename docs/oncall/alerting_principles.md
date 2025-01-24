---
cover: assets/img/covers/alerting\_principles.png description:私たちは、単純な原則に基づいてアラートを取得する方法を管理します。アラートは、人間がアクションを実行することを必要とするものです。それ以外は通知であり、それは私たちがコントロールできないものであり、私たちがそれらに影響を与える行動を取ることができないものです。通知は役に立つが、全ての通知が人を叩き起こすべきものではありません。
---
私たちは、単純な原則に基づいて、どのようにアラートを受けるかを管理します。**アラートは、人間がアクション**を実行する必要があるものです。それ以外は通知であり、それは私たちがコントロールできないものであり、私たちがそれらに影響を与える行動を取ることができないものです。通知は役に立つが、全ての通知が人を叩き起こすべきものではありません。

## アラート優先度

!!! warning "High Priority Alerts"夜中に人間を起こすものは**即座に人間が実行可能な**でなければなりません。そのようなものがない場合は、その時点でページが表示されないようにアラートを調整する必要があります。

| 優先度 | アラート | 応答 |
| -------- | ------ | -------- |
| 高 | PagerDuty Alert 24/7/365. |  **即時の人間の行動**必要. |
| 中 | 優先度の高いPagerDuty Alert **営業時間のみ**. | 24時間以内の人間の行動が必要 |
| 低 | 優先度の低いPagerDuty Alert 24/7/365 | ある時点での人間の行動が必要 |
| 通知 | 抑制されたPagerDuty イベント | 応答不要。情報提供のみ。 |

新しいアラート/通知を設定する場合は、上記のチャートを考慮して、ユーザーにアラートを送信する方法を確認します。たとえば、すぐに対応する必要がない場合は、優先度の高い新しいアラートを作成しないように注意してください。

## 優先度の例

#### "プロダクションサービスがリクエストの75%で失敗し、自動化が解決できません。"_
これは、**High** priority ページになり、解決するためにすぐに人間のアクションが必要になります。

![緊急性が高い](../assets/img/screenshots/high_urgency.png)

#### "プロダクション・サーバーのディスク領域がいっぱいになり、48時間でいっぱいになると予想されます。ログのローテーションでは解決できません。"
これは**Medium** priorityとなり、すぐに人間の行動が必要になりますが、直ちに今すぐというほどではありません。

![中程度の緊急度](../assets/img/screenshots/high_business_hours.png)

#### "SSL証明書の有効期限は1週間です。"
これは、**Low** priorityになり、いずれ人間の行動が必要になりますが緊急性は低いです。

![低緊急度](../assets/img/screenshots/low_urgency.png)

#### "デプロイが成功しました。"
これは**Notification**となり、抑制されたイベントとして送信されます。これは、インシデントが発生した場合に有用な情報を提供しますが、人間に通知する必要はありません。

![通知](../assets/img/screenshots/suppressed.png)


## アラートの内容

アラートには、問題を迅速に特定し、可能性のある修復手順を特定するのに十分な有用な情報が含まれていることを確認する必要があります。一般的なタイトルまたは説明を含むアラートは有用ではなく、混乱を引き起こす可能性があります。私たちは、すべてのアラートが従うべき、アラートの内容に関する一連のガイドラインを持っています。

#### タイトル/要約をわかりやすく簡潔にする。
  * <span class="icon bad"></span> 警告:何か問題があった。
  * <span class="icon good"></span> ディスクは `prod-web-loadbalancer-af5462ce` で 80% フルです。

#### アラートをトリガーしたメトリクスは、本体のどこかに含まれていることを確認してください。
  * <span class="icon bad"></span> ディスクのディスク容量がいっぱいになっています。
  * <span class="icon good"></span> `avg(last_1h):max:system.disk.in_use{env:prod-web-loadbalancer} by {host} > 0.8`

#### ボディには、実際の問題が何であるか、そしてなぜ問題なのかについての説明も含めるべきです。
  * <span class="icon bad"></span> ディスクがいっぱいです。
  * <span class="icon good"></span> このホストのディスクの容量は80% です。いっぱいになりすぎると、新しいファイルが作成されず、現在のファイルが書き込まれないため、システムが不安定になることがあります。

#### 問題を解決するための明確な手順を提供するか、ランブックにリンクします。これらのいずれもないアラートは役に立たない。
  * <span class="icon bad"></span> 何かを削除して修正します。
  * <span class="icon good"></span> ディスクスペースの問題を特定して解決するには、次のランブックを参照してください。https://example.com/run book/disk.さらに、ログローテーションのしきい値でこの問題が再発しないようにできるかどうかを調べる必要があります。以下の実行ブックには、必要な手順があります。https://example.com/run book/log-rotate


## アラートのテスト

!!! info "Testing is Critical"テストされていないアラートは、アラートをまったく持たないことと同等です。時間が来たときにそれがまたアラートを出すか確証を持てません。アラートが実際に機能するかどうかをテストすることは、適切なサービスの健全性にとって重要であり、リリースの計画/デプロイメントの取り組みに含める必要があります。

新しいアラートと変更されたアラートをすべてテストするようにしてください。これは通常、新しいサービスの[Failure Friday](https://www.pagerduty.com/blog/failure-friday-at-pagerduty/) の一部として扱われますが、より迅速に必要な場合は手動でテストする必要があります。テストすべき事項:

* しきい値が適切に設定されていることを確認します。私たちは必要以上のアラートを望んでいません。
* "No Data"条件(該当する場合)のアラートを取得するテスト。一般的に、データを受信しないことは、しきい値を破ることと同じです。
* メトリクスが正常に戻ったときにアラートが自動的に解決するかどうかをテストします。
