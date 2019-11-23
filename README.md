# mirror

Helm Chart to enable mc mirror across two MinIO clusters.

## Prerequisite

Setup local chart repository to serve this chart using [Chartmuseum](https://github.com/helm/chartmuseum). Download `chartmuseum` as [explained here](https://github.com/helm/chartmuseum#installation). Start the `chartmuseum` server like so:

```sh
chartmuseum --debug --port=8080   --storage="local"   --storage-local-rootdir="./chartstorage"
```

Then clone this repository locally and navigate to the directory. Then use the below commands:

```sh
helm package ./
curl --data-binary "@mirror-0.1.0.tgz" http://localhost:8080/api/charts
```

Then add helm `chartmuseum` repo using the below command:

```sh
helm repo add chartmuseum http://localhost:8080
helm repo update
```

It is expected that you have a Kubernetes cluster running at this point, with `helm` configured to talk to the cluster.

## Get Started

Now that everything is configured, lets start using the Chart. To customise your chart deploy with your own values, you could supply them via a simple yaml file as below,

```yaml
env:
  ## Format to set the alias is MC_HOST_<alias>=https://<Access Key>:<Secret Key>@<YOUR-S3-ENDPOINT>
  ## After this, mc should be able to access this cluster using "alias"
  MC_HOST_sitea: "http://AKIAIOSFODNN7EXAMPLE:wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY@site-a-minio:9000"
  MC_HOST_siteb: "http://AKIAIOSFODNN7EXAMPLE:wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY@site-b-minio:9000"

## Use this field to set the source and destination for mirror.
## source and destination can be site wide / bucket wide / prefix wide.
## Please ensure to add alias before bucket names. Alias should be set using env variables above.
location:
  source: "sitea"
  target: "siteb"
  buckets:
    - "bucket1"
    - "bucket2"
```

```sh
helm install chartmuseum/mirror -f args.yaml --generate-name
```

You can check the progress of mirror of the provided buckets between the source and target locations like so,
```sh
kubectl logs mirror-5fc7f5dc69-lkxrm mirror-bucket1

kubectl logs mirror-5fc7f5dc69-lkxrm mirror-bucket2
```

Finally verify if data is being mirrored from Source to target using `mc ls` or MinIO browser.
