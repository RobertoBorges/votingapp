1 - kubectl create namespace cert-manager

2 - kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v0.13.1/cert-manager.yaml

3 - kubectl label namespace cert-manager cert-manager.io/disable-validation=true

4 - helm repo update

5 - # Install the cert-manager Helm chart
    helm install cert-manager --namespace ingress --version v0.13.0 jetstack/cert-manager

6 - Create a CA Cluster Issuer

apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: MY_EMAIL_ADDRESS
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx

7 - Create a Ingress for each service

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: result-ingress
  namespace: voting
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - result.kubepfe.com
    secretName: result-cert
  rules:
  - host: result.kubepfe.com
    http:
      paths:
        - path: /
          backend:
                serviceName: result-service
                servicePort: 80