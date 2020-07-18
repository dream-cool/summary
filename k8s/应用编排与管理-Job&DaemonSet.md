# 应用编排与管理 Job&DaemonSet

## Job

![avatar](https://images.gitbook.cn/FlJHcV-dW9UFVax6IaehPYtKCld-)

restartPolicy：在 Job 里面我们可以设置 Never、OnFailure、Always 这三种重试策略。在希望 Job 需要重新运行的时候，我们可以用 Never；希望在失败的时候再运行，再重试可以用 OnFailure；或者不论什么情况下都重新运行时 Alway；

backoffLimit ：Job重试的次数

