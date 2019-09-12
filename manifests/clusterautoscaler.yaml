---
apiVersion: autoscaling.openshift.io/v1
kind: ClusterAutoscaler
metadata:
  name: default
spec:
  podPriorityThreshold: -10
  resourceLimits:
    maxNodesTotal: 24
    cores:
      min: 8
      max: 128
    memory:
      min: 4
      max: 256
  scaleDown:
    enabled: true
    delayAfterAdd: 30m
    delayAfterDelete: 30m
    delayAfterFailure: 5m
    unneededTime: 5m
