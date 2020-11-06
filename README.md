# Densify Operator

<img src="https://www.densify.com/wp-content/uploads/densify.png" width="300">

The Densify-Operator deploys the Densify Container Data Collection utility, which collects data from a Prometheus server and sends it to a Densify instance for analysis. 

## Prerequisites

- Densify account, which is provided with a Densify subscription or through a free trial (https://www.densify.com/service/signup)
- Prometheus (https://prometheus.io/)
- Kube-state-metrics (https://github.com/kubernetes/kube-state-metrics)

## Installing

### Basic Install
The instruction below deploys the basic Densify Container Data Collection with minimal configuration.

1. Update the Densify hostname, user and encrypted password, Prometheus hostname and port, and zip filename:
```
apiVersion: densify.com/v1
kind: Densify
metadata:
  name: densify
spec:
  config:
    densify:
      hostname: <instance>.densify.com
      port: 443
      protocol: https
      user: <densify user>
      epassword: <encrypted densify user password>
    prometheus:
      hostname: <prometheus hostname>
      port: <prometheus port>
      protocol: http
    zipname: data/<zip name>
```

### Install Using a Secret for User and Password
The instructions below deploy the Densify Container Data Collection using a secret to store the username and encrypted password.

1. Create a secret that contains the username and epassword keys.
```
oc -n "<namespace deploying operator into>" create secret generic densify-secret \
  --from-literal=username="<densify username>" \
  --from-literal=epassword="<encrypted password>" \
```

2. Update the Densify hostname, Prometheus hostname and port, and zip filename:
```
apiVersion: densify.com/v1
kind: Densify
metadata:
  name: densify
spec:
  config:
    densify:
      hostname: <instance>.densify.com
      port: 443
      protocol: https
      UserSecretName: densify-secret
    prometheus:
      hostname: <prometheus hostname>
      port: <prometheus port>
      protocol: http
    zipname: data/<zip name>
```
## Configuration
 
The configuration parameters are the same set as the [Densify Forwarder](https://github.com/densify-dev/helm-charts/tree/master/charts/container-optimization-data-forwarder#configuration) helm chart:

| Parameter        | Description           | Default |
| ------------- |-------------|--------|
| `config.densify.hostname` | Specify the Densify instance hostname. You may need to specify a fully qualified domain name. | `<instance>.densify.com` |
| `config.densify.protocol` | Specify the Densify connection protocol: http or https. | `<http/https>` |
| `config.densify.port` | Specify the Densify connection port number. | `443` |
| `config.densify.user` | Specify the Densify user account. You must specify either a password or an epassword along with this parameter. This user must already exist in your Densify instance and have API access privileges. Alternatively, you can use config.densify.UserSecretName for authentication instead of user/(e)password combination (see UserSecretName below). | `nil` |
| `config.densify.password` | Specify the password for the Densify user. Only specify a password or an encrypted password (not both). | `nil` |
| `config.densify.epassword` | Specify the encrypted password for the Densify User. If you specify an encrypted password, then disable the config.densify.password paramter. | `nil` |
| `config.densify.UserSecretName` | Specify the secret name used to store the Densify user and epassword (keys must contain username and epassword). If this parameter is used, then config.densify.username\password\epassword parameters must be disabled. | `nil` |
| `config.prometheus.hostname` | Specify the Prometheus address. Using the internal service name is recommended (i.e. `<service name>.<namespace>.svc`). | `nil` |
| `config.prometheus.protocol` | Specify the Prometheus connection protocol: http or https. | `<http/https>` |
| `config.prometheus.port` | Specify the Prometheus service connection port. | `9090` |
| `config.prometheus.clustername` | Specify the name to identify the cluster in the Densify UI. If this parameter is disabled, then the specified Prometheus hostname is used to identify the cluster in the Densify UI. | `nil` |
| `config.zipname` | Specify the name of the zipfile to send to Densify. | `data/nil` |
| `authenticated.create` | This flag controls the deployment of service account, cluster role, and cluster role binding in a secured Prometheus server environment. If OpenShift is used, then this flag should be set to true. | `true` |

Refer to the [Densify Forwarder](https://github.com/densify-dev/helm-charts/tree/master/charts/container-optimization-data-forwarder#configuration) helm chart for the full list of parameters.

## Limitation
* OS Support: Linux
* Architecture: AMD64

## Documentation
* [Densify Feature Description and Reference Guide](https://www.densify.com/docs/Content/Welcome.htm)

## License
Apache 2 Licensed. See [LICENSE](LICENSE) for full details.