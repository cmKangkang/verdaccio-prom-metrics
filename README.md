# verdaccio-prom-metrics

A fork of [verdaccio-prometheus-middleware](https://github.com/xlts-dev/verdaccio-prometheus-middleware).

## 安装

Because Verdaccio automatically prefixes the string `verdaccio-` to the directory it looks for when loading a plugin,
this plugin will need to be installed to a directory named `verdaccio-metrics` _IF_ your `middlewares` configuration
uses `metrics` as the configuration key for this plugin (as shown in the [Configuration](#configuration) example). Below
is an example `Dockerfile` that can be used to build a custom image containing this plugin. The Verdaccio
[`plugins`](https://verdaccio.org/docs/configuration#plugins) configuration option is assumed to be set to
`/verdaccio/plugins`.

```Dockerfile
# Docker multi-stage build - https://docs.docker.com/develop/develop-images/multistage-build/
# Use an alpine node image to install the plugin
FROM node:lts-alpine as builder

# Install the metrics middleware plugin. Replace `x.y.z` with the plugin version.
RUN mkdir -p /verdaccio/plugins \
    && cd /verdaccio/plugins \
    && npm install --global-style --no-bin-links --omit=optional @xlts.dev/verdaccio-prometheus-middleware@x.y.z

# The final built image will be based on the standard Verdaccio docker image.
FROM verdaccio/verdaccio:5

# Copy the plugin files over from the 'builder' node image.
# The `$VERDACCIO_USER_UID` env variable is defined in the base `verdaccio/verdaccio` image.
# Refer to: https://github.com/verdaccio/verdaccio/blob/v5.13.3/Dockerfile#L32
COPY --chown=$VERDACCIO_USER_UID:root --from=builder \
  /verdaccio/plugins/node_modules/@xlts.dev/verdaccio-prometheus-middleware \
  /verdaccio/plugins/verdaccio-metrics
```

## 配置

```yaml
middlewares:
  metrics:
    metricsPath: /custom/path/metrics
    defaultMetrics:
      enabled: true
    requestMetrics:
      enabled: true
      metricName: 'registry_http_requests'
      pathExclusions:
        - '^/$'
        - '^/[-]/ping'
        - '^/[-]/(static|verdaccio|web)'
        - '[.]ico$'

    packageMetrics:
      enabled: true
      metricName: 'registry_package_downloads'
      packageGroups:
        '@angular/[^/]*': angular
        'react': react
        '.*': other
```
