## Trabalho Serviços de Rede – Primeira Parte - Passo a Passo

Pacotes: SSH, FTP, DHCP3-server, PROFTPD

IP’s Máquinas:
- MV1 🡪 192.168.1.1
- MV2 🡪 192.168.1.2

Criar duas Interfaces de Rede para cada uma. A primeira deixar como "Modo Bridge" e a segunda como "Rede Interna".

### SSH

### Configuração Servidor MV1

- Abrir o arquivo de configuração:  
  ```shell
  gedit /etc/ssh/sshd_config
  ```

- Descomentar o ***X11Forwarding yes***  
  Isso faz com que o servidor MV1 permita que usuários abram aplicativos gráficos remotamente.

- Acrescentar **"no"** na linha **"permitRootLogin"** ficando do seguinte modo: **"permitRootLogin no"**.  
  Com essa linha o servidor MV1 impede que usuários remotos possam logar como root no mesmo.

- Descomentar a opção ***"issue.net"***  
  Usaremos essa linha para habilitar a exibição da mensagem que aparecerá quando o cliente MV2 logar com um determinado usuário no servidor.

  - Abrir o arquivo:
    ```shell
    gedit /etc/issue.net
    ```
  - Escrever a mensagem que desejamos que apareça: "BEM VINDO AO SERVIDOR SSH VIRTUAL"

### Configuração Cliente MV2

No SSH não faremos uma configuração específica, apenas utilizaremos ele para a geração das chaves assimétricas. Para isso, fazer:

- Inserir o comando:  
  ```shell
  ssh-keygen –t rsa
  ```
  - Dar "Enter"
  - Colocar a senha (info.ifes)
  - Repetir a senha (info.ifes)

- Feito isso, geraram-se duas chaves: Uma pública e uma privada:
  - .ssh/id_rsa
  - .ssh/id_rsa.pub

- Utilizar o comando:  
  ```shell
  ssh-copy-id aluno4@192.168.1.1
  ```  
  Com esse comando nós exportamos a chave pública para o servidor.

- Após isso, ir no servidor e reestartar o serviço com:  
  ```shell
  sudo systemctl restart ssh
  ```

Para testar, basta conectar no servidor através do MV2 com o comando:
```shell
ssh 192.168.1.1
```

Para saber se a chave foi mesmo copiada, verificar em cd ***/home/aluno4/.ssh*** (Oculto). Verificar se existe o arquivo ***authorized_keys***.

### DHCP

- Abrir o arquivo de configuração:  
  ```shell
  gedit  /etc/dhcp3/dhcpd.conf
  ```

- Descomentar a linha ***"range ..."*** e editar como abaixo:  
  - ```shell
    range 192.168.1.10 192.168.1.20;
    ```  
    Faixa de endereços que serão distribuídos.

  - ```shell
    subnet 192.168.1.0 netmask 255.255.255.0
    ```
    IP da Rede + Máscara

  - ```shell
    option routers 192.168.1.1;
    ```  
    Gateway da Rede

  - ```shell
    option domain-name "servredes.net"
    ```
    Nome do domínio da Rede

  - ```shell
    default-lease-time 3600;
    ```  
    Para fazer renovar o empréstimo a cada 3600 segundos (1 Hora)

  - ```shell
    option nis-servers 192.168.1.1
    ```
    Para configurar o MV1 como servidor NIS.

  - ```shell
    option nis-domain SERVREDES;
    ```  
    Nome do domínio NIS.

### FTP

- Entrar no arquivo de Configuração:
  ```shell
  gedit /etc/proftpd/proftpd.conf
  ```

- Deixar a sessão "Anonymous~ftp" comentada, para que o MV1 não aceite logins de usuários anônimos.

- Criar as pastas com o nome dos integrantes do grupo:
  ```shell
  mkdir /home/ftp/Lara
  mkdir /home/ftp/Marlon
  ```

- Criar os usuários com o nome dos integrantes do grupo dentro de suas respectivas pastas:  
  ```shell
  adduser –home /home/ftp/Lara –no-create-home lara
  adduser –home /home/ftp/Marlon –no-create-home marlon
  ```

- Para limitar o acesso dos usuarios ao diretorio home, descomentar o parâmetro em ***gedit /etc/proftpd/proftpd.conf***:  
  ```shell
  DefaultRoot               ~
  ```

- Para atribuir as permissões de dono e assim poder fazer upload e download na sua pasta, basta dar o comando:  
  ```shell
  chown –R lara.lara /home/ftp/Lara
  chown –R marlon.marlon /home/ftp/Marlon
  ```

- Restartar o servidor ftp com:
  ```shell
  sudo systemctl restart proftpd
  ```

- Para mudar a mensagem que aparecerá quando o cliente logar no ftp, mude ou acrescente se necessário as linhas abaixo no /etc/proftpd/proftpd.conf:  
  ```shell
  DisplayChdir            .message
  AccessGrantMsg          "Bem-vindos ao servidor FTP virtual"
  DisplayConnect          display.msg
  DisplayLogin            welcome.msg
  ListOptions             "-l"
  ```

- Para estabelecer uma política de quotas, vá no arquivo: ***gedit /etc/fstab***, e acrescente os parâmetros no local abaixo:  
  - Na frente de: "...re-mount-ro" acrescentar **sem espaços**:  
  ```shell
  ,usrquota,grpquota
  ```

- Após isso, reinicie o ftp com:  
  ```shell
  sudo systemctl restart proftpd
  ```

- Assim, para colocar o limite de quota, de o comando para os dois:  
  ```shell
  edquota lara
  edquota marlon
  ```  
  Ao der o comando, vá abaixo do local "hard" e mude o valor para 10000.

- Para inciar o sistema de quota, de o comando:  
  ```shell
  quotaon -a
  ```  