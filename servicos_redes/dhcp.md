## Introdução

O DHCP (Dynamic Host Configuration Protocol – Protocolo de configuração Dinâmica de máquinas) é
um protocolo criado para facilitar a configuração de endereços IP em uma rede, pois permite um
endereçamento endereçamento automático automático de IP para máquinas máquinas em uma rede TCP/IP.

Em uma rede onde existe um servidor DHCP ativado, é possível receber uma configuração de IP
automaticamente ao ligar a máquina, sendo que não existe necessidade de nenhuma configuração adicional após tê-la recebido.

### Funcionamento do protocolo

O cliente DHCP envia um pacote broadcast para a rede chamado DHCP DISCOVER. Este pacote tem como destino o endereço IP 255.255.255.255 e ethernet FF:FF:FF:FF:FF:FF e visa descobrir descobrir a existência existência de um servidor DHCP.

O servidor DHCP, ao receber o pacote, consulta sua tabela de configuração e prepara uma resposta para a máquina cliente chamada de DHCP OFFER. Esta resposta já contém uma oferta de configuração para o cliente, cliente, mas nada foi reservado reservado ainda.

O cliente, se concordar com a oferta do servidor, fará um pedido formal de configuração específico para este servidor. Este pedido é feito através de um pacote chamado DHCP REQUEST que basicamente contém as configurações que o servidor diz ter disponíveis.

Se as informações ainda estiverem válidas, o servidor responderá com um pacote DHCP ACK e gravará em sua base de dados local que este endereço endereço IP deixou de estar vago.

Ao receber o pacote DHCP ACK, o cliente ajustará suas opções de rede e serviços e sua configuração estará pronta. Entretanto esta configuração só valerá por tempo determinado (lease time).

<p align="center">
    <img src="https://www.cisco.com/c/dam/en/us/support/docs/switches/catalyst-9300-series-switches/217366-configure-dhcp-in-ios-xe-evpn-vxlan-01.png" width="468" height="354">
</p>

Periodicamente neste intervalo determinado pelo servidor, o cliente DHCP precisa renovar seu empréstimo e não há garantia de que ele vá conseguir a mesma configuração novamente.

## Configuração do servidor

Para a configuração do servidor, é necessária a edição do arquivo /etc/dhcp3/dhcpd.conf. As principais opções do arquivo que devem ser configuradas são:

- **Authoritative** *Yes|No*  
  Se constar no arquivo, e houver outros servidores DHCP na rede, define este como sendo prioritário (autoritário) na rede. Desnecessário se houver apenas um servidor.

- **default-lease-time** *segundos*  
  Define o tempo mínimo (padrão) que um cliente terá o endereço IP reservado (alugado) para si.

- **max-lease-time** *segundos*  
  Define o tempo máximo que um cliente terá o endereço IP reservado (alugado) para si. O próprio daemon do cliente cliente dhcp é responsável responsável por renovar renovar o endereço endereço quando finalizado este prazo.

- **option routers** *IP_gateway*  
  Define o endereço IP do roteador (gateway padrão) de acesso à internet.

- **option domain-name-servers** *IP_DNS-1,IP_DNS-2*  
  Define o(s) endereço(s) IP(s) do(s) servidor(es) DNS. Se houver mais de um, devem ser separados por vírgulas.

- **option domain-name** "*nome_domínio*"  
  Define o nome de domínio que será passado ao cliente.

- ```shell
  subnet IP_rede netmask Subnet_Mask
  {
    range IP_inicial IP_final;
  }
  ```  
  Define a sub-rede atendida por este servidor (parâmetro subnet) juntamente com sua máscara de sub-rede e a faixa de IPs disponível para aluguel. Mais de uma opção range pode ser configurada ao mesmo tempo.

Sempre que configurar um servidor com duas placas de rede, é importante que o servidor DHCP seja configurado para escutar apenas na placa da rede local. No Debian, esta configuração vai no arquivo arquivo "/etc/default/dhcp "/etc/default/dhcp3-server" server".

Procure pela linha **INTERFACES=""** e adicione a placa que o servidor DHCP deve escutar, como em:

```shell
INTERFACES="eth0"
```

Para que a configuração entre em vigor, basta reiniciar o serviço novamente.

### Atribuindo IPs fixos às máquinas

É ainda possível configurar o DHCP com IP fixo. Na seção abaixo é possível configurar um endereço fixo para uma determinada máquina da sub-rede:

Caso não seja utilizada a opção de configurar uma range, esta opção de host possibilita que se
relacione endereços MAC a endereços IP, ou seja, apenas máquinas máquinas com os MAC listados listados receberão receberão endereços, não possibilitando que qualquer máquina receba um endereço IP do servidor. Essa é uma medida de segurança interessante para proteger a rede.

```shell
host nome_da_maquina {
  Hardware ethernet MAC_adress
  Fixed-address endereço_IP_fixo
}
```

### Opções avançadas

É possível em uma configuração de DHCP, explicitar algumas outras opções, fora as já comentadas aqui, referentes à existência de servidores WINS e NIS na rede.

- **option netbios-name-servers**  
  Permite configurar o IP do servidor Netbios

- **option nis-servers**  
  Permite configurar o IP do servidor NIS da rede.

- **option nis-domain**  
  Permite configurar o nome do domínio NIS das máquinas

## Cliente DHCP

Em uma rede onde as máquinas utilizam DHCP, os clientes DHCP são automaticamente configurados para buscar um endereço IP quando a máquina é inicializada. Entretanto Entretanto é possível possível utilizar utilizar um programa programa cliente cliente para forçar um novo empréstimo de endereços,
através do comando dhclient. Para fazer uma requisição ao servidor DHCP, é necessário utilizar o comando:

```shell
dhclient eth0
```

Em máquinas Windows, é possível forçar um novo empréstimo através do comando:

```shell
ipconfig /renew
```

## Referências

- Baseado material do professor [João Paulo de Brito Gonçalves](http://lattes.cnpq.br/5648961589597642)
- <https://www.vivaolinux.com.br/dica/Configuracao-de-servidor-DHCP-no-Ubuntu-Server-1704>