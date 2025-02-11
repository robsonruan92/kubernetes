# Pré-requisitos para rodar o cluster

[Habilitar o 'foward' de pacotes IPV4](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional)

```bash
# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system

#Verify that net.ipv4.ip_forward is set to 1 with
sysctl net.ipv4.ip_forward
```

Agora para configurar o containerd (apenas ele) nas VMs que é o novo padrão [(Link)](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)

```bash
# Add Docker's official GPG key:

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

#Install the Docker packages.

sudo apt-get update
sudo apt-get install containerd.io -y
```

Arquivo de configuração padrão do containerd, basicamente pegar o arquivo generico do containerd e jogar dentro do `config.toml`

```bash
containerd config default > /etc/containerd/config.toml
```

Desabilitar o SWAP, verificar no fstab se caso remover, depois só rodar um `swapoff -a`

```bash
#Verify swap

cat /proc/swaps
cat /etc/fstab
```

[Instalar kubeadm, kubelet and kubectl](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

# Criação do cluster Kubernetes

Seguindo agora com a documentação ([link](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)), mas você pode usar o comando `kubeadm init --help` para obter mais informações de parâmetros que podem ser passados para uma melhor configuração.

Vale uma menção aos abaixo:

- `--apiserver-advertise-address` → Setar/Forçar a utilização de uma interface de rede específica
- `--pod-network-cidr` →Definir o tamanho da rede (range)

Para seguir nossa configuração precisamos entender da necessidade do CNI e para isso leia abaixo a explicação.

- Conteiner Network Interface (CNI)
    
    Com isso dentro da infraestrutura do kubernetes temos um rede especifica para os pods que irão receber um endereço IP. Esse mesmo endereço IP é gerenciado por um recurso externo ao cluster do k8s, o **CNI** (conteiner network interface).
    
    - CNI = Plugin que irá entregar rede para seu cluster kubernetes
    
    Na documentação o trecho [*Installing a Pod network add-on*](https://kubernetes.io/pt-br/docs/concepts/cluster-administration/addons/#rede-e-pol%C3%ADtica-de-rede) podemos ver algumas opções de CNIs para instalar e configurar, a mais comum pode ser a [calico](https://docs.projectcalico.org/latest/introduction/) (entrega um firewall a nivel dos pods) porém a mais simples é a [Flannel](https://github.com/flannel-io/flannel#deploying-flannel-manually).
    
    Mas cada uma tem sua particularidade, e tem que seguir algumas regras, por exemplo:
    
    - Caso você faça um deploy de um cluster EKS na AWS, a cni será a própria da AWS, no caso a [amazon-vpc-cni-k8s](https://github.com/aws/amazon-vpc-cni-k8s)**.** Por ser algo nativo da AWS ele consegue usar recursos da propria AWS.
        - Criar intefaces de rede, security groups e etc.

Com isso iremos optar pelo CNI Flannel, o qual já tem predefinido qual o CIDR com o qual iremos trabalhar, no caso o `10.244.0.0/16` e com isso passaremos como parametro no comando de configuração do nosso cluster. 

```bash
# kubeadm init --apiserver-advertise-address <range_local> --pod-network-cidr <range_lda_flannel>

kubeadm init --apiserver-advertise-address 192.168.10.10 --pod-network-cidr 10.244.0.0/16

```

Após a execução ele deve retornar instruções de como configurar seu comando de `kubectl` e o comando necessário para colocar nos workers para entrarem no cluster, algo como `kubeadm join <IP:6443> token token-ca`

IMPORTANTE !!! 

Sempre que tiver a mensagem `the connection to the server localhost was refused - did you specify the right host or port` quase sempre é porque não foi feita a configuração do `KUBECONFIG` e ele fica tentando a configuração padrão.

Para resolver isso:

```bash
# Exportar a variavel

export KUBECONFIG=/etc/kubernetes/admin.conf
```

Voltando para nossa configuração, agora precisamos incluir os workers no nosso cluster usando o comando `kubeadm join ...` 

Agora por fim possivelmente temos alguns nodes ainda como peding (coredns) isso porque ainda não definimos o CNI, para resolver isso bastar seguir a documentação e rodar o comando para deployar o flannel dentro do cluster, e lembrando é no control plane ou “node master”.

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Agora temos o cluster entregue e funcional.