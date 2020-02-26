+++
<<<<<<< HEAD
title = "スポットインスタンスを利用するシステムの堅牢化"
weight = 155
+++

### スポットインスタンス中断への対処
あるアベイラビリティゾーンのあるインスタンスタイプにおいてキャパシティが不足するとき、EC2サービスはキャパシティを回復させる必要があります。このとき、対象のアベイラビリティゾーンにおけるインスタンスタイプにスポットインスタンスが起動していれば、EC2サービスは該当するスポットインスタンスに2分前の中断通知を送付し、スポットインスタンスを中断することでEC2キャパシティの回復に充てます。2分前の中断通知はインスタンスメタデータサービス、およびCloudWatch Eventsから受け取ることができます。中断通知の詳細は[こちら](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices)のドキュメントを参照してください。
ここでは、中断通知イベントをCloudWatch Events経由で受信し、それをトリガーにLambda関数を実行する仕組みを構築します。中断通知イベントは`EC2 Spot Instance Interruption Warning`という名称です。Lambda関数には、中断対象とマークされたスポットインスタンスをAuto Scalingグループからデタッチ([DetachInstances](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html))する動作を記述します。デタッチの詳細はEC2 Auto Scalingユーザーガイドの[Auto Scaling グループからの EC2 インスタンスのデタッチ](https://docs.aws.amazon.com/ja_jp/autoscaling/ec2/userguide/detach-instance-asg.html)を参照してください。

  1. まず検討すべきポイントは、スポットインスタンスをデタッチするタイミングでAuto Scalingグループの希望容量を1台減らすのか、それとも台数変更なしにするのか、という点です。もし希望容量を変更しない場合、デタッチの直後にAuto Scalingサービスが新しいインスタンスを自動的に起動します。

  1. 次に、今回のワークショップのようにAuto Scalingグループがロードバランサー配下に登録される場合、インスタンスがAuto Scalingグループからデタッチされるとき、ロードバランサーからも登録解除(deregister)されます。このとき、ロードバランサー(あるいはターゲットグループ)にConnection Draining(あるいは登録解除の遅延とも呼ばれます)が設定されていれば、Auto Scalingは処理中のコネクションが終了するまでデタッチを待ちます。今回、このタイムアウト値は中断通知の長さに合わせて120秒を設定しています。

EC2 Auto Scalingのデタッチ動作については[こちらのドキュメント](https://docs.aws.amazon.com/autoscaling/ec2/userguide/detach-instance-asg.html)を参照してください。

<<<<<<< HEAD
<<<<<<< HEAD
ここでは、Lambda関数とCloudWatch Eventsを作成し、それぞれを関連付けるCloudFormationテンプレートを用いてこの仕組みを構築します。

  1. CloudFormationテンプレートの内容を確認し、次のコマンドでスタックを作成します。
=======
このワークショップでは、Lambda関数とClowdWatch Eventsを作成し、それぞれを関連付けるCloudFormationテンプレートを準備しました。

  1. CloudFormationテンプレートの内容を確認します。次のコマンドでスタックをデプロイします。
>>>>>>> update spot_resilience
=======
ここでは、Lambda関数とClowdWatch Eventsを作成し、それぞれを関連付けるCloudFormationテンプレートを用いてこの仕組みを構築します。

  1. CloudFormationテンプレートの内容を確認し、次のコマンドでスタックを作成します。
>>>>>>> Modifieng all other feedbacks from the dry-run session
=======
title = "Resilience with Spot Instances"
weight = 155
+++

### Handling Spot Interruptions
When EC2 needs the capacity back in a specific capacity pool (a combination of an instance type in an Availability Zone) it could start interrupting the Spot Instances that are running in that AZ, by sending a 2 minute interruption notification, and then terminating the instance. The 2 minute interruption notification is delivered via [EC2 instance meta-data](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-interruptions.html#spot-instance-termination-notices) as well as CloudWatch Events. 

Let's deploy a Lambda function that would catch the CloudWatch event for `EC2 Spot Instance Interruption Warning` and automatically detach the soon-to-be-terminated instance from the EC2 Auto Scaling group. 
By calling the [DetachInstances](https://docs.aws.amazon.com/autoscaling/ec2/APIReference/API_DetachInstances.html) API you achieve two things:

  1. You can specify on the API call whether to keep the current desired capacity on the Auto Scaling group, or decrement it by the number of instances being detached. By keeping the same desired capacity, Auto Scaling will immediately launch a replacement instance.

  1. If the Auto Scaling group has a Load Balancer or a Target Group attached (as we have in this workshop), the instance is deregistered from it. Also, if connection draining is enabled for your Load Balancer (or Target Group), the Auto Scaling group waits for in-flight requests to complete (up to the configured timeout, which we have set up to 120 sec).

    You can learn more about the Detaching EC2 Instances from an Auto Scaling group [here](https://docs.aws.amazon.com/autoscaling/ec2/userguide/detach-instance-asg.html).


To save time, we will use a CloudFormation template to deploy the Lambda Function that will handle EC2 Spot interruptions, and the CloudWatch event rule to catch the Spot Interruption notifications, and subscribe the Lambda Function to it. 

  1. Take some time to review the CloudFormation template and understand what will be launched. Then, execute the following command to deploy the template: 
>>>>>>> updating

    ```
    aws cloudformation deploy --template-file spot-interruption-handler.yaml --stack-name spotinterruptionhandler --capabilities CAPABILITY_IAM
    ```

<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
  1. スタックの作成が完了したらば[Lambdaコンソール](https://console.aws.amazon.com/lambda/home)を開き、新規作成された関数名をクリックします。作成完了の目安は2分以内を想定しています。

 
 1. 作成されたLambda関数の内容を確認します。Cloud9のインラインコードエディタを活用してください。

これでスポットインスタンスの中断通知を受け取り、そのインスタンスを自動的にAuto Scalingグループからデタッチさせることができるようになりました。中断そのものをシミュレーションすることはできませんが、ここではLambda関数のテスト機能を使い、処理が正しく実行されるかを確認します。

  1. Lambdaコンソール右上にあるドロップダウンメニューから「テストイベントの選択」をクリックしてドロップダウンメニューを表示させ、「テストイベントの設定」を選択します。グレーアウトされている場合にもそのままクリックし、ドロップダウンメニューを表示できます。
  1. 「テストイベントの設定」ダイアログボックスではイベント名に任意の名前を設定し(TestSpotInterruptionなど), 次のjsonを入力します。
=======
  1. When the CloudFormation deployment completes (under 2 minutes), open the [AWS Lambda console](https://console.aws.amazon.com/lambda/home) and click on the newly deployed Function name.
 
 1. Feel free to examine the code in the Inline code editor.
=======
  1. スタックの作成が完了したらば[Lambdaコンソール](https://console.aws.amazon.com/lambda/home)を開き、新規作成された関数名をクリックします。作成完了の目安は2分以内を想定しています。
>>>>>>> update spot_resilience

 
 1. 作成されたLambda関数の内容を確認します。Cloud9のインラインコードエディタを活用してください。

これでスポットインスタンスの中断通知を受け取り、そのインスタンスを自動的にAuto Scalingグループからデタッチさせることができるようになりました。中断そのものをシミュレーションすることはできませんが、ここではLambda関数のテスト機能を使い、処理が正しく実行されるかを確認します。

<<<<<<< HEAD
  1. In the top right corner of the AWS Lambda console, click the dropdown menu **Select a test event** -> **Configure test events**
  1. With **Create a new test event** selected, provide an Event name (i.e TestSpotInterruption). In the event text box, paste the following:
>>>>>>> updating
=======
  1. Labmdaコンソール右上にあるドロップダウンメニューから「テストイベントの選択」をクリックしてドロップダウンメニューを表示させ、「テストイベントの設定」を選択します。グレーアウトされている場合にもそのままクリックし、ドロップダウンメニューを表示できます。
  1. 「テストイベントの設定」ダイアログボックスではイベント名に任意の名前を設定し(TestSpotInterruptionなど), 次のjsonを入力します。
>>>>>>> update spot_resilience
=======
  1. When the CloudFormation deployment completes (under 2 minutes), open the [AWS Lambda console](https://console.aws.amazon.com/lambda/home) and click on the newly deployed Function name.
 
 1. Feel free to examine the code in the Inline code editor.


Now our infrastructure is ready to respond to Spot Interruptions by detaching Spot Instances from the Auto Scaling group when they receive a Spot interruption notification. We can't simulate an EC2 Spot Interruption, but we can invoke the Lambda Function with a simulation of a CloudWatch event for an EC2 Spot Instance Interruption Warning, and see the result.

  1. In the top right corner of the AWS Lambda console, click the dropdown menu **Select a test event** -> **Configure test events**
  1. With **Create a new test event** selected, provide an Event name (i.e TestSpotInterruption). In the event text box, paste the following:
>>>>>>> updating
  
    ```json
    {
      "version": "0",
      "id": "92453ca5-5b23-219e-8003-ab7283ca016b",
      "detail-type": "EC2 Spot Instance Interruption Warning",
      "source": "aws.ec2",
      "account": "123456789012",
      "time": "2019-11-05T11:03:11Z",
      "region": "eu-west-1",
      "resources": [
        "arn:aws:ec2:eu-west-1b:instance/<instance-id>"
      ],
    "detail": {
      "instance-id": "<instance-id>",
      "instance-action": "terminate"
      }
    }
    ```
    
<<<<<<< HEAD
<<<<<<< HEAD
<<<<<<< HEAD
=======
>>>>>>> update spot_resilience
  1. このjsonに2箇所存在する**"\<instance-id>"**を、作成したAuto Scalingグループ内の任意の1台のインスタンスIDで置き換えます。インスタンスIDは[EC2 Auto Scalingコンソール](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:view=details)のインスタンスタブから確認できます。

  1. 作成を押します。
  1. 「テストイベントの選択」で先ほど選択したテストイベントを選択し、「テスト」ボタンをクリックします。
  1. 画面上部に実行結果の成功が表示されます。「詳細」をクリックして内容を確認します。
  1. [EC2 Auto Scalingコンソール](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:view=details)から対象Auto Scalingグループの「アクティビティ履歴」タブを開き、成功ログとして"Instance i-01234567890123456 belongs to AutoScaling Group runningAmazonEC2WorkloadsAtScale. Detaching instance..."のようなメッセージが出ていることを確認します。またその直後のログに新しいインスタンスの起動が記録されているのを確認します。
  1. [EC2 ELBターゲットグループコンソール](https://console.aws.amazon.com/ec2/v2/home?1#TargetGroups:sort=targetGroupName)から今回のターゲットグループを選択し、ターゲットタブを開きます。該当インスタンスがdrainingのステータスになっていることを確認します。
<<<<<<< HEAD

ここまでの手順で、スポットインスタンスの中断通知を受けたタイミングでシームレスに新しいスポットインスタンスを起動し、スポットインスタンスが終了する前に安全に入れ替える仕組みを構築することができました。

{{% notice warning %}} 
実際に中断が発生する場合、Auto ScalingグループからデタッチしたインスタンスはEC2スポットサービスにより自動的に終了(Terminate)されます。今回は単に中断イベントをシミュレーションしただけであったためインスタンスは終了されません。デタッチしたインスタンスを忘れずに終了させてください。
{{% /notice %}}


### スポットインスタンスを利用するシステムの堅牢化

これまでスポットインスタンスの起動に際して、アベイラビリティゾーンごとに最も安い順に4種類のインスタンスタイプを9種類のインスタンスタイプから選択できるように構成してきました。ここからさらに、使用するインスタンスタイプとアベイラビリティゾーンの組み合わせを増やすことで、特定のスポットキャパシティプール(アベイラビリティゾーンとインスタンスタイプの組み合わせ)での中断の影響を緩和していくことができます。

#### チャレンジしてみましょう

今回作成したスポットインスタンスによる音楽ストリーミングアプリケーションをより堅牢にするために、どのような構成の改善を検討したらよいでしょうか。

{{%expand "考え方の一例" %}}

1. アベイラビリティゾーンを追加します。今の設定では2つのアベイラビリティゾーンを定義しています。ここにもうひとつアベイラビリティゾーンを追加することで、ある1箇所のスポットキャパシティプールでキャパシティが不足し、スポットインスタンスの中断が発生するとき、引き続きワークロードを受けることのできる可能性を高めることができます。

2. インスタンスタイプを追加します。さらに増やせば増やすほど中断発生する確率を下げることができます。今回は9種類のインスタンスタイプを定義していますが、さらに追加できるものがあるでしょうか。

3. スポットインスタンスを起動する対象となるスポットキャパシティプールを増やします。今回は9種類のインスタンスタイプのうち、4種類を上限に指定しました。これを増やすことで使用できるインスタンスタイプの多様性を高めることができ、中断発生確率の低下につながります。
{{% /expand %}}

#### チャレンジしてみましょう
スポットインスタンスの配分戦略には、今回選択したlowest-priceの他にどのようなものがあるでしょうか。今回のワークロードに最も適した配分戦略はどれになるでしょうか。
ヒント：新しい配分戦略についての[記事] (https://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/)を参照してください。

{{%expand "Click here for the answer" %}}
今回はいわゆるステートレスなアプリケーションを構築しました。これよりも中断を許容しにくい、ステートがインスタンス内に残るようなアプリケーションなどを検討する場面では、capacity-optimizedと呼ばれる配分戦略を選択することを検討してください。AWSがその時点で最も中断の発生しにくいスポットキャパシティプールを自動的に選択し、そこからスポットインスタンスを立ち上げる配分戦略です。
=======
=======
>>>>>>> updating
  1. Replace both occurrences of **"\<instance-id>"** with the instance-id of one of the Spot Instances that are currently running in your EC2 Auto Scaling group (you can get an instance-id from the Instances tab in the bottom pane of the [EC2 Auto Scaling groups console](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:view=details) ). You don't need to change any of the other parameters in the event json.
  1. Click **Create**
  1. With your new test name (i.e TestSpotInterruption) selected in the dropdown menu, click the **Test** button.
  1. The execution result should be **succeeded** and you can expand the details to see the successful log message: "Instance i-01234567890123456 belongs to AutoScaling Group runningAmazonEC2WorkloadsAtScale. Detaching instance..."
  1. Go back to the [EC2 Auto Scaling groups console](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:view=details), and under the **Activity History** tab in the bottom pane, you should see a **Detaching EC2 instance** activity, followed shortly after by a **Launching a new EC2 instance** activity.
  1. Go to the [EC2 ELB Target Groups console](https://console.aws.amazon.com/ec2/v2/home?1#TargetGroups:sort=targetGroupName) and click on the **runningAmazonEC2WorkloadsAtScale** Target Group, go to the Targets tab in the bottom pane, you should see the instance in `draining` mode.
<<<<<<< HEAD
=======
>>>>>>> update spot_resilience

ここまでの手順で、スポットインスタンスの中断通知を受けたタイミングでシームレスに新しいスポットインスタンスを起動し、スポットインスタンスが終了する前に安全に入れ替える仕組みを構築することができました。

{{% notice warning %}} 
実際に中断が発生する場合、Auto ScalingグループからデタッチしたインスタンスはEC2スポットサービスにより自動的に終了(Terminate)されます。今回は単に中断イベントをシミュレーションしただけであったためインスタンスは終了されません。デタッチしたインスタンスを忘れずに終了させてください。
{{% /notice %}}


### スポットインスタンスを利用するシステムの堅牢化

これまでスポットインスタンスの起動に際して、アベイラビリティゾーンごとに最も安い順に4種類のインスタンスタイプを9種類のインスタンスタイプから選択できるように構成してきました。ここからさらに、使用するインスタンスタイプとアベイラビリティゾーンの組み合わせを増やすことで、特定のスポットキャパシティプール(アベイラビリティゾーンとインスタンスタイプの組み合わせ)での中断の影響を緩和していくことができます。

#### チャレンジしてみましょう

今回作成したスポットインスタンスによる音楽ストリーミングアプリケーションをより堅牢にするために、どのような構成の改善を検討したらよいでしょうか。

{{%expand "考え方の一例" %}}

1. アベイラビリティゾーンを追加します。今の設定では2つのアベイラビリティゾーンを定義しています。ここにもうひとつアベイラビリティゾーンを追加することで、ある1箇所のスポットキャパシティプールでキャパシティが不足し、スポットインスタンスの中断が発生するとき、引き続きワークロードを受けることのできる可能性を高めることができます。

2. インスタンスタイプを追加します。さらに増やせば増やすほど中断発生する確率を下げることができます。今回は9種類のインスタンスタイプを定義していますが、さらに追加できるものがあるでしょうか。

3. スポットインスタンスを起動する対象となるスポットキャパシティプールを増やします。今回は9種類のインスタンスタイプのうち、4種類を上限に指定しました。これを増やすことで使用できるインスタンスタイプの多様性を高めることができ、中断発生確率の低下につながります。
{{% /expand %}}

#### チャレンジしてみましょう
スポットインスタンスの配分戦略には、今回選択したlowest-priceの他にどのようなものがあるでしょうか。今回のワークロードに最も適した配分戦略はどれになるでしょうか。
ヒント：新しい配分戦略についての[記事] (https://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/)を参照してください。

{{%expand "Click here for the answer" %}}
<<<<<<< HEAD
If you have workloads that are not stateless and fault-tolerant like the Web application that you deployed in this workshop, you can use the capacity-optimized allocation strategy, in order to instruct the ASG to launch instances in the capacity pools which are least likely to be interrupted.
>>>>>>> updating
=======
今回はいわゆるステートレスなアプリケーションを構築しました。これよりも中断を許容しにくい、ステートがインスタンス内に残るようなアプリケーションなどを検討する場面では、capacity-optimizedと呼ばれる配分戦略を選択することを検討してください。AWSがその時点で最も中断の発生しにくいスポットキャパシティプールを自動的に選択し、そこからスポットインスタンスを立ち上げる配分戦略です。
>>>>>>> update spot_resilience
=======

Great result! by leveraging the EC2 Spot Instance Interruption Warning, the Lambda Function detached the instance from the Auto Scaling group and the ELB Target Group, thus draining existing connections, and launching a replacement instance before the current instance is terminated.

{{% notice warning %}} 
In a real scenario, EC2 would terminate the instance after two minutes, however in this case we simply mocked up the interruption so the EC2 instance will keep running outside the Auto Scaling group. Go to the EC2 console and terminate the instance that you used on the mock up event.
{{% /notice %}}


### Increasing the application's resilience when using Spot Instances

In a previous step in this workshop, you learned that the EC2 Auto Scaling group is configured to fulfill the 4 lowest-priced instance types (out of a list of 9 types) in each Availability Zone. Since Spot is spare EC2 capacity, its supply and demand vary. By diversifying your usage of capacity pools (a combination of an instance type in an Availability Zone), you increase your chances of getting the desired capacity, and decrease the potential number of interrupted instances in case Spot Instances are interrupted (when EC2 needs the capacity back for On-Demand).

#### Knowledge check
How can you increase the resilience of the Koel music streaming application that you deployed in this workshop, when using EC2 Spot Instances?

{{%expand "Click here for the answer" %}}
1. Add an Availability Zone - the EC2 Auto Scaling group is currently deployed in two AZs. By adding an AZ to your application, you will tap into more EC2 Spot capacity pools, further diversifying your usage and decreasing the blast radius in case a Spot interruption occurs in one of the Spot capacity pools.
2. Add Instance Types - the 9 instance types that are configured in the Auto Scaling group have small performance variability, so it's possible to run all these instance types in a single ASG and scale on the same dynamic scaling policy. Are there any other instance types that can be added?
3. Increase SpotInstancePools - The ASG is configured to fulfill the 4 lowest-priced instance types. Increase this number to further diversify the usage of capacity pools and decrease the blast radius.
{{% /expand %}}

#### Challenge 
What other Spot allocation strategy can you choose, would it be suitable for this workload? if not, when will you use it?\
Hint: read or skim through the following [article] (https://aws.amazon.com/blogs/compute/introducing-the-capacity-optimized-allocation-strategy-for-amazon-ec2-spot-instances/)

{{%expand "Click here for the answer" %}}
If you have workloads that are not stateless and fault-tolerant like the Web application that you deployed in this workshop, you can use the capacity-optimized allocation strategy, in order to instruct the ASG to launch instances in the capacity pools which are least likely to be interrupted.
>>>>>>> updating
{{% /expand %}}
