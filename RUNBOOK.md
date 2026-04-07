# test0407 Dify + Ollama Deployment Runbook

## 1. Commit changes

```bash
cd /home/nutanix/test0407
git status
git add dify/
git commit -m "Add Dify Helm chart, templates, and deployment documentation"
git log -1 --oneline
```

## 2. Push to GitHub (HTTPS + PAT)

1. Create a PAT on GitHub (`yukinarai` account).
   - Recommended: fine-grained token
   - Repository: `yukinarai/test0407`
   - Permission: `Contents: Read and write`
2. Push:

```bash
cd /home/nutanix/test0407
GIT_TERMINAL_PROMPT=1 git push -u origin main
```

Username is your GitHub username, and password is PAT (not GitHub account password).

## 3. Configure kubeconfig

If `kubectl` or `helm` tries `localhost:8080`, kubeconfig is not set correctly.

```bash
export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf
kubectl get nodes
```

Optional persistence:

```bash
echo 'export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf' >> ~/.bashrc
source ~/.bashrc
```

## 4. Helm command by current directory

- If current dir is `.../test0407/dify`:

```bash
helm upgrade --install dify . -n dify --create-namespace
```

- If current dir is `.../test0407`:

```bash
helm upgrade --install dify ./dify -n dify --create-namespace
```

## 5. Cleanup failed/canceled release

```bash
export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf
helm list -n dify -a
helm status dify -n dify
helm uninstall dify -n dify
kubectl delete namespace dify --wait=true
```

## 6. Fresh deploy

```bash
cd /home/nutanix/test0407
export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf
helm upgrade --install dify ./dify -n dify --create-namespace --wait --timeout 15m
```

Verify:

```bash
helm list -n dify
kubectl get pods -n dify -o wide
```

## 7. Access check

```bash
kubectl get svc -n dify dify-traefik -o wide
curl -I http://<EXTERNAL-IP>/
curl -k -I https://<EXTERNAL-IP>/
curl -k -I https://<EXTERNAL-IP>/apps
```

Expected: `/apps` returns `200 OK`.

## 8. Install otwld/ollama-helm

```bash
export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf
cd /home/nutanix
git clone https://github.com/otwld/ollama-helm.git
cd /home/nutanix/ollama-helm
helm upgrade --install ollama . -n ollama --create-namespace --wait --timeout 15m
```

If the repository already exists:

```bash
cd /home/nutanix/ollama-helm
git pull
helm upgrade --install ollama . -n ollama --create-namespace --wait --timeout 15m
```

Verify:

```bash
helm list -n ollama
kubectl get pods -n ollama -o wide
kubectl get svc -n ollama -o wide
```

Expected:
- Helm status is `deployed`
- `ollama` pod is `Running`
- Service `ollama` is available on port `11434`

## 9. Pull model in Ollama pod

```bash
export KUBECONFIG=/home/nutanix/yukicluster-kubeconfig.conf
kubectl exec -it deploy/ollama -n ollama -- ollama pull 7shi/ezo-gemma-2-jpn:2b-instruct-q8_0
kubectl exec deploy/ollama -n ollama -- ollama list
```

Expected: `7shi/ezo-gemma-2-jpn:2b-instruct-q8_0` appears in model list.

## Security notes

- Never paste PAT/token into chat, docs, or screenshots.
- Revoke leaked PAT immediately and create a new one.
- kubeconfig may contain certificates/keys. Handle as sensitive data.
