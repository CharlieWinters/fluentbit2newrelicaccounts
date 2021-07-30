**Introduction**

This repo provides an example of how to route logs to two different New Relic accounts. The idea is that most logs will be sent to a standard account, but some pod logs will be sent to a secure account.

A rewrite tag filter has been added which looks at the Kubernetes pod annotations, if the pod is annotated with `"newrelic.com/secure": "true"`, then the configuration will add the `secure` prefix to the tag. There are two newrelic output plugins, the first will route all tags matching `kube.`, the second will route all tags matching `secure.`.

In the `new-relic-fluent-plugin.yml`, the `LICENSE_KEY` env can be set for standard logs and the `SECURE_LICENSE_KEY` can be set for the secure logs.

In summary with this configuration, you can annotate you Kubernetes pods with `"newrelic.com/secure": "true"` to route them to an alternative secure account.

**Steps**

Start the minikube
`minikube start`

Apply the logging yamls
`kubectl apply -f .`

To run an annotated pod, use the command below:
`kubectl run -i --tty --rm busyboxannotated --image=busybox --restart=Never   --overrides='{ "apiVersion": "v1", "metadata": {"annotations": { "newrelic.com/secure":"true" } } }'   -- sh`

Use the command below, to generate some stout and therefore logs for the secure pod:
`echo I'm secure pod`

To run another pod, open a seperate terminal and use the command below:
`kubectl run -i --tty --rm busybox --image=busybox --restart=Never -- sh`

Use the command below, to generate some stout and therefore logs for the pod:
`echo I'm an insecure pod`

In New Relic, you should see the annotated pod logs going to the account defined with the `SECURE_LICENSE_KEY` and the other pods logs going to the account defined with the `LICENSE_KEY`.
