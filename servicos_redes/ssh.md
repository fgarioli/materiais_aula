## O que é SSH?

SSH (Secure Shell) é um padrão para comunicação e acesso remoto a máquinas Linux de forma segura, ou seja, utilizando criptografia. Ele permite administrar máquinas remotamente, executando inclusive aplicativos gráficos e permite transferir arquivos de várias formas diferentes.

A grande vantagem do SSH sobre outras ferramentas de acesso remoto é a grande ênfase na segurança. Um servidor SSH bem configurado é virtualmente impenetrável e você pode acessá-lo de forma segura, mesmo que a sua rede local esteja comprometida. Ele utiliza um conjunto de técnicas de criptografia para assegurar que apenas as pessoas autorizadas terão acesso ao servidor, que todos os dados transmitidos sejam impossíveis de decifrar e que a integridade da conexão seja mantida.

O SSH é dividido em dois módulos. O sshd é o módulo servidor, um serviço que fica residente na máquina que será acessada, enquanto o ssh é o módulo cliente, um utilitário que você utiliza para acessá-lo.

![Demonstração básica do SSH](https://www.hostinger.com.br/tutoriais/wp-content/uploads/sites/12/2017/08/criptografia-simetrica-ssh-hostinger.jpg)

## Configuração do servidor

A configuração do servidor, independentemente da distribuição usada, vai no arquivo "/etc/ssh/sshd_config". Uma das primeiras linhas do arquivo de configuração é referente a porta que será usada pelo servidor SSH para aceitar requisições do cliente:

```shell
Port 22
```

A opção "PermitRootLogin" pode ser configurada para impedir que usuários se loguem como root no servidor, bem como bloquear o acesso root com senha, sendo somente via chave ssh (utilize apenas uma opção por vez):

```shell
PermitRootLogin yes|no|prohibit-password
```

Para impedir logins de usuários específicos, colocar no arquivo sshd_config a variável:

```shell
DenyUsers usuário
```

Para permitir logins somente de usuários específicos, colocar no arquivo sshd_config a variável:

```shell
AllowUsers usuário
```

Banner: Alguns servidores exibem mensagens de advertência antes do prompt de login, avisando que todas as tentativas de acesso estão sendo monitoradas ou coisas do gênero. A mensagem é especificada através da opção "Banner", onde você indica um arquivo de texto com o conteúdo a ser mostrado, ou editar o arquivo padrão:

```shell
Banner /etc/issue.net
```

## Criptografia do SSH

Quando você se conecta a um servidor SSH, seu micro e o servidor trocam suas chaves públicas, permitindo que um envie informações para o outro de forma segura. Através deste canal inicial é feita a autenticação, seja utilizando login e senha, seja utilizando chave e passphrase.

![Criptografia](https://www.linuxnaweb.com/images/post/2020/chaves.png)

Até aqui, tudo é feito utilizando chaves de 512 bits ou mais (de acordo com a configuração). O problema é que, embora impossível de quebrar, este nível de encriptação demanda uma quantidade muito grande de processamento. Se todas as informações fossem transmitidas desta forma, o SSH seria muito lento.

Para solucionar este problema, depois de fazer a autenticação, o SSH passa a utilizar um algoritmo mais simples, que demanda muito menos processamento, para transmitir os dados. Por padrão é utilizado o 3DES (triple-DES), que é um algoritmo de criptografia simétrica.

Para criação de chaves criptográficas para uso no ssh, é necessário utilizar o comando sempre como seu usuário padrão e não como root.

```shell
ssh-keygen -o -a 100 -t ed25519 -C "usuario@linuxnaweb.com"
```

```shell
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase): <PALAVRA CHAVE>
Enter same passphrase again: <REPITA A PALAVRA CHAVE>
Your identification has been saved in /home/usuario/.ssh/id_ed25519
Your public key has been saved in /home/usuario/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:pWha/8WTmUMKKk2JNcA941cfGL8tywgZXxAxP6WJRFY usuario@linuxnaweb.com
The key's randomart image is:
+--[ED25519 256]--+
|   ...   .X*E .  |
|    ..+  oo*.+   |
|     .oo..o.B.   |
|     o.+.* ..+   |
|    . *.S . + .  |
|     * o o * *   |
|    o o . o @    |
|     .   . . o   |
|          .      |
+----[SHA256]-----+
```

Isso vai gerar os arquivos ".ssh/id_ed25519" e ".ssh/id_ed25519.pub" dentro do seu diretório home, que são respectivamente sua chave privada e a chave pública. O ".ssh/id_ed25519" é um arquivo secreto, que deve usar obrigatoriamente o modo de acesso "600" (que você define usando o chmod), para evitar que outros usuários da máquina possam lê-lo. Muito servidores recusam a conexão caso os arquivos estejam com as permissões abertas.

## Exportando a chave pública

Para exportar a chave pública, de forma que seja possível uma comunicação criptografada com o servidor, é necessário seguinte comando para que a chave pública seja enviada para o diretório remoto do usuário no servidor:

```shell
ssh-copy-id login@servidor
```

A partir daí, ao invés de pedir sua senha, o servidor verifica a chave privada, instalada na sua máquina e em seguida pede a passphrase. Mesmo que alguém consiga roubar sua chave privada, não conseguirá conectar sem saber a passphrase e vice-versa.

## Conectando em um servidor SSH

Para se logar em um servidor SSH, utilize o comando:
```shell
ssh ip_servidor
```
Para se logar no servidor com a possibilidade de abrir aplicativos gráficos entre com o comando:
```shell
ssh -X ip_servidor
```
Antes é necessário descomentar a opção ForwardX11 Yes no arquivo ssh_config e acrescentar X11Forwarding yes no arquivo sshd_config no servidor.

Para se logar como um usuário específico ao invés do usuário padrão use o comando:
```shell
ssh usuário@servidor
ssh -l usuário servidor
```

## Transferência de arquivos no SSH

O SSH possui uma primitiva de transferir arquivos chamada scp (secure copy). Tal ferramenta permite que arquivos sejam enviados entre máquinas utilizando os mesmos recursos de autenticação e criptografia que o ssh. Usando o comando scp é possível especificar em uma única linha o login e o endereço do servidor, junto com o arquivo que será transferido. A sintaxe é a seguinte:

```shell
scp arquivo_local login@servidor:pasta_remota
```

Exemplo:

```shell
scp /home/arquivo.tar usuario@empresa.com.br:/var/www/download
```

## SSH no Windows

Existem diversas versões do SSH. A maioria das distribuições Linux inclui o OpenSSH, que não possui um cliente for Windows. No entanto, isso não chega a ser um problema, pois o SSH é um protocolo aberto, o que permite o desenvolvimento de clientes para várias plataformas, inclusive Windows.

Um exemplo de cliente simples e gratuito é o Putty. Usar o putty para se conectar a servidores Linux é bem simples, pois não precisa sequer ser instalado. A grande limitação do uso de clientes SSH no Windows é que são limitados ao modo texto, sendo impossível executar aplicativos gráficos no mesmo, devido às diferenças entre as interfaces gráficas do Windows e Linux.

Ambos podem ser baixados no: https://www.putty.org/. Outro exemplo é a versão da SSH Security, que tem vários recursos mas é gratuita apenas para universidades e usuários domésticos. O link é: http://www.ssh.com.

## Referências

- Baseado material do professor [João Paulo de Brito Gonçalves](http://lattes.cnpq.br/5648961589597642)
- <https://slideplayer.com.br/amp/334380/>
- <https://linuxize.com/post/how-to-enable-ssh-on-ubuntu-20-04/>
- <https://www.linuxnaweb.com/criando-chaves-ssh-no-linux>
- <https://askubuntu.com/questions/108217/ssh-host-key-verification-failed>
- <https://www.youtube.com/watch?v=a2BVQ42CQyA>