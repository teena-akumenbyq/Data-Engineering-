# Session 02 — Deployment Strategies

## 1. Recreate (Big Bang)
Stop old version, deploy new. Simple but causes downtime.
- Use for: dev, non-critical batch jobs.

## 2. Rolling Deployment
Replace instances gradually, a few at a time.
- No full downtime; slower rollout.
- Common for services / AKS-hosted APIs.

## 3. Blue-Green Deployment
Two identical environments: **Blue** (live) and **Green** (new). Deploy to Green, test, then switch all traffic.
```
Traffic → [Blue v1]   |   [Green v2] (idle, tested)
   switch →           |   Traffic → [Green v2]
```
- Instant rollback (switch back to Blue).
- Cost: two full environments.

## 4. Canary Deployment
Release to a small % of traffic/users, watch metrics, then ramp up.
```
95% → v1
 5% → v2  → monitor → 25% → 50% → 100%
```
- Lowest risk; needs good monitoring.

## 5. Feature Flags / Toggles
Deploy code disabled, turn it on for subsets of users at runtime.
- Decouples deploy from release.
- Tools: LaunchDarkly, config flags.

## Comparison
| Strategy | Downtime | Rollback | Cost | Risk |
|----------|----------|----------|------|------|
| Recreate | Yes | Slow | Low | High |
| Rolling | No | Medium | Low | Medium |
| Blue-Green | No | Instant | High | Low |
| Canary | No | Fast | Medium | Lowest |
| Feature Flags | No | Instant (toggle) | Low | Low |

## For Data Pipelines
- **Blue-green tables**: write new data to a new table/schema, swap via view/pointer.
- **Canary for models**: run new dbt model in parallel, compare outputs before cutover.
- **Shadow runs**: run new pipeline alongside old, diff results, then promote.

## Lab
1. Match each strategy to a data-engineering scenario.
2. Design a blue-green swap using a view over two tables.
