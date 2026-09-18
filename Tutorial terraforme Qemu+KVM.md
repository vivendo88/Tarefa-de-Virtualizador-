

## Configuração do Ambiente e Instalação do Terraform e KVM

## 1. Atualização e Instalação de Dependências

Realize o update dos pacotes da VM e instale as ferramentas necessárias:




```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install -y curl unzip gnupg software-properties-common git python3-pip python3-venv

```

## 2. Adição do Repositório HashiCorp

Adicione o repositório oficial da HashiCorp para instalar o Terraform:

```bash
wget -O- [https://apt.releases.hashicorp.com/gpg](https://apt.releases.hashicorp.com/gpg) | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] [https://apt.releases.hashicorp.com](https://apt.releases.hashicorp.com) $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

```

## 3. Instalação e Verificação do Terraform

Atualize os repositórios e instale o Terraform:

```bash
sudo apt-get update && sudo apt-get install terraform -y

```

Verifique a versão instalada:

```bash
terraform -version

```

## 4. Instalação e Configuração do QEMU/KVM

Instale o virtualizador e seus componentes, habilite o serviço e adicione seu usuário ao grupo de libvirt:

```bash
sudo apt-get install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $USER && newgrp libvirt

```

Gere a chave SSH:

```bash
ssh-keygen -t rsa -b 4096 -N "" -f /root/.ssh/id_rsa

```

## 5. Criação e Execução do Terraform

Crie um arquivo chamado `main.tf`:

```bash
nano main.tf

```

(Nota: Escreva o código do arquivo `Main.tf_1_VM` para criar 1 VM ou `Main.tf_2_VM` para criar 2 VMs).

Valide a sintaxe e os argumentos do arquivo:

```bash
terraform validate

```

Simule a execução:

```bash
terraform plan

```

Execute o arquivo (digite `yes` quando solicitado):

```bash
terraform apply

```

## 6. Gerenciamento e Acesso às VMs

Visualize as máquinas criadas:

```bash
virsh list --all

```

Liste o IP de todas as VMs criadas:

```bash
virsh -c qemu:///system net-dhcp-leases qemu_cluster_net

```

Acesse a VM via SSH:

```bash
ssh ubuntu@<IP_DO_OUTPUT>

```

---

## Observações e Resolução de Problemas

### Erro de Permissão no QEMU/KVM

Caso o seguinte erro ocorra durante a criação:

> `libvirt_domain.k8s_node: Creating...`
> `Error: error creating libvirt domain: internal error: process exited while connecting to monitor: ... Could not open '/var/lib/libvirt/images/ubuntu-22.04-base.qcow2': Permission denied`
> `with libvirt_domain.k8s_node, on main.tf line 69`

Altere as permissões do diretório de imagens:

```bash
chmod 755 /var/lib/libvirt/images
chown -R libvirt-qemu:kvm /var/lib/libvirt/images
chmod 644 /var/lib/libvirt/images/*.qcow2

```

Edite o arquivo de configuração do libvirt:

```bash
nano /etc/libvirt/qemu.conf

```

Descomente e edite as linhas para ficarem assim:

```text
security_driver = "none"
user = "root"
group = "root"

```

Reinicie o serviço do libvirt para aplicar as alterações:

```bash
systemctl restart libvirtd

```

```

Se preferir, posso salvar este conteúdo formatado em um novo arquivo `.md` ou `.txt` para você baixar!

```
