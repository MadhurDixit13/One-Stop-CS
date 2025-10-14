# Deployment Strategies

Deployment strategies are methods for releasing new versions of software to production with minimal risk, downtime, and user impact.

---

## What Problem Does It Solve?

Deployment strategies help:
- **Reduce Deployment Risk**: Catch issues before full rollout
- **Enable Zero-Downtime Deployments**: Keep services available during updates
- **Support Fast Rollback**: Quickly revert to previous versions if problems arise
- **Test in Production**: Validate changes with real traffic
- **Improve User Experience**: Minimize disruptions during releases

---

## Common Strategies

### Recreate (Big Bang)
- Shut down old version, deploy new version
- **Pros**: Simple, clean state
- **Cons**: Downtime, high risk
- **Use Case**: Development/staging environments, non-critical apps

### Rolling Update
- Gradually replace instances with new version
- **Pros**: No downtime, gradual rollout
- **Cons**: Mixed versions running simultaneously
- **Use Case**: Stateless apps, Kubernetes default strategy

### Blue-Green Deployment
- Run two identical environments (blue=current, green=new)
- Switch traffic from blue to green after validation
- **Pros**: Instant rollback, full testing before switch
- **Cons**: Requires 2x resources, complex database migrations
- **Use Case**: Mission-critical apps, database schema changes

### Canary Deployment
- Deploy new version to a small subset of users/servers
- Gradually increase traffic to new version while monitoring metrics
- **Pros**: Early issue detection, controlled risk, real-world validation
- **Cons**: Requires sophisticated monitoring and traffic routing
- **Use Case**: High-traffic apps, feature testing, gradual rollouts

### A/B Testing
- Deploy multiple versions to compare user behavior and business metrics
- Route users to versions based on rules (e.g., 50/50 split)
- **Pros**: Data-driven decision making, user feedback
- **Cons**: Requires analytics infrastructure, longer evaluation period
- **Use Case**: Feature experiments, UX changes, pricing tests

### Shadow Deployment
- Route production traffic to both old and new versions, but only serve responses from old
- Compare results from both versions
- **Pros**: Zero user impact, real-world testing
- **Cons**: Doubles resource usage, complex to implement
- **Use Case**: Testing critical changes, performance validation

---

## Canary Deployment Deep Dive

### How It Works
1. Deploy new version to a small percentage of infrastructure (e.g., 5%)
2. Monitor key metrics (error rate, latency, CPU usage)
3. If metrics are healthy, gradually increase traffic (10% → 25% → 50% → 100%)
4. If issues detected, rollback to previous version

### Key Metrics to Monitor
- **Error Rate**: 5xx errors, exceptions
- **Latency**: p50, p95, p99 response times
- **Throughput**: Requests per second
- **Resource Usage**: CPU, memory, disk I/O
- **Business Metrics**: Conversion rate, checkout success

### Implementation Tools
- **Kubernetes**: Istio, Linkerd, Flagger
- **Cloud Providers**: AWS CodeDeploy, Google Cloud Deploy, Azure DevOps
- **CI/CD Tools**: Argo Rollouts, Spinnaker, Harness
- **Service Mesh**: Istio, Linkerd, Consul Connect
- **Feature Flags**: LaunchDarkly, Split.io, Unleash

### Example: Kubernetes Canary with Istio
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - match:
    - headers:
        user-agent:
          prefix: "canary"
    route:
    - destination:
        host: my-service
        subset: v2
      weight: 10  # 10% to canary
    - destination:
        host: my-service
        subset: v1
      weight: 90  # 90% to stable
```

---

## Best Practices

- **Automate Everything**: Use CI/CD pipelines for consistent deployments
- **Monitor Continuously**: Track metrics during and after deployment
- **Define Success Criteria**: Set thresholds for error rates, latency before proceeding
- **Use Feature Flags**: Decouple deployment from release for fine-grained control
- **Plan Rollback Strategy**: Always have a quick rollback mechanism
- **Test Thoroughly**: Run automated tests before deploying
- **Communicate**: Notify stakeholders of deployment windows and status
- **Database Migrations**: Handle schema changes carefully (backward compatibility)
- **Start Small**: Begin with low-risk services to gain confidence

---

## Further Reading

- [Kubernetes Deployment Strategies](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [Martin Fowler - Blue-Green Deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html)
- [Martin Fowler - Canary Release](https://martinfowler.com/bliki/CanaryRelease.html)
- [Argo Rollouts Documentation](https://argoproj.github.io/argo-rollouts/)
- [Istio Traffic Management](https://istio.io/latest/docs/concepts/traffic-management/)
- [Google SRE Book - Release Engineering](https://sre.google/sre-book/release-engineering/)

