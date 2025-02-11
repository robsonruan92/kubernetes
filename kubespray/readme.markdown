# Planejando um cluster de Alta Disponibilidade

Usando: kubespray, proxmox e terraform.

O principal motivo de estarmos usando o [kubespray](https://github.com/kubernetes-sigs/kubespray) é justamente o que a propria documentação dele diz “Deploy a Production Ready Kubernetes Cluster” 

Ele utiliza `ansible` para automação e oferece suporte a múltiplos provedores de nuvem e sistemas operacionais. 

Baixar o [repositório do kubespray](https://github.com/kubernetes-sigs/kubespray) 

```bash
git clone https://github.com/kubernetes-sigs/kubespray
```

Os principais arquivos dentro desse repositório que vale a pena pontuar:

- `cluster.yml` →Playbook que rodamos quando precisamos criar o cluster
- `remove-node.yml`→Playbook para remover o node
- `scale.yml` → Playbook para aumentar o cluster
- `reset.yml` →Playbook para resetar o cluster tudo

Dentro da pasta `inventory/sample/group_var`s nós temos como consultar alguns exemplos de como configurar um cluster. 

Dentro do group_vars é onde você pode configurar tudo do cluster.

Seguindo na documentação novamente, proximo passo para setar o cluster:

```bash
ansible-playbook -i inventory/mycluster/ -b  cluster.yml
```

Depois disso você pode consultar os tudo funcionando

```bash
# Listar os nodes
kubectl get node

#Listar os Pods
kubectl get po -A
```