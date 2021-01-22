### Maybe not required.......
```bash
chmod g+r /etc/rancher/k3s/k3s.yaml
chown root:wheel /etc/rancher/k3s/k3s.yaml

scp 10.133.1.10:/etc/rancher/k3s/k3s.yaml ~/.kube/config
sed -i 's/127.0.0.1/10.133.1.10/g' ~/.kube/config
```

```bash
echo "[ INFO ] install k3s"
echo "[ INFO ] getting k3s config from github repo"
mkdir -p /etc/rancher/k3s
touch /etc/rancher/k3s/config.yaml
# curl -sfLo /home/$user/.ssh/authorized_keys github.com/$user.keys

echo "[ INFO ] install k3s"
systemctl disable firewalld
systemctl stop firewalld

echo "[ INFO ] starting k3s installation"
tlsSAN="k3s.dus.waigant.io"
clusterDomain="cluster.k3s.dus.waigant.io"
curl -sfL https://get.k3s.io | sh -s - --disable=traefik --tls-san ${tlsSAN} --cluster-domain ${clusterDomain} --write-kubeconfig-mode 0644
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

echo "[ INFO ] installing helm"
curl -sfL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | sh -s -

# echo "[ INFO ] installing ingress-controller traefik"
# https://medium.com/@fache.loic/k3s-traefik-2-9b4646393a1c
# https://artifacthub.io/packages/helm/traefik/traefik
# helm repo add traefik https://helm.traefik.io/traefik
# helm repo update
# kubectl create namespace traefik
# helm install traefik traefik/traefik -n traefik

echo "[ INFO ] installing ingrex-controller nginx"
# https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
kubectl create namespace ingress-nginx
helm install latest ingress-nginx/ingress-nginx --namespace ingress-nginx

echo "[ INFO ] installing cert-manager"
# https://cert-manager.io/docs/installation/kubernetes/#installing-with-helm
helm repo add jetstack https://charts.jetstack.io
helm repo update
kubectl create namespace cert-manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.1.0  --set installCRDs=true 
#--set 'extraArgs={--dns01-recursive-nameservers-only,--dns01-recursive-nameservers=1.1.1.1:53,1.0.0.1:53}'

# curl -sfL https://cert-manager.io/docs/tutorials/acme/example/staging-issuer.yaml    | sed 's/Issuer//' | sed 's/user@example.com/sergei@waigant.com/g' | kubectl create -n cert-manager -f -
# curl -sfL https://cert-manager.io/docs/tutorials/acme/example/production-issuer.yaml | sed 's/user@example.com/sergei@waigant.com/g' | kubectl create -n cert-manager -f -
kubectl apply -f k8s.letsencrypt.clusterissuer.yml

echo "[ INFO ] installing kubernetes-dashboard"
# https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
GITHUB_URL=https://github.com/kubernetes/dashboard/releases
VERSION_KUBE_DASHBOARD=$(curl -w '%{url_effective}' -I -L -s -S ${GITHUB_URL}/latest -o /dev/null | sed -e 's|.*/||')
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/${VERSION_KUBE_DASHBOARD}/aio/deploy/recommended.yaml

kubectl -n kubernetes-dashboard apply -f k8s.dashboard.ingress.yml

# Create Admin user and retrieve login token
kubectl create -f k8s.dashboard.admin-user.yml
kubectl -n kubernetes-dashboard describe secret admin-user-token | grep ^token | awk '{print $2}'

echo "[ INFO ] installing Argo CD"
# https://argoproj.github.io/argo-cd/getting_started/
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl apply -n argocd -f k8s.argocd.ingress.yml

```
