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
python -c "import requsts; print(requests.get('http://localhost', headers={'Host': 'localhost'}).text)"
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
python -c "import requsts; print(requests.get('http://localhost', headers={'Host': 'sub.my-domain.io'}).text)"
```

Or using cURL :

```bash
curl -i http://localhost -H "Host: sub.my-domain.io"
```

Do not change `localhost` though, as it's your minikube entrypoint, which will
then forward the request to your ingress.
