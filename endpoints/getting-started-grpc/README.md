# Google Cloud Endpoints sample for Node.js

This sample demonstrates how to use Google Cloud Endpoints with Node.js.

For a complete walkthrough showing how to run this sample in different
environments, see the [Google Cloud Endpoints Quickstarts][docs_quickstart].

## Running locally

### Start a local server
```
$ node server.js -p 50051
```

### Run the client
```
$ node client.js -h localhost:50051
```

## Running on Google Cloud Platform
### Setup
Make sure you have [gcloud][gcloud] and [Node.js][nodejs] installed.

To update `gcloud`, use the `gcloud components update` command.

### Selecting an authentication method
1. Determine the appropriate API configuration file to use based on your authentication method.
- [JSON Web Tokens][jwt_io]: use `api_config.jwt.yaml`
- [API keys][gcp_api_key]: use `api_config.key.yaml`

2. Rename the `api_config.*.yaml` file you chose in Step 1 to `api_config.yaml`.

### Deploying to Endpoints
1. Install [protoc][protoc].

1. Compile the proto file using protoc.
```
$ protoc --include_imports --include_source_info protos/helloworld.proto --descriptor_set_out out.pb
```

1. In `api_config.yaml`, replace `MY_PROJECT_ID` and `SERVICE-ACCOUNT-ID` with your Project ID and your service account's email address respectively.

1. Deploy your service's configuration to Endpoints. Take note of your service's config ID and name once the deployment completes.
```
$ gcloud service-management deploy out.pb api_config.yaml
...
Service Configuration [SERVICE_CONFIG_ID] uploaded for service [SERVICE_NAME]
```

1. Build a Docker image for later use using the following command. Make sure to replace `[YOUR_PROJECT_ID]` with your Project ID.
```
$ gcloud container builds submit --tag gcr.io/[YOUR_PROJECT_ID]/endpoints-example:1.0 .
```

### Running your service
#### Compute Engine
1. [Create][console_gce_create] a Compute Engine instance. Be sure to check **Allow HTTP traffic** and **Allow HTTPS traffic** when creating the instance.

1. Once your instance is created, take note of its IP address.

Note: this IP address is _ephemeral_ by default, and may change unexpectedly. If you plan to use this instance in the future, [reserve a static IP address][docs_gce_static_ip] instead.

1. SSH into your instance, and install Docker.
```
$ sudo apt-get update
$ sudo apt-get install docker.io
```

1. Using the SSH connection to your instance, initialize the required Docker images in the order specified below. Replace `[YOUR_GCLOUD_PROJECT]`, `[YOUR_SERVICE_NAME]` and `[YOUR_SERVICE_CONFIG_ID]` with your GCloud Project ID, your service's name and your service's config ID respectively.
```
$ sudo docker run -d --name=helloworld gcr.io/[YOUR_GCLOUD_PROJECT]/endpoints-example:1.0
```

```
$ sudo docker run --detach --name=esp \
    -p 80:9000 \
    --link=helloworld:helloworld \
    gcr.io/endpoints-release/endpoints-runtime:1 \
    -s [YOUR_SERVICE_NAME] \
    -v [YOUR_SERVICE_CONFIG_ID] \
    -P 9000 \
    -a grpc://helloworld:50051
```

1. On your local machine, use the client to test your Endpoints deployment. Replace `[YOUR_INSTANCE_IP_ADDRESS]` with your instance's external IP address, and `[YOUR_API_KEY]` with a [valid Google Cloud Platform API key][gcp_api_key].
```
$ node client.js -h [YOUR_INSTANCE_IP_ADDRESS]:80 -k [YOUR_API_KEY]
```

#### Container Engine
1. If you haven't already, install `kubectl`.
```
$ gcloud components install kubectl
```

1. [Create][console_gke_create] a container cluster with the default settings. Remember the cluster's name and zone, as you will need these later.


1. Configure `kubectl` to have access to the cluster. Replace `[YOUR_CLUSTER_NAME]` and `[YOUR_CLUSTER_ZONE]` with your cluster's name and zone respectively.
```
$ gcloud container clusters get-credentials [YOUR_CLUSTER_NAME] --zone [YOUR_CLUSTER_ZONE]
```

1. Edit the `container_engine.yaml` file, and replace `GCLOUD_PROJECT`, `SERVICE_NAME`, and `SERVICE_CONFIG` with your Project ID and your Endpoints service's name and config ID respectively.

1. Add a [Kubernetes service][docs_k8s_services] to the cluster you created. Note that Kubernetes services should not be confused with [Endpoints services][docs_endpoints_services].
```
$ kubectl create -f container-engine.yaml
```

1. Get the external IP of your service. This may take a few minutes to be provisioned.
```
$ kubectl get service
```

### Testing your service
You can use the included client to test your Endpoints deployment.

1. Determine your service's IP address.

* If your service is hosted on Compute Engine, this will be your _instance's_ external IP address.

* If your service is hosted on Container Engine, this will be your _service's_ external IP address.

2. Run the client to connect to your service. When running the following commands, replace `[YOUR_IP_ADDRESS]` with the IP address you found in Step 1.

  * If you're using an API key, run the following command and replace `[YOUR_API_KEY]` with the appropriate [API key][gcp_api_key].
    ```
    $ node client.js -h [YOUR_CLUSTER_IP_ADDRESS]:80 -k [YOUR_API_KEY]
    ```

  * If you're using a [JSON Web Token][jwt_io], run the following command and replace `[YOUR_JWT_AUTHTOKEN]` with a valid JSON Web Token.
    ```
    $ node client.js -h [YOUR_CLUSTER_IP_ADDRESS]:80 -j [YOUR_JWT_AUTHTOKEN]
    ```

## Cleanup
If you do not intend to use the resources you created for this tutorial in the future, delete your [VM instances][console_gce_instances] and/or [container clusters][console_gke_instances] to prevent additional charges.

## Troubleshooting
If you're having issues with this tutorial, here are some things to try:
- [Check][console_logs] your GCE/GKE instance's logs
- Make sure your Compute Engine instance's [firewall](https://console.cloud.google.com/networking/firewalls/list) permits TCP access to port 80

If those suggestions don't solve your problem, please [let us know][github_issues] or [submit a pull request][github_pulls].

[nodejs]: https://nodejs.org/
[gcloud]: https://cloud.google.com/sdk/gcloud/
[jwt_io]: https://jwt.io
[protoc]: https://github.com/google/protobuf/#protocol-compiler-installation
[gcp_api_key]: https://support.google.com/cloud/answer/6158862?hl=en
[github_issues]: https://github.com/GoogleCloudPlatform/nodejs-docs-samples/issues
[github_pulls]: https://github.com/GoogleCloudPlatform/nodejs-docs-samples/pulls
[console_gce_instances]: https://console.cloud.google.com/compute/instances
[console_gce_create]: https://console.cloud.google.com/compute/instancesAdd
[console_gke_instances]: https://console.cloud.google.com/kubernetes/list
[console_gke_create]: https://console.cloud.google.com/kubernetes/add
[console_logs]: https://console.cloud.google.com/logs/viewer
[docs_k8s_services]: https://kubernetes.io/docs/user-guide/services/
[docs_endpoints_services]: https://cloud.google.com/endpoints/docs/grpc
[docs_gce_static_ip]: https://cloud.google.com/compute/docs/configure-ip-addresses#reserve_new_static
[docs_quickstart]: https://cloud.google.com/endpoints/docs/quickstarts
