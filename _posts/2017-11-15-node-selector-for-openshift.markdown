---
layout: post
title:  "How to use NodeSelector in OpenShift"
date:   2017-11-15 11:00:01 -0400
categories: openshift ocp nodeselector
---
# How to use nodeSelector in OpenShift Container Platform 3.x to run your pods on a specific node

Often in debugging scenarios I've found that I want my pods to be scheduled on a specific node. While this is partially documented in the OpenShift documentation, this is really not the intended behavior of OpenShift (you're kind of taking the orchestrator part out of container orchestrator) but regardless it is possible.

The way I've been going about this is to modify the deployment config yaml files and specifying the following snippet under the replication controller:

```
nodeSelector:
        kubernetes.io/hostname: ocp-appnode1.vm.example.com
```
where you specify the hostname as the hostname of the node you wish to run the pods on. Hence, a full snippet from a sample yaml file is as follows:

```
spec:
  replicas: 5
  selector:
    deploymentconfig: test-httpd
  strategy:
    activeDeadlineSeconds: 21600
    resources: {}
    rollingParams:
      intervalSeconds: 1
      maxSurge: 25%
      maxUnavailable: 25%
      timeoutSeconds: 600
      updatePeriodSeconds: 1
    type: Rolling
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test-httpd
        deploymentconfig: test-httpd
    spec:
      containers:
        - image: >-
            docker-registry.default.svc:5000/my-placement-project/test-httpd@sha256:4e03a55cbec322cd20c2b2337a46a64bd1c9ba8c3cf99ea9f7440caec088bc4c
          imagePullPolicy: Always
          name: test-httpd
          ports:
            - containerPort: 8080
              protocol: TCP
            - containerPort: 8443
              protocol: TCP
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      nodeSelector:
        kubernetes.io/hostname: ocp-appnode1.vm.example.com
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

Once you edit the yaml file (can do this through the OC CLI or through the web cconsole), either kick off a new deployment if you have to manually or simply have OpenShift pick up the config change.
