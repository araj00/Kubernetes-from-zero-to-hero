# Summary

1. Among nodeSelector, nodeaffinity, nodeselector is a simplest one to choose if you need to do pod deployment on nodes just based on its labels with no expressions required.

2. In requiredDuringSchedulingIgnoredDuringExecution, nodeSelectorTerms is a list type because if the rules matches to any one of it, it assigned pods to that node.
3. In preferredDuringSchedulingIgnoredDuringExecution, there is no nodeselectorterms field in yaml but a preference with a weight

```javascript
syntax differences:-

 requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: key-name
        operator: In | Exists | Gt | Lt | NotIn | NotExists
        values: [ values ]

preferredDuringSchedulingIgnoredDuringExecution 
- preference:
  - matchExpressions:
    - key: key-name
    operator: In | Exists | Gt | Lt | NotIn | NotExists
    values: [ values ]
  weight: value in number (Higher the number, more the preference is)
```

## NodeAffinity Types

Kubernetes offers two types of NodeAffinity:-

1. requiredDuringSchedulingIgnoredDuringExecution (Hard Requirement)

- Pod must be placed on nodes matching the rules
- If no matching node exists, Pod stays Pending
- Similar to nodeSelector but more expressive
- preferredDuringSchedulingIgnoredDuringExecution (Soft Preference)

2. Scheduler prefers nodes matching the rules

- If no matching nodes exist, Pod can still be scheduled elsewhere
- Uses weights (1-100) to influence scheduler decisions

**Note: weight: 50 in preferred NodeAffinity only influences scoring and cannot ensure equal pod distribution, use topologySpreadConstraints with maxSkew: 1 for guaranteed even spreading.**

## References

1. [Node Affinity](https://kubernetes.io/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/)
2. [Pod Topology Spread Constraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/)