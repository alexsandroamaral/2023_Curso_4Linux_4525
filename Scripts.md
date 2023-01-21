![rundeck](rundeck.png)

**Script's Utilizados no Treinamento de Rundeck**
===
Repositório de Chaves
---

Criação do par de chaves SSH em um host remoto atarvés do **Vagrant**:
```bash
# vagrant ssh "HOSTNAME"
sudo -i
mkdir keys
ssh-keygen -m PEM -N '' -f keys/rundeck
```

Parâmetros Utilizados:

  - -m: Define o formato da chave, o formato **PEM** é obrigatório, as versões mais novas do comando geram uma chave **SSH** incompatível.
  - -N: informa qual a senha da chave, no exemplo foi criado uma chave sem senha.
  - -f: informa onde o arquivo será gerado e qual será seu nome.


Estrutura do Rundeck:
---

Arquivos de configuração:
    
**/etc/rundeck:**
  
  - real.properties: Lida com usuários locais, senhas e "papéis" (roles).
  - .aclpolicy: Os arquivos terminados com .aclpolicy lidam com as regras de acesso dos usuários, também é possível definí-las através do painel.
  - framework.properties: Temos as configurações dos diretórios utilizados pelo Rundeck.
  - rundeck-config.properties: Configurações relacionadas ao funcionamento interno do **Rundeck**.


**/var/lib/rundeck:**
  - Dados variáveis do rundeck como log's de execução da tarefa, projetos ou banco de dados local.

  **Nodes:**
  ---
  Criação de diretórios necessários para **Lab**, versões mais recentes do **Rundeck** não utiliza mais:

  ```
# vagrant ssh automation
sudo -i
mkdir -p /var/lib/rundeck/projects/infra-agil/
chown -R rundeck: /var/lib/rundeck/projects/
```

**Exemplo de arquivo nodes.yml para inventário dos hosts a serem gerenciados:**
```yml
172.27.11.10:
  nodename: automation
  hostname: 172.27.11.10
  osVersion: 4.19.0-9-amd64
  osFamily: unix
  osArch: amd64
  description: Rundeck server node
  osName: Linux
  username: rundeck
  tags: ['jenkins', 'rundeck', 'gitea']
172.27.11.20:
  nodename: balancer
  hostname: 172.27.11.20
  osVersion: 4.19.0-9-amd64
  osFamily: unix
  osArch: amd64
  description: Balancer node
  osName: Linux
  username: rundeck
  tags: ['balancer', 'haproxy']
172.27.11.30:
  nodename: database
  hostname: 172.27.11.30
  osVersion: 4.19.0-9-amd64
  osFamily: unix
  osArch: amd64
  description: Database node
  osName: Linux
  username: rundeck
  tags: ['database', 'mysql']
172.27.11.100:
  nodename: docker1
  hostname: 172.27.11.100
  osVersion: 4.19.0-9-amd64
  osFamily: unix
  osArch: amd64
  description: Docker node
  osName: Linux
  username: rundeck
  tags: ['docker', 'apps']
172.27.11.200:
  nodename: docker2
  hostname: 172.27.11.200
  osVersion: 4.19.0-9-amd64
  osFamily: unix
  osArch: amd64
  description: Docker node
  osName: Linux
  username: rundeck
  tags: ['docker', 'apps']
``` 


Criação de Script para roteiro abaixo:
---
  - Criar usuário **rundeck** em todas as máquinas;
  - Adcionar o usuário **rundeck** aos sudoers *;
  - Copiar a chave do **rundeck** para as outras máquinas;

    - _No caso o usuário **rundeck** precisará executar comandos administrativos sem a necessidade de senha._



```bash
KEY="$(vagrant ssh automation -c 'sudo cat /root/keys/rundeck.pub' < /dev/null)"
read -r -d '' CMD <<EOF
sudo useradd -r -s /bin/bash -m -d /var/lib/rundeck rundeck;
echo "rundeck ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/rundeck;
sudo mkdir -p /var/lib/rundeck/.ssh;
echo '$KEY' | sudo tee /var/lib/rundeck/.ssh/authorized_keys;
sudo chown -R rundeck: /var/lib/rundeck/.ssh
EOF
for M in $(vagrant status | grep 'running' | cut -d' ' -f1); do
vagrant ssh $M -c "$CMD";
done
```

