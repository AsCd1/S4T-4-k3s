# S4T-4-k3s
Di seguito sonno fornite diverse guide:
- una configurazione veloce di k3s, vedi [K3s Rapid Setup](#k3s-rapid-setup-)
- una configurazzione completa, vedi [K3s Multi-Cluster Setup Guide](#k3s-multi-cluster-setup-guide-)

## K3s Deployment Guide

| **K3s Deployment Guide** | **Steps**                                 |
|--------------------------|-------------------------------------------|
| 📁 **K3s-S4T Rapid Setup ⚡** | 1. 🚀 Installazione di K3s             |
|                          | 2. 🔗 Clonazione S4T            |
|                          | 3. 📌 Deploy su Kubernetes                |
|                          | 4. ✅ Verifica dei Pod e dei Servizi      |
|                          |                                           |
| 📁 **K3s-Calico-MetalLB-Istio-S4T Multi-Cluster Setup** | 1. ⚙️ Installazione di K3s (senza Traefik) |
|                          | 2. 🌐 Configurazione di Calico            |
|                          | 3. 📡 Setup di MetalLB                    |
|                          | 4. 🚀 Deploy di Istio                    |
|                          | 5. 🔗 Clonazione S4T        |
|                          | 7. 📌 Deploy su Kubernetes                |
|                          | 8. ✅ Verifica dei Pod e dei Servizi      |




# K3s Rapid Setup ⚡
- [Quick-Start-K3s](https://docs.k3s.io/quick-start)

## 🚀 Installazione di K3s (Master Node Unico)
```bash
curl -sfL https://get.k3s.io | sh -
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get nodes
```
🔹 Cosa fa questo comando?
- Installa K3s come master node
- Avvia automaticamente il servizio
- ⚠️ Nota: Configura kubectl per gestire il cluster
 S4T - Stack4Things Deployment 
- ⚠️ Nota: In questa configurazione non sono presenti Calico, istio e MetalLb necessari per alcuni esempi
- ⚠️ Con questa configurazione si otterrà S4T con servizi interni al cluster in una configurazione minimale ma configurabile a piacere.

## [Muoversi direttamente alla sezione S4t](#-s4t---stack4things-deployment)

# K3s Multi-Cluster Setup Guide 🛠
Con la seguente guida si otterrà una configurazione con:
1. Calico come CNI
2. MeatalLB come Loadbalancer
3. Istio come Gateway
4. S4T

## Guida al Setup di un Cluster K3s

Questa guida ti aiuterà a configurare un cluster K3s utilizzando una macchina come **server** (control plane) e una o più macchine come **worker nodes**.

### Nota:
1. Se preferisci eseguire un'installazione pulita di Calico, salta alla sezione dedicata a Calico più avanti.
2. Potrebbe essere necessario utilizzare gli IP interni delle macchine. (LASCIARE?)

## 1. Installazione del Server K3s

### Configurazione del Server (Control Plane)

1. **Accedi alla VM che fungerà da server**, chiamato anche *server host*. Utilizza SSH per connetterti alla macchina.

2. **Esegui il seguente comando** per installare K3s sul server:

    ```bash
    curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --disable traefik --disable servicelb" sh -
    ```

    Questo comando installerà K3s sul server host e disabiliterà Traefik e il bilanciamento del servizio (service load balancer).

## 2. Recupera il Token del Nodo

Dopo aver installato K3s sul server, recupera il **token del nodo** che sarà utilizzato per aggiungere i worker nodes al cluster.

1. **Esegui il seguente comando** sul server per ottenere il token:

    ```bash
    cat /var/lib/rancher/k3s/server/node-token
    ```

    oppure:

    ```bash
    sudo cat /var/lib/rancher/k3s/server/node-token
    ```

2. Il **token** che otterrai servirà per il comando successivo = `YourToken`.

## 3. Recupera il Certificato del Cluster

Per poter comunicare con il cluster, è necessario recuperare il certificato di configurazione di K3s.

1. **Esegui il seguente comando** per ottenere il file di configurazione:

    ```bash
    cat /etc/rancher/k3s/k3s.yaml
    ```

    oppure:

    ```bash
    sudo cat /etc/rancher/k3s/k3s.yaml
    ```

2. **Salva il contenuto del file `k3s.yaml`** sul tuo computer nella directory `~/.kube/` come un file di configurazione personalizzato (`<nome del tuo file>.yaml`).

3. **Modifica il file** sostituendo l'IP del server con l'IP corretto del control plane (Server Host) e assicurati che il server utilizzi HTTPS.

    Esempio di modifica del file `k3s.yaml`:
    ```yaml
    server: https://<Contol Plane IP>:6443
    ```
4. **Verifica la connessione al cluster**:

   ⚠️ Configura il tuo ambiente:
    ```bash
    export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
    ```

    Esegui il comando:
    ```bash
    kubectl get nodes
    ```
   
    Esempio di output:

    ```
    NAME            STATUS  ROLES                   AGE VERSION
    ubuntuserver    Ready   control.plane,master    xx  xx
    ```

## 4. Aggiungi i Worker Node

Ora puoi aggiungere i nodi di lavoro (worker nodes) al cluster.

### Aggiungi un Nodo Worker

1. **Accedi alla VM che fungerà da worker node**. Questo è il nodo che eseguirà i carichi di lavoro, ed è diverso dal nodo server.

2. **Esegui il seguente comando** sul nodo worker:

    ```bash
    curl -sfL https://get.k3s.io | K3S_URL=https://<Contol Plane IP>:6443 K3S_TOKEN=<YourToken> sh -
    ```

    Sostituisci `<Contol Plane IP>` con l'IP del server (control plane) e `<YourToken>` con il token che hai ottenuto in precedenza.


5. **Configurazione di `kubectl` sul Worker Node**:

    - Crea un file `~/.kube/config` sul nodo worker.
    - Copia il contenuto del file `k3s.yaml` dal server (control plane) in questo file.
    - Modifica il campo `server` per includere l'IP del server.

6. **Configura `kubectl`**:

    Esegui i seguenti comandi sul nodo worker per impostare il tuo ambiente Kubernetes:

    ```bash
    export KUBECONFIG=~/.kube/config
    kubectl get nodes
    ```

    Esempio di output:

    ```
    NAME            STATUS  ROLES                   AGE VERSION
    ubuntuserver    Ready   control.plane,master    xx  xx
    ubuntuworker    Ready   <none>                  xx  xx
    ```

---

Con questi passaggi avrai un cluster K3s funzionante con un server e uno o più nodi worker.

## Calico
### Comandi per installare l'operatore Calico:

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/tigera-operator.yaml
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.29.2/manifests/custom-resources.yaml
kubectl get nodes -o wide  # Da eseguire anche sul worker --opzionale
kubectl get pods -n calico-system -o wide
ip route show
kubectl get pods -A | grep -E "calico|flannel" #flannel è presente di default se non si segue l'installazione pulita
```

## Installing Helm   
Il [progetto Helm](https://helm.sh/docs/intro/install/) offre due metodi ufficiali per scaricare e installare Helm. Oltre a questi, la community di Helm fornisce anche altri metodi di installazione tramite diversi gestori di pacchetti.

### 🚀 Installazione tramite Script  
Helm fornisce uno script di installazione che scarica e installa automaticamente l'ultima versione di Helm sul tuo sistema.
  
Puoi scaricare lo script ed eseguirlo localmente. È ben documentato, quindi puoi leggerlo in anticipo per capire cosa fa prima di eseguirlo.
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Install Cert-Manager  
 
- Questo passaggio è necessario solo se devi utilizzare certificati emessi dalla CA generata da Rancher (ingress.tls.source=rancher) o richiedere certificati emessi da Let's Encrypt (ingress.tls.source=letsEncrypt).

```bash
# Set the KUBECONFIG environment variable
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

# Apply the Cert-Manager Custom Resource Definitions (CRDs)
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.crds.yaml

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install Cert-Manager using Helm
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace

# Verify Cert-Manager pods are running
kubectl get pods --namespace cert-manager
>> OUTPUT: 3 PODS IN RUNNING STATE

# Check installed Custom Resource Definitions (CRDs)
kubectl get crds | grep cert-manager
>> OUTPUT: 6 cert-manager CRDs found
```

## MetalLB
- [Documentazione qui](MetalLB/README.md)

### Installazione con Helm
```bash
# Aggiungi il repository Helm di MetalLB
helm repo add metallb https://metallb.github.io/metallb

# Installa MetalLB
helm install metallb metallb/metallb
```

### Configurazione Layer2
Creazione del file di configurazione per Layer 2:
- [metallb-configuration](./MetalLB/metallb-configuration.yaml)
```bash
echo "Apri il file di configurazione: https://github.com/AsCd1/K3s-Configuration/blob/main/MetalLB/metallb-configuration.yaml"
nano metallb-configuration.yaml
```
⚠️ **Importante:** Durante l'applicazione della configurazione si è verificato un errore risolto con un secondo apply:
- Creazione del file [L2Advertisement](./MetalLB/l2advertisement.yaml) separato.
```bash
nano l2advertisement.yaml
```
Apply della configurazione:
```bash
kubectl apply -f l2advertisement.yaml
kubectl apply -f metallb-configuration.yaml
```
Verifica della configurazione:
```bash
kubectl get ipaddresspools -n metallb-system
>> Output: Lista dei range IP

kubectl get l2advertisements -n metallb-system
>> Output: Nome e range IP
```

## 🔹 Istio Install with Helm  

🔗 **Guida ufficiale**: [Istio Helm Installation](https://istio.io/latest/docs/setup/install/helm/)  

#### 📌 Aggiunta del repository Helm di Istio  
```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
>> Output atteso: "istio" has been added to your repositories
```

### 📌 Aggiornamento dei repository
```bash
helm repo update
>> Output atteso: Update Complete. Happy Helming!
```

### 📌 Installazione della base di Istio
```bash
helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace
>> Output atteso:
- NAME: istio-base
- LAST DEPLOYED: Tue Feb 25 09:19:24 2025
- NAMESPACE: istio-system
- STATUS: deployed
- REVISION: 1
- TEST SUITE: None
- NOTES:
- Istio base successfully installed!
```

### 📌 Verifica dello stato di istio-base
```bash
helm status istio-base -n istio-system
helm get all istio-base -n istio-system
helm ls -n istio-system
```
### 📌 Installazione del servizio istiod
```bash
helm install istiod istio/istiod -n istio-system --wait
```

### 📌 Verifica dell'installazione
```bash
helm ls -n istio-system
helm status istiod -n istio-system
```

### 📌 Controllo dello stato dei pod di istiod
```bash
kubectl get deployments -n istio-system --output wide
>> Output atteso:
NAME     READY   UP-TO-DATE   AVAILABLE   AGE  CONTAINERS  SELECTOR
istiod   1/1     1            1           23m  discovery   istio=pilot
```

### 📌 Creazione dello spazio dei nomi per il gateway
```bash
kubectl create namespace istio-ingress
>> Output atteso: namespace/istio-ingress created
```

### 📌 Installazione del gateway di Istio
```bash
helm install istio-ingress istio/gateway -n istio-ingress --wait
```

### 📌 Verifica dei servizi
```bash
kubectl get svc -A
>> Output atteso: Istio ha creato il suo LoadBalancer.
```

## 🎯 Cosa abbiamo ottenuto  

### 📌 Verifica dei pod di Istio Ingress  
```bash
kubectl get pods -n istio-ingress
>>OUTPUT atteso:
NAME                             READY   STATUS
istio-ingress-<PodID>   1/1     Running
```

### 📌 Verifica del Service di Istio Ingress
```bash
kubectl get svc -n istio-ingress
>> OUTPUT atteso:
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)
istio-ingress   LoadBalancer   x.x.x.x         x.x.x.x         15021:30268/TCP,80:31240/TCP,443:32410/TCP
```



## Kompose
### 🔹 Installation

Kompose è rilasciato tramite GitHub:

```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.35.0/kompose-linux-amd64 -o kompose

chmod +x kompose
sudo mv ./kompose /usr/local/bin/kompose

kompose version
>> 1.35.0
```

## 🚀 S4T - Stack4Things Deployment

Questa guida descrive come clonare, configurare e avviare **Stack4Things** su Kubernetes.
## 📌 **1. .zip**  -- OPZIONE 2 DISPONIBILE
### 📂 Contenuto della Cartella S4T

All'interno della cartella troverai:
- [**ComposeDeployment**](./S4T/ComposeDeployment)
    - **`deployments/`** → Contiene i file YAML per la definizione dei **Pod**, **Deployment** e **Service** di S4T.
    - **`storage/`** → Definizioni di **PersistentVolumeClaim (PVC)** per la gestione dei dati.
    - **`.env`** → File con le variabili d’ambiente necessarie per l'installazione.  
    - **`configmaps/`** → Configurazioni personalizzate per i servizi di S4T in Kubernetes.
- [ConfigurazioneIstio](./S4T/istioconf)
    - - **`istio/`** → Configurazioni di **Istio** per il bilanciamento del traffico e il gateway di accesso.

### 🚀 **Come Utilizzare i File**
1. **Estrarre la cartella ZIP** sul proprio sistema.
2. **Accedere alla cartella**
3. Applicare i file YAML al cluster Kubernetes:
```bash
Kubectl apply -f .
```
4. Verificare che i Pod siano attivi:
```bash
kubectl get pods
```
5. Verificare i servizi disponibili:
```bash
kubectl get svc
```

### 🛠 4. Creazione del Gateway e VirtualService per Istio -- VALIDO PER ENTRAMBE LE OPZIONI
- 📁 Definizione file yaml [qui](./S4T/istioconf)

Creiamo una cartella per i file di configurazione di Istio:
```bash
mkdir istioconf
```

Apriamo un nuovo file per definire il Gateway e il VirtualService:
```bash
nano gateway-virtualservice-istio.yaml
kubectl apply -f .
```

Verifichiamo che le risorse siano state create correttamente:
```bash
kubectl describe virtualservice iotronic-ui
```

### 📡 5. Controllo del Servizio Istio-Ingress
Verifichiamo il servizio istio-ingress per ottenere l'IP pubblico del bilanciatore di carico:
```bash
kubectl get svc istio-ingress -n istio-ingress
```
🔎 Esempio di output:
```bash
NAME            TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                                      AGE
istio-ingress   LoadBalancer   10.x.x.x      x.x.x.x         15021:30152/TCP,80:31152/TCP,443:30936/TCP   3d3h
```

Verifichiamo la creazione del VirtualService:
```bash
kubectl get virtualservice
```

🔎 Esempio di output:
```bash
NAME          GATEWAYS                  HOSTS   AGE
iotronic-ui   ["iotronic-ui-gateway"]   ["*"]   11m
```

Controlliamo il Gateway:
```bash
kubectl get gateway
```

🔎 Esempio di output:
```bash
NAME                  AGE
iotronic-ui-gateway   12m
```

### 🌍 6. Test dell’accesso al servizio
Utilizziamo curl per testare l'accesso alla UI di Iotronic tramite l'IP di istio-ingress:
```bash
curl x.x.x.x/iotronic-ui
```

🔎 Output atteso:
```bash
>> Apache Default Page
```

🔄 7. Configurare il Port Forwarding --opzionale tramite tailscale
Per esporre il servizio localmente:
```bash
kubectl port-forward --address 0.0.0.0 svc/istio-ingress 8100:80 -n istio-ingress
```

Ora possiamo accedere alla UI da un browser utilizzando gli indirizzi seguenti:
```bash
>> http://x.x.x.x:8100/iotronic-ui
>> http://x.x.x.x:8100/horizon/auth/login/?next=/horizon/
```
