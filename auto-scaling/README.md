# Summary related to AutoScaling

- HorizontalPodAutoscaler (HPA) automatically scales the number of Pods based on observed metrics
- Multiple metrics (CPU and memory) can be used simultaneously; HPA uses the highest calculated replica count
- minReplicas sets the lower bound for scaling to ensure minimum availability
- maxReplicas sets the upper bound to prevent resource exhaustion
- Stabilization windows prevent rapid scaling oscillations (flapping)
- HPA requires metrics-server or another metrics provider to function
- The autoscaling/v2 API provides advanced features like multiple metrics and custom behaviors

## Without HPA:

Fixed replicas (15) 
→ High cost during low traffic
→ Insufficient capacity during peak traffic

With HPA (minReplicas: 2, maxReplicas: 8):

Low traffic    → Scales down to 2 replicas ✅ (cost optimization)
Moderate load  → Maintains 3-5 replicas ✅ (balanced)
High traffic   → Scales up to 8 replicas ✅ (performance)
Stabilization  → Waits 5s before scaling down ✅ (prevents flapping)

## How HPA Makes Scaling Decisions

1. Metrics Collection:
- HPA queries metrics-server every 15 seconds (default)
- Retrieves current CPU and memory usage for all Pods

2. Calculation (for each metric):
desiredReplicas = ceil[currentReplicas × (currentMetric / targetMetric)]
   
Example (CPU):
- Current: 3 replicas using 240m CPU total (80m each)
- Target: 80% of 100m request = 80m per pod
- Current usage: 80m / 80m = 100%
- Desired: ceil[3 × (100 / 80)] = ceil[3.75] = 4 replicas

3. Decision:
- Takes the MAX of all metric calculations
- If CPU suggests 4 and memory suggests 3 → scales to 4
- Respects minReplicas (2) and maxReplicas (8) bounds

4. Stabilization:
- Waits 5 seconds before scaling down (prevents rapid changes)
- Scales up immediately when needed (default behavior)

## References:-

1. [vertical pod autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/vertical-pod-autoscale/)
2. [horizontal pod autoscaling](https://kubernetes.io/docs/concepts/workloads/autoscaling/horizontal-pod-autoscale/)