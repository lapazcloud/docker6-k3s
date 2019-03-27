## Download Flash tool from Hypriot

```sh
curl -LO https://github.com/hypriot/flash/releases/download/2.3.0/flash
chmod +x flash
sudo mv flash /usr/local/bin/flash
```

## servidor

Connect your SD card

	flash --userdata ./user-data.yaml 
	
Insert the SD card on the RPi and boot the device.

Wait around 5 minutes.

	scp -P 4444 ops@servidor.local:~/kubeconfig ./
	sed -i'' -e "s/localhost/servidor.local/g" kubeconfig
	export KUBECONFIG=$PWD/kubeconfig
	kubectl get pods -o wide --all-namespaces

## servidor-worker

	sed -i'' -e "s/localhost/servidor.local/g" servidor-worker.yaml
	flash --userdata ./servidor-worker.yaml

## Deploy an Ingress

  	kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
	kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml

## Editing CoreDNS

  kubectl -n kube-system edit configmap coredns -oyaml

```yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        rewrite name git.servidor.local traefik-ingress-service.kube-system.svc.cluster.local
        rewrite name drone.servidor.local traefik-ingress-service.kube-system.svc.cluster.local
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . 1.1.1.1
        cache 30
        loop
        reload
        loadbalance
    }
```

# Deploy services

	  kubectl create -f storage-class.yaml
	
	  kubectl create ns gitea
	  kubectl create ns drone
	
	  kubectl create -f persistent-volumes.yaml
	
	  kubectl create -f volume-claims.yaml
	  kubectl create -f gitea.yaml
	  kubectl create -f drone.yaml
