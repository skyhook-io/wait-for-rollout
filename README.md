# Argo Rollouts Wait

Waits for an Argo Rollout to complete with configurable timeout and health verification.

## Features

- ⏳ **Configurable timeout** - Set maximum wait time
- 🔍 **Health monitoring** - Tracks rollout status
- 📊 **Detailed output** - Returns status, message, and revision
- 🔧 **Plugin management** - Auto-installs kubectl plugin
- 📝 **Summary generation** - GitHub Actions summary

## Usage

```yaml
- name: Wait for rollout
  uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: backend
    namespace: production
    timeout: 600s
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `rollout_name` | Name of the Argo Rollout | ✅ | - |
| `namespace` | Kubernetes namespace | ❌ | `default` |
| `timeout` | Maximum wait time (e.g., 300s, 5m, 10m) | ❌ | `300s` |
| `verify_only` | Only verify health without promoting | ❌ | `false` |
| `install_plugin` | Install Argo Rollouts kubectl plugin | ❌ | `true` |
| `plugin_version` | Plugin version to install | ❌ | `latest` |

## Outputs

| Output | Description | Example |
|--------|-------------|---------|
| `status` | Final rollout status | `Healthy`, `Degraded`, `Progressing` |
| `message` | Status message | `Rollout is healthy` |
| `revision` | Current revision number | `5` |

## Examples

### Basic usage
```yaml
- name: Deploy and wait
  run: kubectl apply -f manifests/

- name: Wait for rollout
  uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: my-service
    namespace: production
```

### With custom timeout
```yaml
- name: Wait for complex rollout
  uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: backend
    namespace: staging
    timeout: 15m  # Long timeout for canary
```

### Verify only mode
```yaml
- name: Check rollout health
  uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: frontend
    namespace: production
    verify_only: true  # Don't promote, just check
```

### With specific plugin version
```yaml
- name: Wait with specific plugin
  uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: api
    namespace: default
    plugin_version: v1.6.0
```

### Complete deployment flow
```yaml
jobs:
  deploy:
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: skyhook-io/cloud-login@v1
        with:
          provider: aws
          region: us-east-1
          cluster: production
      
      - name: Apply manifests
        run: |
          kustomize build overlays/production | kubectl apply -f -
      
      - name: Wait for rollout
        uses: skyhook-io/rollouts-wait@v1
        with:
          rollout_name: ${{ env.SERVICE_NAME }}
          namespace: production
          timeout: 10m
      
      - name: Run smoke tests
        if: success()
        run: ./scripts/smoke-test.sh production
```

## Rollout Strategies

The action supports all Argo Rollouts strategies:

### Canary
```yaml
# Waits for canary to complete
- uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: my-canary
    timeout: 20m  # Longer for gradual rollout
```

### Blue-Green
```yaml
# Waits for blue-green switch
- uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: my-bluegreen
    timeout: 5m  # Faster switch
```

### Progressive Delivery
```yaml
# Monitors progressive stages
- uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: my-progressive
    timeout: 30m  # Multiple stages
```

## Prerequisites

- Kubernetes cluster access (configured kubectl)
- Argo Rollouts installed in the cluster
- Appropriate RBAC permissions

## Plugin Installation

The action automatically installs the kubectl-argo-rollouts plugin if not present.

To skip installation (if pre-installed):
```yaml
- uses: skyhook-io/rollouts-wait@v1
  with:
    rollout_name: my-service
    install_plugin: false
```

## Error Handling

The action will fail if:
- Rollout doesn't exist
- Timeout is exceeded
- Rollout enters Degraded state
- kubectl connection fails

## Monitoring Output

The action provides detailed status in GitHub Actions summary:
- Rollout name and namespace
- Final status and message
- Revision number
- Success/failure indication

## Notes

- Requires kubectl context to be configured
- Use with cloud-login action for authentication
- Timeout format supports: seconds (300s), minutes (5m), hours (1h)
- verify_only mode useful for health checks without side effects