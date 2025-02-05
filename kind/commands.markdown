# Useful Commands Cheat Sheet

| **Command**                                  | **Description**                                  |
|----------------------------------------------|--------------------------------------------------|
| `kind create cluster --name my-cluster`      | Create a single-node cluster.                   |
| `kind get clusters`                          | List all Kind clusters.                         |
| `kind delete cluster --name my-cluster`      | Delete a specific cluster.                      |
| `kind load docker-image my-app:latest`       | Load a Docker image into the cluster.           |
| `kubectl get nodes`                          | List all nodes in the cluster.                  |
| `kubectl config current-context`             | Check the current Kubernetes context.           |
|----------------------------------------------|-------------------------------------------------|

# A real example creating cluster:
`kind create cluster --config config.yaml --name projeto-devops-kubernetes --kubeconfig config`

# A real example destroying cluster:
`kind delete cluster --config config.yaml --name projeto-devops-kubernetes --kubeconfig config`