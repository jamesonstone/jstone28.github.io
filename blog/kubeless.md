## Getting Started with Kubeless

[Kubeless](https://github.com/kubeless/kubeless) is a Kubernetes native serverless framework. It works by creating a "function" Custom Resource Definition (CRD). These functions are serverless functions that execute when called.

## Installing

Install the kubeless cli for function deployment

```bash
brew install kubeless
```

Create the namespace and function CRD from kubeless

```bash
kubectl create ns kubeless
kubectl create -f https://github.com/kubeless/kubeless/releases/download/v1.0.6/kubeless-v1.0.6.yaml
```

## Quick Start

Deploy the hello function into the cluster

```bash
kubeless function deploy hello --runtime python2.7 \
                                --from-file src/hello.py \
                                --handler test.hello
```

Open a proxy to `localhost:8080`

```bash
kubectl proxy -p 8080 &
```

Create a test call

```bash
curl -L --data '{"Another": "Echo"}' \
  --header "Content-Type:application/json" \
  localhost:8080/api/v1/namespaces/default/services/hello:http-function-port/proxy/
```

View the function pod logs

```bash
10.244.0.1 - - [23/Feb/2020:22:34:07 +0000] "GET /healthz HTTP/1.1" 200 2 "" "kube-probe/1.15" 0/117
{'event-time': None, 'extensions': {'request': <LocalRequest: POST http://localhost:8080/>}, 'event-type': None, 'event-namespace': None, 'data': {u'Another': u'Echo'}, 'event-id': None}
10.244.0.1 - - [23/Feb/2020:22:34:37 +0000] "POST / HTTP/1.1" 200 19 "" "curl/7.64.1" 0/9402
10.244.0.1 - - [23/Feb/2020:22:34:37 +0000] "GET /healthz HTTP/1.1" 200 2 "" "kube-probe/1.15" 0/143
```

## Sources

[Kubeless Quickstart](https://kubeless.io/docs/quick-start/)
