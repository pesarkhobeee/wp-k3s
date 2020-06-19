# wp-k3s

I was thinking how can I set up my personal website on a minimal VPS with the help of Kubernetes?

I just tried to collect all phases of the puzzle:
* [K3S project](https://k3s.io/) which is a lightweight Kubernetes
* [cert-manager.io](https://cert-manager.io/) which is a automate certificate management in the cloud 
* [Grafana Loki](https://grafana.com/oss/loki/) which is a log aggregation system inspired by Prometheus.

Then on top of them, I changed this example of WordPress for Kubernetes:
https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

What I have added to that?
* ReadinessProbe
* Replicas number for the WordPress deployment
* Requests and Limits for the WordPress deployment
* HPA for WordPress
* Ingress controller
* I changed PersistentVolume manner and you can find WordPress and MySQL data in your host machine in /opt/wp_k3s path

My test VPS has just 2 GB ram and 1 vCPU with 20GB disk space and it seems it is working :) but I should mention that this is not recommended specification and the only missing part is a regular mechanism for back up which I think Rclone it the solution for that but I will leave it to you to decide which way fits you better.

## Instalation:

Let's start with installing K3s which is super easy, just run below command:

```
  curl -sfL https://get.k3s.io | sh -
  # Check for Ready node, takes maybe 30 seconds
  k3s kubectl get node
```

It is better to install git on your server and clone the current repository on your server and run a setup step like this:

```
git clone https://github.com/pesarkhobeee/wp-k3s.git
cd wp-k3s
kubectl apply -f setup-cert-manager.yaml
```

The next step is editing the `kustomization.yaml` file and giving a strong password to MySQL:

```
vi kustomization.yaml
```

Now it is time to enter our domain name and email address, please edit `ingress.yaml` and change all example.com cases and after that apply the whole directory like this,  don't forget that requirement of this step is your domain must refer to this server Ip address:

```
vi ingress.yaml
kubectl apply -k .
```

In the end, you should be able to see the WordPress installation page on your domain name.

## Optional step:

If you have enough resource on your server and you like to have a log aggregation layer, Grafana Loki project is one of the most efficient options out there, just apply the desire YAML file with this command:

```
kubectl apply -f optional-basic-log-aggregation_fluentd-daemonset.yaml
```

then you should add an HTTP endpoint for that in your `ingress.yaml` like:

```
...
    - host: example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: wordpress
              servicePort: 80
    - host: logs.example.com
      http:
        paths:
          - path: /
            backend:
              serviceName: grafana
              servicePort: 3000
  tls:
  - hosts:
    - example.com
    - logs.example.com
    secretName: letsencrypt-prod
```

please make sure that you already defined that subdomain in your control panel and now you can apply the changes by:

```
kubectl apply -k .
```

now you can login to your new subdomain, the default user name, and password for Grafana is `Admin`, login and change the password, then go to `Configuration` and click on `Add data source` and chose `Loki`, now you should fill URL which is `http://loki:3100` and save it. 
The last step is going to `explore` section and select `app` -> `wordpress` from `Log labels`:

![ScreenShot](https://raw.github.com/pesarkhobeee/wp-k3s/master/screenshot.png)

## TODO:
- [x] Add HPA and local storage
- [x] Add log aggregation layer with fluentd
- [x] Add Cert-manager and Traefik
- [ ] Add Rclone
- [x] Add the documentations
