# Ingress

To use ingress with minikube, you first need to enable the `ingress` addon once.

```bash
minikube addons enable ingress
```

Once the addon is enabled, you will have to start a tunnel between minikube and
the host.

```bash
minikube tunnel
```

Once done, you can apply the manifest files to your minikube cluster.

```bash
kubectl apply -f .
```

This project contains a basic nginx application, replicated twice, that you should
be able to reach on your browser through `http://localhost:8080`.

# How to test

To test everything works fine, you can use this Python one-liner :

```bash
python -c "import requests; print(requests.get('http://localhost', headers={'Host': 'localhost'}).text)"
```

Or if you can access a shell :
```bash
curl -i http://localhost -H "Host: localhost"
```

Note that the header `Host` must match your ingress `host` rule. For instance,
if the ingress manifest looked like this :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
      - host: sub.my-domain.io
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: nginx-service
                  port:
                    number: 8080
```

The `Host` header must be `sub.my-domain.io`, and the request would look like
this :

```bash
python -c "import requests; print(requests.get('http://localhost', headers={'Host': 'sub.my-domain.io'}).text)"
```

Or using cURL :

```bash
curl -i http://localhost -H "Host: sub.my-domain.io"
```

Do not change `localhost` though, as it's your minikube entrypoint, which will
then forward the request to your ingress.

# TLS setup

If you want to setup TLS, you can uncomment the following in the `nginx.ingress.yaml`:

```yaml
tls:
  - hosts:
      - localhost # your actual domain name
    secretName: nginx-secret
```

and edit the `nginx.secret.yaml` to fill both the public and private key of your
TLS context.

You will then be able to reach your application through https. Be careful if
you're using self-signed certificates, as cURL, requests and browsers are going
to warn their users that the certificate is self-signed and might be harmful.

If you know what you're doing, you can use the `-k` option in curl, `verify=False`
argument in Python requests, or click the `Accept the risks` button in your
browser.

# /etc/hosts (it should work but it won't for me lol)

If you don't want to go through all of that headers managements, and be able to
access your domain name through your web browser, you could do something like
this :

### 1. Get your ingress IP address

Once you applied your ingress resource to your cluster, it will have a given IP
address. You can catch that IP using the following command :

```bash
kubectl get ingress <your-ingress-name>
```

Here is the output for this use case :
```
nginx-ingress   nginx   sub.my-domain.io   192.168.49.2   80      60m
```

You can now copy that IP address.

### 2. /etc/hosts

If you are on a Linux machine, the file we will look into is located to the
`/etc/hosts` path.

If you are on a Windows machine, the file we will look into is located to the
`C:\Windows\System32\drivers\etc\hosts`

You will have to open either one in sudo / administrator mode, as we will edit
it.

Once opened, here is an example of what you may see :

```
127.0.0.1       localhost
::1             localhost
```

We will add the ingress IP address and map it to the given domain name :


```
127.0.0.1       localhost
::1             localhost
192.168.49.2    sub.my-domain.io
```
