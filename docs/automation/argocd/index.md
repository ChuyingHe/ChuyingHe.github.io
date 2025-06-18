
# GitOps

[reference](https://www.youtube.com/watch?v=f5EpcWp0THw&ab_channel=TechWorldwithNana)

!!! info "X as Code"
    - Infrastructure as Code
    - Network as Code
    - Policy as Code
    - Configuration as Code
    - Security as Code


How it works?

1. **X as Code** on Git repo
2. Engineer create the <u>Pull Request</u>
3. Run **CI Pipeline**:
    - validates the manifest files
    - run automated tests
4. Another Engineer review and approve <u>Pull Request</u>
5. **CD Pipeline** deployes the changes to the Infrastructure --> instead of running `kubectl apply -f xxx.yaml` on your local computer



## Two Deployment Methods

- **Push Deployment**: tranditional, the Pipeline executes command and deploys the new version to the Infrastructure 
    - Jenkins, Gitlab CICD
- **Pull Deployment**: an **Agent** installed in the Infrastructure(such as a K8S cluster), and it actively pulls changes from Git Repo. The **Agent** compares <u>desired state (in git repo)</u> and the <u>actual state(in Infrastructure)</u>
    - FluxCD
    - ArgoCD

<img src="./imgs/gitops.png" />

## Advantages

- easy rollback
- single source of truth
- increases security by limiting access: 
    - CD Pipeline deploys the changes, NOT any team member
    - team member only <u>propose</u> changes


# ArgoCD

[reference](https://www.youtube.com/watch?v=MeU5_k9ssrs&pp=ygUGYXJnb2Nk)


## How does CD work?
How does Continual Delivery(CD) Tool like **Jenkins** and **GitlabCICD** work?


<img src="./imgs/cicd_without_argo.png" />


Disadvantages:
1. need to install & steup tools like `kubectl`
2. configure cluster credential to **Jenkins** - security issue
3. once deployed, **Jenkins** doesn't know the deployment status

--> ArgoCD is targeting to improve the problematic CD process


## How does ArgoCD help
ArgoCD reverse the flow - use **Pull Deployment** instead of **Push Deployment**