# Roadmap

## In progress

- External review of the Helm package by [SixSq](https://sixsq.com/) including:
  - Test and validate Network policies
  - Test and validate requests and limits
  - Investigate redis overcommit

## To Do

- Define update strategy (rolling update or not)
- Develop liveness, readiness and startup probes
- Implement a logging system (i.e., Prometheus, Grafana)
- Customize the Helm package to allow deployment on different infrastructure
