# n8n su Kubernetes locale (Docker Desktop)

Scopo
- Guida rapida per inizializzare e verificare un'istanza di `n8n` in un cluster Kubernetes locale (es. Docker Desktop) usando i manifest presenti in questa cartella.

Prerequisiti
- Docker Desktop con Kubernetes abilitato (Windows/macOS).
- `kubectl` installato e configurato per il cluster locale.
- Controller Ingress `ingress-nginx` installato e funzionante (necessario per hostname custom).
- Privilegi di amministratore per modificare il file `hosts` locale.

Voci da aggiungere al file hosts (richiede permessi amministratore)
```text
127.0.0.1 n8n.kubernetes.local
127.0.0.1 kubernetes.docker.internal
```

Su Windows: apri il file hosts con Notepad come amministratore:
```pwsh
notepad C:\Windows\System32\drivers\etc\hosts
# aggiungi le righe sopra, salva e chiudi
ipconfig /flushdns
```

Su macOS/Linux (esempio):
```bash
sudo -- sh -c 'echo "127.0.0.1 n8n.kubernetes.local" >> /etc/hosts'
sudo -- sh -c 'echo "127.0.0.1 kubernetes.docker.internal" >> /etc/hosts'
sudo dscacheutil -flushcache  # macOS
sudo systemd-resolve --flush-caches  # Linux (se systemd-resolved)
```

Deploy (step-by-step)
1. Creare il namespace (idempotente):
```bash
kubectl create namespace n8n --dry-run=client -o yaml | kubectl apply -f -
```
2. Applicare la PVC:
```bash
kubectl apply -f n8n-pvc.yaml -n n8n
```
3. Applicare il Deployment:
```bash
kubectl apply -f n8n-deployment.yaml -n n8n
```
4. Applicare il Service:
```bash
kubectl apply -f n8n-service.yaml -n n8n
```
5. Applicare l'Ingress (assicurati che l'host usato nel manifest corrisponda a `n8n.kubernetes.local`):
```bash
kubectl apply -f n8n-ingress.yaml -n n8n
```

Verifiche rapide
```bash
kubectl get ingress -n n8n
kubectl get pods -n n8n
kubectl describe pod <pod-name> -n n8n
kubectl logs -f <pod-name> -n n8n
kubectl get events -n n8n
```

Accesso
- Se l'Ingress è configurato per l'host `n8n.kubernetes.local`, apri `http://n8n.kubernetes.local` nel browser dopo aver aggiunto le voci al file hosts.
- In alternativa (se non usi ingress) puoi esporre con port-forward:
```bash
kubectl port-forward svc/n8n 30078:5678 -n n8n
# poi apri http://localhost:30078
```

Ingress controller
- Assicurati che `ingress-nginx` sia installato. Esempio rapido (seguire la documentazione ufficiale per produzione):
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller/main/deploy/static/provider/kind/deploy.yaml
```

Troubleshooting comune
- Pod in `CrashLoopBackOff`: esegui `kubectl describe pod <pod> -n n8n` e `kubectl logs <pod> -n n8n`.
- Forzare il riavvio dei pod:
```bash
kubectl delete pod -n n8n -l app=n8n
```
- Verificare rollout del deployment:
```bash
kubectl rollout status deployment/<deployment-name> -n n8n
```

Rimozione immagine locale (se necessario)
```bash
docker images --filter "reference=n8nio/n8n*"
docker rmi <IMAGE_ID>
```

Note di sicurezza e comportamento
- Non eseguire comandi distruttivi senza conferma (es. rimozione volumi o immagini).
- Modifiche ai manifest: preferire `kubectl apply` e patch idempotenti; mostrare sempre un diff minimo prima di applicare modifiche importanti.
