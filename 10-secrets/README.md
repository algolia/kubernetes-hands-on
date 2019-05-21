# Secrets

Objects of type `Secret` are intended to hold sensitive information, such as passwords, OAuth tokens, and ssh keys. Putting this information in a secret is safer and more flexible than putting it verbatim in a pod definition or in a docker image.

```yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4= # admin
  password: cGFzc3dvcmQ= # password
```

* `data`: is a list of key/values. The values must be in base64.

You can apply the file:

```sh
$ kubectl apply -f 10-secrets/01-secrets.yml
secret "mysecret" created
```

You can reference a secret from a pod, either per env variable or mounting a volume containing a secret.

Here we mount the secret `mysecret` to the path `/etc/foo` inside the pod:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-volume-secrets
spec:
  containers:
  - name: redis
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

Here we bind the value `username` from the secret `mysecret` to the env variable `SECRET_USERNAME`,
`password` from the secret `mysecret` to the env variable `SECRET_PASSWORD`:

```yml
apiVersion: v1
kind: Pod
metadata:
  name: redis-with-env-secrets
spec:
  containers:
  - name: redis
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
```

Careful, if you change a secret after starting the pods, it won't update the pods. So you need to restart them.

## Clean up

```sh
kubectl delete service,deployment,pod,secrets --all
```

## Links

* https://kubernetes.io/docs/concepts/configuration/secret/
