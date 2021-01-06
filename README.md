
# Standalone Deployment

Single instance installation without TLS enabled

```
oc new-project hashicorp

oc apply -f ./install/
```

The following kubernetes components will be created:

* vault-server-binding ClusterRoleBinding
* vault ServiceAccount
* vault-storage PersistentVolumeClaim with 10Gi size
* vault-config ConfigMap
* vault Service
* vault Deployment
* vault Route
* vault NetworkPolicy

>
> vault-server-binding ClusterRoleBinding allows vault service account to leverage Kubernetes oauth with the oauth-delegator ClusterRole
>

>
> In case of OpenShift SDN Multitenant
>

```
oc adm  pod-network make-projects-global hashicorp
```


## Initialize Vault

```
POD=$(oc get pods -lapp.kubernetes.io/name=vault --no-headers -o custom-columns=NAME:.metadata.name)
oc rsh $POD

vault operator init -key-shares=1 -key-threshold=1
```

Save the `Unseal Key 1` and the `Initial Root Token`:

```
Unseal Key 1: QzlUGvdPbIcM83UxyjuGd2ws7flZdNimQVCNbUvI2aU=

Initial Root Token: s.UPBPfhDXYOtnv8mELhPA4br7
```

And export them as environment variables, for further use:

```
export KEYS=QzlUGvdPbIcM83UxyjuGd2ws7flZdNimQVCNbUvI2aU=
export ROOT_TOKEN=s.UPBPfhDXYOtnv8mELhPA4br7
export VAULT_TOKEN=$ROOT_TOKEN
```

## Unseal Vault

```
vault operator unseal $KEYS
```

## Enable a key/value secrets engine

```
vault secrets enable -path=secret kv
```

## Fetch vault hostname

From your local environment: 

```
echo http://$(oc get route vault --no-headers -o custom-columns=HOST:.spec.host)
```

# OCS storage cluster creation KMS settings:

Address: (vault host name with http protocol (ex: http://vault-hashicorp.apps))
Port: 80
Token: (Vault token)

In Advanced Settings:

Backend Path: secret

  


