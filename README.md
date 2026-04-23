# filmapp

A Netflix-style movie/TV browser built with **React + TypeScript + Vite**, running on data from the TMDB API. Packaged with Docker and delivered through a Jenkins pipeline that covers SAST (SonarQube), dependency scanning (OWASP Dependency-Check), container scanning (Trivy), and rollout to Kubernetes.

**Stack:** React · TypeScript · Vite · Material UI · Redux Toolkit · Docker · Jenkins · Kubernetes · SonarQube · Trivy · OWASP Dependency-Check

## Features

- Home page with trending / genre carousels
- Movie and TV detail modals with trailers (video.js + YouTube)
- Genre breadcrumbs, search, watchlist state via Redux Toolkit
- Responsive, dark-themed UI with MUI + Emotion

## CI/CD pipeline

The `Jenkinsfile` wires up a DevSecOps delivery loop:

| Stage | Tool | Purpose |
|---|---|---|
| Checkout | Git | Clone the repo |
| SAST | SonarQube | Code smells, bugs, coverage |
| Quality gate | SonarQube | Fail-fast on quality threshold |
| Dependencies | OWASP Dependency-Check | Known-CVE scanning of `package.json` |
| Filesystem scan | Trivy | Secrets / misconfig scan |
| Build + Push | Docker + Docker Hub | Multi-stage image (Node build → nginx serve) |
| Image scan | Trivy | Vulnerabilities in the final image |
| Deploy | `kubectl` | Apply manifests in `Kubernetes/` |

## Build

```bash
docker build --build-arg TMDB_V3_API_KEY=<your-tmdb-key> -t ofdengiz/netflix:latest .
docker run -d -p 8081:80 ofdengiz/netflix:latest
```

Inject the TMDB key as a Jenkins credential in real pipelines — never hardcode it.

## Kubernetes

```bash
kubectl apply -f Kubernetes/
```

## For my current AWS / DevOps work

See **[omerdengiz.com](https://omerdengiz.com)** — static site on AWS S3 + CloudFront + Lambda@Edge, provisioned with Terraform across two AWS accounts. Source: [`ofdengiz/omerdengiz-com`](https://github.com/ofdengiz/omerdengiz-com).
