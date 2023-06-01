# Roadmap

## In progress

- Implement a logging system (i.e., Prometheus, Grafana) _[outsourced to [SixSq](https://sixsq.com/)]_
- Customize the Helm package to allow deployment on different infrastructure _[outsourced to [SixSq](https://sixsq.com/)]_

## To Do

- Use `envFrom` for better environment variables handling (shorter manifests)
- Define update strategy (rolling update or not)
- Develop liveness, readiness and startup probes
- Test and validate requests and limits from a real use of the MapX instance
