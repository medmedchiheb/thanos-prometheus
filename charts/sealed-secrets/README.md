# MCS Overview
* Documentation: [Intranet store secrets](https://intranet.maibornwolff.de/display/MCS/%5BConcept%5D+store+secrets+in+service+repositories)
* Code: [Github sealed-secrets](https://github.com/bitnami-labs/sealed-secrets)
* Chart: [Github helm/charts](https://github.com/helm/charts/tree/master/stable/sealed-secrets)

# Sealed Secrets

This chart contains the resources to use [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets).

## Prerequisites

* Kubernetes >= 1.9

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
$ helm install --namespace kube-system --name my-release stable/sealed-secrets
```

The command deploys a controller and [CRD](https://kubernetes.io/docs/tasks/access-kubernetes-api/custom-resources/custom-resource-definitions/) for sealed secrets on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

## Uninstalling the Chart

To uninstall/delete the `my-release` deployment:

```bash
$ helm delete [--purge] my-release
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Using kubeseal

Install the kubeseal CLI by downloading the binary from [sealed-secrets/releases](https://github.com/bitnami-labs/sealed-secrets/releases).

Fetch the public key by passing the release name and namespace:

```bash
kubeseal --fetch-cert \
--controller-name=my-release \
--controller-namespace=my-release-namespace \
> pub-cert.pem
```

Read about kubeseal usage on [sealed-secrets docs](https://github.com/bitnami-labs/sealed-secrets#usage).

## Configuration

| Parameter                     | Description                                                                | Default                                     |
|------------------------------:|:---------------------------------------------------------------------------|:--------------------------------------------|
| **controller.create**         | `true` if Sealed Secrets controller resources should be created            | `true`                                      |
| **rbac.create**               | `true` if rbac resources should be created                                 | `true`                                      |
| **rbac.pspEnabled**           | `true` if psp resources should be created                                  | `false`                                     |
| **serviceAccount.create**     | Whether to create a service account or not                                 | `true`                                      |
| **serviceAccount.name**       | The name of the service account to create or use                           | `"sealed-secrets-controller"`               |
| **secretName**                | The name of the TLS secret containing the key used to encrypt secrets      | `"sealed-secrets-key"`                      |
| **image.tag**                 | The `Sealed Secrets` image tag                                             | `v0.9.7`                                    |
| **image.pullPolicy**          | The image pull policy for the deployment                                   | `IfNotPresent`                              |
| **image.repository**          | The repository to get the controller image from                            | `quay.io/bitnami/sealed-secrets-controller` |
| **resources**                 | CPU/Memory resource requests/limits                                        | `{}`                                        |
| **crd.create**                | `true` if crd resources should be created                                  | `true`                                      |
| **crd.keep**                  | `true` if the sealed secret CRD should be kept when the chart is deleted   | `true`                                      |
| **networkPolicy**             | Whether to create a network policy that allows access to the service       | `false`                                     |
| **securityContext.runAsUser** | Defines under which user the operator Pod and its containers/processes run | `1001`                                      |
| **commandArgs**               | Set optional command line arguments passed to the controller process       | `[]`                                        |
| **ingress.enabled**           | Enables Ingress                                                            | `false`                                     |
| **ingress.annotations**       | Ingress annotations                                                        | `{}`                                        |
| **ingress.path**              | Ingress path                                                               | `/v1/cert.pem`                              |
| **ingress.hosts**             | Ingress accepted hostnames                                                 | `["chart-example.local"]`                   |
| **ingress.tls**               | Ingress TLS configuration                                                  | `[]`                                        |
| **podAnnotations**            | Annotations to annotate pods with.                                         | `{}`                                        |
| **priorityClassName**         | Optional class to specify priority for pods                                | `""`                                        |


- In the case that **serviceAccount.create** is `false` and **rbac.create** is `true` it is expected for a service account with the name **serviceAccount.name** to exist _in the same namespace as this chart_ before installation.
- If **serviceAccount.create** is `true` there cannot be an existing service account with the name **serviceAccount.name**.
- If a secret with name **secretName** does not exist _in the same namespace as this chart_, then on install one will be created. If a secret already exists with this name the keys inside will be used.

## Secret Rotation

To rotate  your secrets using the same key, do the following:

`kubeseal  --controller-name=sealed-secrets --format yaml --re-encrypt <old.json >new.json`


## Key Renewal: 


Genereate  new key
`kubectl exec  -n kube-system sealed-secrets-5b84cb7844-pb859 -- controller --key-cutoff-time="Tue, 18 Feb 2020 12:04:05 -0700"`

to rotate the actual key used by the controller, you have to do the following tasks:


- Label the current latest key as compromised `sealed-secret-key..` 

`kubectl label secrets -n kube-system sealed-secrets-key4mg5n sealedsecrets.bitnami.com/sealed-secrets-key=compromised --overwrite`



