# How to argoCD Connects to K8s and Configuring GitOps Repo 

- Go to gitlab.com and login.
- Create a new project called <"Name of your Project">. The project should be blank, private, and without a README file.* Go back to the terminal and run the following commands:

```bash
cd myapp
git remote add origin git@gitlab.com:[your username]/Name of your Project.git
git push -u origin --all
```

- Go back to Gitlab and refresh the page.
- Click on the profile picture and select preferences. Select peronsl access tokens.
- Create one called argo cd. The access level should be api .
- Copy the value of the token.
- On the terminal, create a secret by running the following command:

```bash
argocd repo add "https://gitlab.com/username/Name of your Project" --username " [your username]" --password "[your personal token]"
```

- Create the Argo CD directory by running the following commands:

```bash
mkdir argo-cd
cd argo-cd
```

---

# Creating the ArgoCD Bootstrap Application to manage himself and any other application

- Create a new YAML file called argocd.yaml and add the following:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: argocd
  namespace: argocd
spec:
  project: default
  source:
    repoURL: 'https://gitlab.com/[your username]/samplegitopsapp.git'
    path: argo-cd
    targetRevision: main
  destination:
    server: 'https://kubernetes.default.svc'
    namespace: argocd
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
```

- Apply the above configuration by running:
```bash
kubectl apply -f argocd.yaml
```

- Go to the UI of Argo CD.

- Login using the username of admin and your password