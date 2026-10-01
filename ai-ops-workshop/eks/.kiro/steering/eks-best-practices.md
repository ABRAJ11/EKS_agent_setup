# EKS Deployment Standards

When deploying to Amazon EKS:

- Always set resource requests and limits on containers
- Always configure liveness and readiness probes
- Use the existing cluster `eksworkshop-eksctl` in the current AWS region
- Use Kubernetes Ingress with ALB annotations for external access following https://docs.aws.amazon.com/eks/latest/userguide/auto-configure-alb.html
- Apply labels `app.kubernetes.io/name` and `app.kubernetes.io/component` to all resources
- Use namespaces to isolate workloads
- Follow the principle of least privilege for RBAC and service accounts

## React Application Build

Generate the workshop React application with exactly the dependency versions below. This
combination is known-good and conflict-free on the Node 20 runtime in this environment, and
it is what makes `npm install && npm run build && npm start` succeed on the first attempt.

Use this `package.json` dependency set verbatim — do not substitute newer majors, and do not
use version ranges that can resolve to a different major:

```json
{
  "type": "module",
  "scripts": {
    "build": "vite build",
    "start": "node server.js"
  },
  "dependencies": {
    "express": "4.22.2",
    "react": "18.3.1",
    "react-dom": "18.3.1"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "4.7.0",
    "vite": "5.4.21"
  }
}
```

Rules that go with those pins:

- Do NOT use Create React App or `react-scripts`. `react-scripts@5.0.1` is unmaintained and its
  peer dependencies conflict with current React majors, which is the usual cause of a failed
  first build. Use Vite as the build tool.
- Do NOT pair React 19 with this toolchain. React and react-dom must both be `18.3.1` and must
  match each other exactly.
- `npm run build` must produce a static bundle in `dist/`. `npm start` must run `node server.js`,
  an Express server that serves `dist/` and exposes `GET /health` returning JSON
  (`{"status":"healthy"}`).
- The server must bind `0.0.0.0` and read the listen port from the `PORT` environment variable,
  defaulting to `3000`, so it works both locally and inside a container behind a load balancer.
- The app is served behind a reverse proxy under a path prefix (for example `/ports/3000/`), NOT
  at the domain root. Everything must resolve relative to that prefix:
  - Configure Vite with `base: './'` so all built asset URLs in `index.html` are relative. Absolute
    asset paths (`/assets/...`) 404 under a path prefix and render a blank page even though `curl`
    of the HTML succeeds.
  - Write all in-app links and API calls as relative paths, never root-absolute. Use `href="health"`
    and `fetch("health")`, NOT `href="/health"` or `fetch("/health")` — a leading slash escapes the
    proxy prefix and 404s. The `/health` route on the Express server is correct; only the client-side
    references to it must be relative.
- Every build and runtime dependency must be declared in `package.json`. Do not rely on globally
  installed tooling and do not require a separate `npm install -g` step.
- Do not add a dependency that is not needed for the above. Extra UI or state libraries introduce
  peer-dependency risk for no benefit in this workshop.
