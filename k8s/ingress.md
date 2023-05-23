## ingress
### 使用yaml 创建ingress
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-test2
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - http:
        paths:
          - path: /n2
            pathType: Prefix
            backend:
              service:
                name: nginx2
                port:
                  number: 80
          - path: /n3
            pathType: Prefix
            backend:
              service:
                name: nginx2
                port:
                  number: 80

  ingressClassName: nginx

```