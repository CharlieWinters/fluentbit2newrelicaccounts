**Introduction**

This repo provides an example of how to route logs to two different New Relic accounts. The idea is that most logs will be sent to a standard account, but some pod logs will be sent to specific accounts defined by pod annotations.

A rewrite tag filter has been added which looks at the Kubernetes pod annotations, if the pod is annotated with `"newrelic.com/account_name": "account_name_1"`, then the configuration will add the `account_name_1` prefix to the tag, if the pod is annotated with `"newrelic.com/account_name": "account_name_2"`, then the configuration will add the `account_name_2` prefix to the tag. 

There are three newrelic output plugins, the first will route all tags matching `kube.`, the second will route all tags matching `account_name_1.` and the third will route all tags matching `account_name_2.`.

In the `new-relic-fluent-plugin.yml`, the `LICENSE_KEY` env can be set for standard logs,  `ACCOUNT_NAME_1_LICENSE_KEY` can be set for the account 1 logs and `ACCOUNT_NAME_2_LICENSE_KEY` can be set for the account 2 logs.

In summary with this configuration, you can annotate you Kubernetes pods with `"newrelic.com/account_name": "account_name_1"` or `"newrelic.com/account_name": "account_name_2"` to route them to an alternative accounts.

**Steps**

Start the minikube
`minikube start`

Apply the logging yamls
`kubectl apply -f .`

To run the first annotated pod and generate some stout and therefore logs for account 1, use the commands below:
`kubectl run -i --tty --rm busyboxannotated1 --image=busybox --restart=Never --overrides='{ "apiVersion": "v1", "metadata": {"annotations": { "newrelic.com/account_name":"account_name_1" } } }' -- sh`

`echo hello account name 1`

To run the second annotated pod and generate some stout and therefore logs for account 2, open a seperate terminal and use the commands below:
`kubectl run -i --tty --rm busyboxannotated2 --image=busybox --restart=Never --overrides='{ "apiVersion": "v1", "metadata": {"annotations": { "newrelic.com/account_name":"account_name_2" } } }' -- sh`

`echo hello account name 2`

To run another pod without annotations, open a seperate terminal and use the command below:

`kubectl run -i --tty --rm busybox --image=busybox --restart=Never -- sh`

`echo hello default account`


In New Relic, you should see the first annotated pod logs going to the account defined with the `ACCOUNT_NAME_1_LICENSE_KEY`, you should see the second annotated pod logs going to the account defined with the `ACCOUNT_NAME_2_LICENSE_KEY` and the other pods logs going to the account defined with the `LICENSE_KEY`.
