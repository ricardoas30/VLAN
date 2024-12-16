
![Logo](https://raw.githubusercontent.com/ricardoas30/VLAN/refs/heads/main/OpenVSwitch_Resized.webp)


# Configurando o Open vSwitch no Proxmox - VLAN

Para configurar o Open vSwitch (OVS) no Proxmox, siga as etapas detalhadas abaixo. O Open vSwitch oferece recursos avançados de gerenciamento de rede comparado aos bridges Linux padrão que vêm com o Proxmox.




## Passo 1: Instalação do Open vSwitch:

Atualizando o sistema

```bash
  apt install openvswitch-switch
```

Esta ação irá instalar os serviços necessários para o funcionamento do OVS no seu servidor Proxmox

## Passo 2: Configuração das Interfaces de Rede:

Após a instalação, você precisa editar o arquivo de configuração de rede /etc/network/interfaces. Aqui está um exemplo básico de configuração para um nó Proxmox:

```bash
  # /etc/network/interfaces  
auto lo  
iface lo inet loopback  

auto vmbr0  
iface vmbr0 inet manual  
    ovs_type OVSBridge  
    ovs_ports eno1 vlan10  

auto eno1  
iface eno1 inet manual  
    ovs_type OVSPort  
    ovs_bridge vmbr0  

auto vlan10  
iface vlan10 inet static  
    address 172.16.255.12/24  
    gateway 172.16.255.1  
    ovs_type OVSIntPort  
    ovs_bridge vmbr0  
    ovs_options tag=10
```    

Neste exemplo:

vmbr0 é a bridge OVS.\
eno1 é uma porta física mapeada para a bridge OVS.\
vlan10 é uma interface VLAN que se conecta à bridge OVS e obtém um endereço IP estático

## Passo 3: Reiniciar o Serviço de Rede:

Após editar o arquivo de configuração, reinicie o serviço de rede com o comando:

```bash
  systemctl restart networking.service
```
Em alguns casos, pode ser necessário reiniciar o servidor para aplicar completamente as alterações

## Passo 4: Verificar o Status do Open vSwitch:

Use o seguinte comando para verificar se o OVS está funcionando corretamente:

```bash
  ovs-vsctl show
```

Isso mostrará as bridges e portas que estão atualmente configuradas no Open vSwitch


## Passo 5: Configuração de Port Mirror (opcional):

Se você precisar de monitoramento de tráfego, pode configurar a duplicação de porta utilizando comandos OVS. Por exemplo, para configurar o mirror, você pode usar:

```bash
  ovs-vsctl -- --id=@m create mirror name=SPAN \
-- add bridge vmbr0 mirrors @m
```    

Em seguida, você define a porta de origem e a porta de saída do mirror conforme necessário

## Considerações Finais

A mudança de bridges Linux padrão para OVS pode requerer um plano cuidadoso sobre a programação das redes envolvidas, especialmente se você tiver máquinas virtuais já em execução. Seria prudente migrar ou desligar as VMs antes de aplicar essas alterações.\
O Open vSwitch oferece vantagens, como VLANs e outras funções avançadas que não estão disponíveis com bridges Linux padrão, portanto, é uma escolha preferível para setups mais complexos.
Esses passos fornecem uma base sólida para configurar e utilizar o Open vSwitch no Proxmox. Para necessidades mais complexas, consulte a documentação oficial ou outros guias específicos.
