## Introdução

O servidor de FTP mais usado no Linux é o ProFTPD, incluído em quase todas as distribuições. O funcionamento do FTP é bem mais simples que o do Samba ou SSH, por isso ele é usado como uma forma simples de disponibilizar arquivos na internet ou mesmo dentro da rede local, sem muita segurança.

A principal limitação do protocolo FTP é que todas as informações são transmitidas pela rede de forma não encriptada, como texto puro, incluindo os logins e senhas. Ou seja, alguém capaz de sniffar a conexão, usando um programa como o Wireshark, veria tudo que está sendo transmitido.

No Debian, durante a instalação do pacote do ProFTPD, geralmente serão feitas algumas perguntas. A primeira é se você deseja deixar o servidor FTP ativo em modo standalone ou em modo inetd.

O standalone é mais seguro e mais rápido, enquanto o inetd faz com que ele fique ativo apenas quando acessado, economizando cerca de 400 KB de memória RAM (que fazem pouca diferença hoje em dia). O modo standalone é a opção recomendada.

Você terá também a opção de ativar o acesso anônimo, que permite acessos anônimos (somente leitura) na pasta "/home/ftp", onde você pode disponibilizar alguns arquivos para acesso público.

Nesse caso, os usuários se logam no seu servidor usando a conta "anonymous" e um endereço de e-mail como senha. Caso prefira desativar o acesso anônimo, apenas usuários com login válido na máquina poderão acessar o FTP.

Depois de concluída a instalação, o servidor fica ativo por padrão, é inicializado automaticamente
durante o boot e pode ser controlado manualmente através do serviço "proftpd", como em "/etc/init.d/proftpd start".

## Configuração do servidor

Instalação do serviço:

```shell
sudo apt-get install proftpd -y
```

Depois da instalação, inicie o serviço do ProFTPD:

```shell
sudo systemctl start proftpd
```

Para que o ProFTPD seja iniciado automaticamente em uma eventual reinicialização, faça a ativação do serviço:

```shell
sudo systemctl enable proftpd
```

Para conferir o status do serviço:

```shell
sudo systemctl status proftpd
```

O arquivo de configuração do ProFTPD é o */etc/proftpd/proftpd.conf*

### Opçoes de Configuração

- **Port 21**  
  Configuração da porta que o serviço irá responder (o padrão é a porta 21)

- **MaxInstances**  
  Configuração do número de conexões simultâneas no servidor

- **DefaultRoot ~**  
  Limitar os usuários o acesso dos usuários ao seu diretório home

- A princípio, apenas os usuários que tiverem logins válidos no servidor poderão acessar o FTP. Caso se queira abrir um FTP público deve-se liberar a seção ***<Anonymous ~ftp>***  
  Dentro desta seção a opção ***MaxClients*** possibilita configurar o número de usuários anônimos do servidor

- A mensagem DisplayLogin welcome.msg indica a mensagem de boas-vindas que é mostrada quando os usuários fazem login no FTP. Por padrão é exibido o conteúdo do arquivo ***/home/ftp/welcome.msg***.

- A mensagem ***DisplayLogin welcome.msg*** indica a mensagem de boas-vindas que é mostrada quando os usuários fazem login no FTP. Por padrão é exibido o conteúdo do arquivo ***/home/ftp/welcome.msg***.

### Criação de usuários

- Para criar usuários para utilizar o servidor, é interessante criar os usuários, os diretórios e
bloquear o uso do Shell, para que eles não possam acessar o servidor remotamente através de outros meios.

- Para prender o usuário nos seus diretórios home, deve-se, no arquivo ***/etc/proftpd.conf*** adicionar ao final a linha:  
  ```shell
  DefaultRoot ~
  ```

- Depois, deve-se criar as pastas que serão home dos novos usuários do sistema:  
  ```shell
  mkdir /home/ftp/paulo
  mkdir /home/ftp/Maria
  ```

- O comando para criar os usuário paulo usando a pasta ***/home/ftp/paulo*** como home e o ***/bin/false*** como Shell, seria:  
  ```shell
  adduser --home /home/ftp/paulo –Shell /bin/false --no-create-home Paulo
  ```

- para o usuário Maria seria:  
  ```shell
  adduser --home /home/ftp/maria –shell /bin/false --no-create-home maria
  ```

- Em distribuições derivadas do Debian, é necessário adicionar a linha ***/bin/false*** no final do arquivo ***/etc/shells***

- Para configurar o dono da pasta criada para que seja possível fazer upload dos arquivos, usar o comando:  
  ```shell
  chown –R usuário.usuario /home/ftp/usuario
  ```

### Usando o SFTP

Possibilita a transferência de arquivos utilizando a criptografia do SSH. Para se conectar à um servidor usando o SFTP deve-se digitar:

```shell
sftp usuário@maquina
```

A partir daí, se tem um prompt do sftp. Usando o comando put, pode-se dar upload em arquivos e get para baixar os arquivos do servidor. Usando o cd, pode-se mudar de diretório, ls para listar os arquivos e pwd para se saber em qual diretório se está.

Existem ainda os comandos lcd ( local cd), lls(local ls), lmkdir( local mkdir) e lpwd(local pwd) que permitem mudar o diretório local.

A utilidade destes comandos locais, é que é possível mudando o diretório local, mudar o diretório de onde se quer fazer download na máquina remota.

Atualmente é mais recomendado o uso do SFTP que o FTP, devido às características de segurança do primeiro.

## Quotas

O sistema de quotas permite a instalação em disco ocupado por usuários e/ou grupos. Existem duas versões do sistema de quotas, sendo que a versão 1 já quase não é mais utilizada, por ser a versão 2 mais robusta. É importante notar também que nem todos os sistemas de arquivo suportam quotas. Para utilizar quotas de usuários e grupos, é importante compreender três conceitos básicos:

- **Soft-limit**  
  Limite “soft” da quota de espaço em disco. Ao ultrapassar este limite, o usuário passa a receber notificações por e-mail.

- **Hard-limit**  
  Limite “hard” da quota de espaço em disco. Ao atingir este limite, o usuário não consegue mais escrever no sistema de arquivos.

- **Grace Period**  
  Tempo no qual o usuário pode permanecer além do soft-limit. Após isso, o usuário não consegue escrever em disco até ajustar sua ocupação de disco para ficar abaixo do soft-limit.

A configuração de qualquer sistema que suporte quotas envolve os seguintes passos:

- Adicionar os parâmetros usrquota/grpquota às entradas correspondentes aos sistemas de arquivo do disco no ***/etc/fstab***.  
  - Exemplo: **/dev/hda2 /home ext3 defaults 0 2**
  - Depois da alteração, a linha ficaria:  
    **/dev/hda2 /home ext3 defaults,usrquota,grpquota 0 2**

- Configurar as quotas de usuários e grupos através dos comandos:  
  edquota <nome_usuario>\
  edquota –g <nome_grupo>

A utilidade do uso de quotas é possibilitar um maior controle no uso do disco do servidor pelos usuários, evitando que os mesmos os sobrecarreguem com arquivos de pouca importância.

## Clientes FTP

- Linux  
  - gFTP  
    - Distribuído sob os termos da licença GNU
    - Escrito em C
    - Suporta servidores proxy FTP e HTTP
    - Fácil utilização

- Windows  
  - WS_FTP  
    - Fácil utilização
    - Não precisa ser instalado
    - Interface simples

## Referências

- <https://vitux.com/ubuntu-proftpd-tls/>
- <https://tecdicas.com/como-criar-um-servidor-ftp-no-ubuntu-ou-debian/>
- <https://www.howtoforge.com/how-to-install-proftpd-with-tls-on-ubuntu-1804/>