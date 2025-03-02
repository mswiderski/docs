# Cloud Events - Java and Spring

A simple web app written in Java using Spring Cloud Function that can receive CloudEvents. It
supports running in two modes:

1. The default mode has the app reply to your input events with the output
   event, which is simplest for demonstrating things working in isolation, but
   is also the model for working with the Knative Eventing `Broker` concept. The
   input event is modified assigning a new source and type attribute.

2. `K_SINK` mode has the app send events to the destination encoded in
   `$K_SINK`, which is useful to demonstrate how folks can synthesize events to
   send to a Service or Broker when not initiated by a Broker invocation (e.g.
   implementing an event source). The input event is modified assigning a new
   source and type attribute.

The application will use `$K_SINK`-mode whenever the environment variable is
specified.

Do the following steps to create the sample code and then deploy the app to your
cluster. You can also download a working copy of the sample, by running the
following commands:

```bash
git clone -b "{{ branch }}" https://github.com/knative/docs knative-docs
cd knative-docs/docs/serving/samples/cloudevents/cloudevents-spring
```

## Before you begin

- A Kubernetes cluster with Knative installed and DNS configured. Follow the
  [installation instructions](../../../../admin/install/serving/install-serving-with-yaml.md) if you need to
  create one.
- [Docker](https://www.docker.com) installed and running on your local machine,
  and a Docker Hub account configured (we'll use it for a container registry).

## Build and deploy the sample

To build the image, run:

```bash
mvn compile jib:build -Dimage=<image_name>
```

To deploy the Knative Service, edit the `service.yaml` file and replace `<image>`
with the deployed image name. Then run:

```bash
kubectl apply -f service.yaml
```

## Testing the sample

Get the URL for your Service with:

```bash
$ kubectl get ksvc
NAME                URL                                            LATESTCREATED             LATESTREADY               READY   REASON
cloudevents-spring   http://cloudevents-java.sslip.io                 cloudevents-spring-86h28   cloudevents-spring-86h28   True
```

Then send a CloudEvent to it with:

```bash
$ curl \
    -X POST -v \
    -H "content-type: application/json"  \
    -H "ce-specversion: 1.0"  \
    -H "ce-source: http://curl-command"  \
    -H "ce-type: curl.demo"  \
    -H "ce-id: 123-abc"  \
    -d '{"name":"Dave"}' \
    http://cloudevents-java.sslip.io
```

You can also send CloudEvents spawning a temporary curl pod in your cluster
with:

```bash
$ kubectl run curl \
    --image=curlimages/curl --rm=true --restart=Never -ti -- \
    -X POST -v \
    -H "content-type: application/json"  \
    -H "ce-specversion: 1.0"  \
    -H "ce-source: http://curl-command"  \
    -H "ce-type: curl.demo"  \
    -H "ce-id: 123-abc"  \
    -d '{"name":"Dave"}' \
    http://cloudevents-java.default.svc
```

You'll see on the console:

```bash
> POST / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.69.1
> Accept: */*
> content-type: application/json
> ce-specversion: 1.0
> ce-source: http://curl-command
> ce-type: curl.demo
> ce-id: 123-abc
> Content-Length: 15
>
< HTTP/1.1 202 Accepted
< ce-specversion: 1.0
< ce-id: 123-abc
< ce-source: https://github.com/knative/docs/docs/serving/samples/cloudevents/cloudevents-spring
< ce-type: curl.demo
< content-type: application/json
< content-length: 15
<
{"name":"Dave"}
```

## Removing the sample app deployment

To remove the sample app from your cluster, delete the service record:

```bash
kubectl delete --filename service.yaml
```

## Removing the sample app deployment

To remove the sample app from your cluster, delete the service:


=== "yaml"

    Run:

    ```bash
    kubectl delete --filename service.yaml
    ```


=== "kn"

    Run:

    ```bash
    kn service delete cloudevents-spring
    ```
