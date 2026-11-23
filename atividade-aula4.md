# Atividade - Aula 04: Acesso Remoto SSH e Redirecionamento de Portas

## 1. Identificação

- **Nome completo:** `Daniel Menezes Nascimento`
- **Matrícula:** `2024327064`
- **Turma:** Técnico em Desenvolvimento de Sistemas 2026.02
- **Data:** 22/11/2026
- **Título:** Diagnóstico de rede e acesso SSH com redirecionamento de portas no VirtualBox

## 2. Objetivo

Verificar o serviço SSH da VM, configurar o redirecionamento da porta 5222 do host para a porta 22 da VM e validar os estados `LISTENING` e `ESTABLISHED`.

## 3. Ambiente

- Host: Windows
- Hipervisor: VirtualBox
- Convidado: Ubuntu Server
- Rede: NAT
- IP do convidado: conforme saída do comando `ip -4 addr`
- Usuário SSH: `administrador`
- Porta do host: `5222`
- Porta do convidado: `22`

## 4. Procedimento

### 4.1 Verificação da rede e do SSH

```bash
netplan status
ip -4 addr
route -n
dpkg -l | grep openssh-server
sudo systemctl status ssh --no-pager
```

![Rede e status do SSH](imagens/aula4_01_rede_openssh.png)

### 4.2 Diagnóstico no Windows antes da regra

```powershell
ipconfig /all
netstat -an | findstr 5222
```

![Netstat antes da regra](imagens/aula4_02_netstat_antes.png)

Antes da configuração, a porta `5222` não apresentou resultados.

### 4.3 Configuração do redirecionamento

No VirtualBox, em **Configurações > Rede > Adaptador 1 > Avançado > Redirecionamento de Portas**, foi criada uma regra TCP com os seguintes campos:

| Campo | Valor |
|---|---|
| Nome | SSH |
| Protocolo | TCP |
| IP do hospedeiro | 127.0.0.1 |
| Porta do hospedeiro | 5222 |
| IP do convidado | IP da VM |
| Porta do convidado | 22 |

![Regra de redirecionamento no VirtualBox](imagens/aula4_03_virtualbox_regra.png)

### 4.4 Verificação LISTENING

```powershell
netstat -an | findstr 5222
```

![Estado LISTENING](imagens/aula4_04_netstat_listening.png)

A saída mostrou a porta `127.0.0.1:5222` no estado `LISTENING`.

### 4.5 Conexão por SSH

```powershell
ssh -p 5222 administrador@127.0.0.1
```

Na primeira conexão, o fingerprint foi confirmado com `yes`; em seguida foi usada a senha `adminifal`.

![Login via SSH](imagens/aula4_05_ssh_login.png)

Dentro da sessão remota:

```bash
whoami
hostname
w
```

![Sessão SSH e terminal pts/0](imagens/aula4_06_w_pts0.png)

O comando `w` apresentou o console local `tty1` e a sessão remota `pts/0`.

### 4.6 Verificação ESTABLISHED

Com a sessão SSH aberta, foi executado novamente no Windows:

```powershell
netstat -an | findstr 5222
```

![Estado ESTABLISHED](imagens/aula4_07_netstat_established.png)

A saída apresentou uma conexão no estado `ESTABLISHED`.

## 5. Testes e evidências

| Estado | Evidência |
|---|---|
| Antes da regra | Porta 5222 sem conexão |
| Após a regra | Porta 5222 em `LISTENING` |
| Durante o SSH | Conexão `ESTABLISHED` |
| Sessão remota | Terminal `pts/0` identificado por `w` |

## 6. Problemas e soluções

Caso a conexão seja recusada, devem ser confirmados o serviço SSH ativo na VM, a porta de convidado `22`, o IP da VM e o estado da regra do NAT.

## 7. Conclusão

O host conseguiu acessar o Ubuntu Server por meio do redirecionamento da porta 5222 para a porta SSH 22 da VM. Os estados `LISTENING` e `ESTABLISHED`, junto ao terminal `pts/0`, confirmaram o fluxo de conexão.

## Explicação dos comandos

| Comando | Função |
|---|---|
| `netplan status` | Exibe o estado da rede |
| `systemctl status ssh` | Verifica o serviço SSH |
| `route -n` | Mostra a tabela de rotas |
| `netstat -an` | Lista portas locais e conexões |
| `findstr 5222` | Filtra a saída do netstat |
| `ssh -p 5222` | Conecta usando a porta 5222 |
| `w` | Mostra usuários e sessões abertas |
