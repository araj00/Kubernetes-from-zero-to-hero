# Summary

1. In requiredDuringSchedulingIgnoredDuringExecution, nodeSelectorTerms is a list type because if the rules matches to any one of it, it assigned pods to that node.
2. In preferredDuringSchedulingIgnoredDuringExecution, there is no nodeselectorterms field in yaml but a preference with a weight

```
requiredDuringSchedulingIgnoredDuringExecution syntax:-

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