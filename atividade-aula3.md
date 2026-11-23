# Atividade - Aula 03: Interfaces e Endereçamento de Rede

## 1. Identificação

- **Nome completo:** `Daniel Menezes Nascimento`
- **Matrícula:** `2024327064`
- **Turma:** Técnico em Desenvolvimento de Sistemas 2026.02
- **Data:** 22/11/2026
- **Título:** Inspeção de interfaces e comparação dos modos NAT e Bridge

## 2. Objetivo

Identificar endereços IP, máscaras, MAC, gateway e rotas em modo NAT e Placa em Ponte. Também testar conectividade em camadas e diagnosticar DNS e rotas.

## 3. Ambiente

- Host: Windows
- VM: Ubuntu Server no VirtualBox
- Interface da VM: `enp0s3`
- Modos testados: NAT e Placa em Ponte
- Gateways e IPs conforme a saída apresentada pela VM

## 4. Procedimento

### 4.1 Inspeção em modo NAT

No VirtualBox, o Adaptador 1 foi mantido em modo NAT.

![Configuração NAT](imagens/aula3_01_nat_virtualbox.png)

Na VM foram executados:

```bash
ip -c -4 addr
ip -c route
netplan status
```

![Interfaces e rotas em NAT](imagens/aula3_02_nat_interfaces_rotas.png)

A interface virtual recebeu um endereço fornecido pela rede NAT do VirtualBox. A rota padrão normalmente apontou para o gateway `10.0.2.2`.

### 4.2 Testes em camadas

```bash
ping -c 4 127.0.0.1
ping -c 4 <IP-DA-VM>
ping -c 4 10.0.2.2
ping -c 4 8.8.8.8
ping -c 4 google.com
```

![Testes de ping em NAT](imagens/aula3_03_nat_pings.png)

O ping ao gateway testou a conectividade local e o ping a `8.8.8.8` verificou o acesso à internet sem depender de DNS. O ping a `google.com` também testou a resolução de nomes.

### 4.3 Rastreamento da rota

```bash
sudo apt-get update
sudo apt-get install -y traceroute
traceroute 8.8.8.8
```

![Traceroute em NAT](imagens/aula3_04_nat_traceroute.png)

O primeiro salto mostrou o gateway da rede NAT do VirtualBox.

### 4.4 Comparação com o host Windows

No PowerShell do Windows:

```powershell
ipconfig /all
```

![Informações do host Windows](imagens/aula3_05_host_ipconfig.png)

O IP e o MAC da interface física do Windows são diferentes dos da VM, pois o NAT cria uma rede virtual separada.

### 4.5 Alternância para Bridge

A VM foi desligada e o adaptador 1 do VirtualBox foi configurado como **Placa em Ponte**, associado à interface física do host.

![Configuração Bridge](imagens/aula3_06_bridge_virtualbox.png)

Após o novo boot:

```bash
sudo poweroff
# Após configurar Bridge no VirtualBox, iniciar a VM novamente
ip -c -4 addr
ip -c route
ping -c 4 8.8.8.8
```

![Interfaces em Bridge](imagens/aula3_07_bridge_interfaces.png)

![Rota em Bridge](imagens/aula3_08_bridge_rota.png)

![Ping em Bridge](imagens/aula3_09_bridge_ping.png)

## 5. Comparação NAT e Bridge

| Item | NAT | Bridge |
|---|---|---|
| IP da VM | Rede privada do VirtualBox | Rede da interface física |
| Gateway | `10.0.2.2` | Gateway da rede do laboratório |
| Host acessa diretamente a VM | Não por padrão | Sim |
| VM acessa a internet | Sim | Sim, caso liberada pela rede |

## 6. Problemas e soluções

O modo Bridge pode não receber endereço por DHCP em redes que não oferecem esse serviço. Nesse caso, a interface continua configurada e o IP deve ser definido estaticamente na prática seguinte.

## 7. Conclusão

A prática diferenciou endereçamento, gateway e rotas em NAT e Bridge. Os testes demonstraram que o NAT separa a VM do host, enquanto o Bridge coloca a VM diretamente no segmento físico ou no segmento entregue pela rede do laboratório.

## Explicação dos comandos

| Comando | Função |
|---|---|
| `ip addr` | Mostra interfaces, IP e MAC |
| `ip route` | Mostra rotas e gateway |
| `netplan status` | Mostra a configuração de rede gerenciada pelo Netplan |
| `ping` | Testa conectividade ICMP |
| `traceroute` | Exibe os saltos até um destino |
