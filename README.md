# Densify Operator

<img src="https://www.densify.com/wp-content/uploads/densify.png" width="300">

The Densify-Operator deploys the Densify Container Data Collection utility, which collects data from a Prometheus server and sends it to a Densify instance for analysis. 

## Prerequisites

- Densify account, which is provided with a Densify subscription or through a free trial (https://www.densify.com/service/signup)
- Prometheus (https://prometheus.io/)
- Kube-state-metrics (https://github.com/kubernetes/kube-state-metrics)

## Installing

### Basic Install
Deploy the basic Densify Container Data Collection with min configuration
* Need to update the Densify hostname, user and encrypted password, Prometheus hostname and port, and zip name
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
Deploy the Densify Container Data Collection using a secret to store the username and encrypted password
* Create a secret that contains the keys username and epassword 
```
oc -n "<namespace deploying operator into>" create secret generic densify-secret \
  --from-literal=username="<densify username>" \
  --from-literal=epassword="<encrypted password>" \
```
* Need to update the Densify hostname, Prometheus hostname and port, and zip name
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
 
The config parameters are the same set as the Densify Operator helm chart:

| Parameter        | Description           | Default |
| ------------- |-------------|--------|
| `nameOverride` | Specify the helm chart name override. | `densify-forwarder` |
| `image` | Specify the image to use for the Densify Container Optimization Data Forwarder. | `densify/container-optimization-data-forwarder:latest` |
| `pullPolicy` | Specify the image pull policy for the Densify Container Optimization Data Forwarder. | `Always` |
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
| `config.prometheus.interval` | The interval to collect data from Prometheus: hours or days. | `hours` |
| `config.prometheus.intervalSize` | The size of the interval to collect data from Prometheus. For example, if interval=hours and intervalSize=3, then 3 hours block of data would be collected each time. By default, 1 hour of data is collected. | `1` |
| `config.prometheus.history` | The number of intervalSize block of data to collect for historical purposes. For example, if interval=hours, intervalSize=2, and history=12, then 24 hours of historical data is collected. By default, the last hour of data is collected for history. | `1` |
| `config.prometheus.sampleRate` | The sample rate for data points to be collected. By default, samples are recorded every 5 minutes within the intervalSize block of data collected.  | `5` |
| `config.prometheus.includeList` | The list of included data types to collect. | `container,node,nodegroup,cluster` |
| `config.prometheus.oauth_token` | Specify the path to the OAuth token used to authenticate with a secured Prometheus server. | `/var/run/secrets/kubernetes.io/serviceaccount/token` |
| `config.prometheus.ca_certificate` | Specify the CA certificate used to commuicate with a secured Prometheus server. | `/var/run/secrets/kubernetes.io/serviceaccount/service-ca.crt` |
| `config.proxy.host` | Specify name of the proxy host. | `nil` |
| `config.proxy.port` | Specify the proxy port number. | `nil` |
| `config.proxy.protocol` | Specify the proxy protocol: http or https. | `<http/https>` |
| `config.proxy.auth` | Specify the proxy authentication type: Basic or NTLM. | `nil` |
| `config.proxy.user` | Specify the username used for the proxy server. You need to specify either the proxy password or encrypted password in conjunction with this parameter. | `nil` |
| `config.proxy.password` | Specify the password for the proxy server user. If the proxy password is specified, you need to disable the config.proxy.epassword parameter.  | `nil` |
| `config.proxy.epassword` | Specify the encrypted password for the proxy server user. If the proxy epassword is specified, you need to disable the config.proxy.password parameter. | `nil` |
| `config.proxy.domainuser` | Specify the domain user for proxy NTLM authentication. | `nil` |
| `config.proxy.domain` | Specify the domain for proxy NTLM authentication. | `nil` |
| `config.zipEnabled` | This flag indicates whether data files are zipped before sending to Densify. | `true` |
| `config.zipname` | Specify the name of the zipfile to send to Densify. | `data/nil` |
| `config.cronJob.schedule` | The cronjob schedule. By default, data collection is triggered at the top of every hour. This is in line with the default interval settings of collecting the last hour of data. | `0 * * * *` |
| `config.debug` | This flag indicates debug logging. | `false` |
| `authenticated.create` | This flag controls the deployment of service account, cluster role, and cluster role binding in a secured Prometheus server environment. If OpenShift is used, then this flag should be set to true. | `false` |
| `nodeSelector` | The node labels for pod assignments. | `{}` |
| `resources` | The CPU/Memory resource requests/limits. | `{}` |
| `tolerations` | The toleration labels for pod assignments. | `{}` |

## Limitation
* OS Support: Linux
* Architecture: AMD64

## Documentation
* [Densify Feature Description and Reference Guide](https://www.densify.com/docs/Content/Welcome.htm)

## License
Apache 2 Licensed. See [LICENSE](LICENSE) for full details.