Instructions to demo HPA on OCP

Optional- speed up metric collection window by adjusting 'window' field in adapter-config configmap- and then restart prometheus-adapter pods to use new configmap

    oc edit cm adapter-config -n openshift-monitoring
    oc get rs -n openshift-monitoring
    oc delete rs -n openshift-monitoring prometheus-adapter-<hash for promethus-adapter replicaset>

deploy pod which serves PHP webpage with expensive computations- requires privileged user account-

    oc adm policy add-scc-to-user anyuid -z default
    oc apply -f https://k8s.io/examples/application/php-apache.yaml

Create horizontal pod autoscaler to scale pod based on CPU load-

    oc autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

Create pod with shell we will use to request that PHP webpage

    oc run -it --rm load-generator --image=busybox /bin/sh 

Hit enter to reach command prompt

Run Commands to hit said webserver-

    while true; do wget -q -O- http://php-apache; done 

You can view CPU metrics by watching the HPA resource, and see new pods being spun up by watching the RS

    watch -n 0.5 oc get hpa
    watch -n 0.5 oc get rs

Important notes about HPAs-
- Requires resource requests and limits at the pod level
- Will only create additional pods. This can produce problems if nodes start to become tightly scheduled or run out of space. Solutions:
    - ClusterAutoScaler to provision additional nodes
    - Use pod priorities to ensure that critical pods are always able to be scheduled
