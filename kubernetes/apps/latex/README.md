# LaTeX web editor

## Choix retenu

Architecture recommandee: `code-server` avec l'extension VS Code LaTeX Workshop, une image custom contenant TeX Live, un PVC Longhorn pour `/home/coder`, et une exposition interne via Envoy Gateway `HTTPRoute`.

Ce choix est plus simple et plus maintenable qu'Overleaf CE pour un usage personnel:

- pas de MongoDB, Redis, file-store applicatif ou stack temps reel;
- projets LaTeX stockes comme des fichiers standards dans `/home/coder/projects`;
- Git utilisable directement depuis le terminal VS Code;
- backup simple du PVC;
- deploiement Kubernetes classique, compatible GitOps.

Alternatives ecartees:

- Overleaf CE: excellent pour la collaboration, trop lourd ici.
- VS Code tunnel / vscode.dev: plus dependant d'un service externe.
- IDE web generaliste type Theia: pas d'avantage net par rapport a code-server pour ce besoin.

## Image

Le Dockerfile est dans `images/code-server-latex/Dockerfile`.

Il installe:

- code-server;
- LaTeX Workshop;
- `pdflatex`, `xelatex`, `lualatex`;
- `latexmk`, `biber`, BibTeX;
- paquets TeX Live courants: francais, Beamer, TikZ, bibliographie, polices, sciences, publishers.

Les extensions sont installees dans `/opt/code-server/extensions`, pas dans `/home/coder`, pour rester disponibles meme quand le PVC masque le home de l'image.

Build et push:

```sh
docker build -t ghcr.io/axelfrache/code-server-latex:latest images/code-server-latex
docker push ghcr.io/axelfrache/code-server-latex:latest
```

Pour une production plus reproductible, remplacer `latest` par un tag versionne dans le Dockerfile de deployment.

## Acces

L'URL par defaut est:

```text
https://latex.${SECRET_LOCAL_DOMAIN}
```

La route utilise `envoy-internal`. Garder cette app interne, ou ajouter une protection externe avant toute exposition publique: Cloudflare Access, Authentik, Authelia, ou equivalent.

Le mot de passe code-server est stocke dans:

```text
kubernetes/apps/latex/code-server/app/secret.sops.yaml
```

Pour le lire ou le changer:

```sh
sops -d kubernetes/apps/latex/code-server/app/secret.sops.yaml
sops kubernetes/apps/latex/code-server/app/secret.sops.yaml
```

## Ressources

Valeurs deployees:

- requests: `100m` CPU, `512Mi` RAM;
- limits: `4` CPU, `8Gi` RAM;
- PVC: `20Gi`, `longhorn`.

Ces valeurs gardent le pod leger au repos tout en autorisant des compilations LaTeX lourdes ponctuelles. Pour de gros projets TikZ/Beamer ou de nombreuses polices, augmenter la limite memoire a `10Gi` ou `12Gi`.

## Backup

Un `CronJob` cree une archive quotidienne de `/home/coder` vers le NFS deja utilise dans le repo:

```text
192.168.1.20:/mnt/tank/backups/latex
```

Retention: 30 jours.

Verifier regulierement une restauration, par exemple dans un PVC temporaire. Si Longhorn snapshot/backup est active, garder ce CronJob comme backup lisible et simple, ou le remplacer par une politique Longhorn dediee.

## Securite LaTeX

La compilation LaTeX execute du code localement dans le conteneur. Points importants:

- ne compile pas de projets non fiables;
- garde `--shell-escape` desactive par defaut;
- evite d'installer des extensions VS Code non necessaires;
- garde le service derriere HTTPS et authentification;
- prefere une exposition interne via `envoy-internal`;
- surveille la consommation CPU/RAM pendant les compilations lourdes;
- considere une `NetworkPolicy` restrictive si le namespace devient public ou multi-utilisateur.

## Commandes de test

Verifier Flux et les ressources:

```sh
kubectl get ns latex
kubectl -n latex get kustomization,deploy,pod,pvc,svc,httproute
kubectl -n latex rollout status deploy/latex-code-server
kubectl -n latex logs deploy/latex-code-server
```

Verifier l'acces HTTP depuis le cluster:

```sh
kubectl -n latex port-forward svc/latex-code-server 8080:80
curl -I http://127.0.0.1:8080
```

Verifier les outils LaTeX:

```sh
kubectl -n latex exec deploy/latex-code-server -- pdflatex --version
kubectl -n latex exec deploy/latex-code-server -- xelatex --version
kubectl -n latex exec deploy/latex-code-server -- lualatex --version
kubectl -n latex exec deploy/latex-code-server -- latexmk --version
kubectl -n latex exec deploy/latex-code-server -- biber --version
```

Compiler un document minimal:

```sh
kubectl -n latex exec deploy/latex-code-server -- sh -ec 'mkdir -p /home/coder/projects/smoke && cat > /home/coder/projects/smoke/main.tex <<EOF
\documentclass{article}
\usepackage[french]{babel}
\usepackage{fontspec}
\begin{document}
Bonjour depuis Kubernetes.
\end{document}
EOF
cd /home/coder/projects/smoke
latexmk -xelatex -interaction=nonstopmode main.tex
test -s main.pdf'
```

Tester `pdflatex` sans `fontspec`:

```sh
kubectl -n latex exec deploy/latex-code-server -- sh -ec 'cat > /tmp/pdflatex-test.tex <<EOF
\documentclass{article}
\usepackage[T1]{fontenc}
\usepackage[utf8]{inputenc}
\usepackage[french]{babel}
\begin{document}
Bonjour.
\end{document}
EOF
cd /tmp && latexmk -pdf -interaction=nonstopmode pdflatex-test.tex && test -s pdflatex-test.pdf'
```
