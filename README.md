# Flux example to manage Helm Chart Deployment and Application

Please refer to the Flux documentation for [installation options.](https://fluxcd.io/docs/installation/)

You can find the blog post for this repository here: [https://anaisurl.com/full-tutorial-getting-started-with-flux-cd/](https://anaisurl.com/full-tutorial-getting-started-with-flux-cd/)
AND the YouTube tutorial: [https://youtu.be/5u45lXmhgxA](https://youtu.be/5u45lXmhgxA)

Other reference: [https://www.digitalocean.com/community/tutorials/how-to-set-up-a-continuous-delivery-pipeline-with-flux-on-digitalocean-kubernetes](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-continuous-delivery-pipeline-with-flux-on-digitalocean-kubernetes)

Export your Git credentials
```
export GITHUB_TOKEN=<your-token>
export GITHUB_USER=<your-username>
```

Install Flux in your K8s cluster
```
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=Flux-POC \
  --branch=main \
  --path=./clusters/my-cluster \
  --personal
```

## Installing the Starboard Helm Chart

Create the starboard namespace
```
kubectl create ns starboard-system
```

The Starboard Helm Chart can either be deployed through the following commands:
```
flux create source helm starboard-operator --url https://aquasecurity.github.io/helm-charts/ --namespace starboard-system
flux create helmrelease starboard-operator --chart starboard-operator \
  --source HelmRepository/starboard-operator \
  --chart-version 0.10.3 \
  --namespace starboard-system
```

OR declaratively by deploying [helm-starboard.yaml](helm-starboard.yaml)
```
kubectl apply -f helm-starboard.yaml
```

## Installing an application through Flux

Create the app namespace
```
kubectl create ns app
```

We are going to install the Kubernetes manifests from the following [Git repository](https://github.com/AnaisUrlichs/react-article-display): 

You can find the Flux installation in [application.yaml](application.yaml)

```
kubectl apply -f application.yaml
```

### Using the Flux bootstrap command

Similar to how you can deploy applications through ArgoCD, you can deploy an application directly through the Flux CLI.

```
flux create source git react \
    --url=https://github.com/AnaisUrlichs/react-article-display \
    --branch=main
```

```
flux create kustomization react-app \
  --target-namespace=app \
  --source=react \
  --path="./manifests" \
  --prune=true \
  --interval=5m \
```

## Create Slack alert for Flux

Creating an alert through Kubernetes manifests with Flux is pretty straight forward.

In this example, we are creating a Slack alert.

First, you need to have a channel with the Slack webhook URL. This video shows how to get the webhook URL.

Create the following secret:
```
kubectl -n flux-system create secret generic slack-url \
--from-literal=address=https://hooks.slack.com/services/YOUR/SLACK/WEBHOOK
```

Create the Provider that references the secret:
```
kubectl apply -f notification-provider.yaml
```

Create the alert:
```
kubectl apply -f alert.yaml 
```
## Some other useful Flux commands
```
watch flux get kustomizations
flux suspend kustomization kustomization_name
flux resume kustomization kustomization_name
```
