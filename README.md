# Monitoramento Automatizado do nginx no Ubuntu 20.04 LTS

## Sobre o projeto

Este projeto consiste na criação de uma instância Amazon EC2 com Ubuntu Server 24.04 LTS para configurar um servidor nginx, monitorar o status do serviço por meio de um script personalizado e automatizar sua execução a cada 5 minutos. O script deve registrar a data, hora, nome do serviço, status e uma mensagem personalizada de ONLINE ou OFFLINE, permitindo monitorar a continuidade e a disponibilidade do serviço.

### Índice

1. [Pré-requisitos](#1-pré-requisitos)
2. [Configuração do Ambiente Virtual (VPC)](#2-configuração-do-ambiente-virtual-vpc)
    - 2.1 [Configuração dos Recursos](#21-configuração-dos-recursos)
    - 2.2 [Criação da VPC](#22-criação-da-vpc)
3. [Criação e Configuração da Instância EC2](#3-criação-e-configuração-da-instância-ec2)
    - 3.1 [Configuração do Security Group](#31-configuração-do-security-group)
    - 3.2 [Criação da Instância](#32-criação-da-instância)
4. [Instalação e Configuração do nginx](#4-instalação-e-configuração-do-nginx)
5. [Criação do script de Monitoramento](#5-criação-do-script-de-monitoramento-do-status-do-nginx)
   - 5.1 [Configuração do Diretório de Logs](#51-configuração-do-diretório-de-logs)
   - 5.2 [Criação do Script](#52-criação-do-script)
6. [Automatização do Script](#6-automatização-do-script)
   - 6.1 [Validando a Automatização do Script](#61-validando-a-automatização-do-script)
7. [Referências](#7-referências)

## 1. Pré-requisitos

- Uma conta ativa na AWS
- Conhecimento básico do terminal Linux
- Conhecimento básico do console AWS

## 2. Configuração do Ambiente Virtual (VPC)

Antes de criarmos nossa instância EC2, precisamos configurar o ambiente de rede onde ela será executada. Criaremos uma VPC (Virtual Private Cloud) dedicada para nosso servidor Nginx.

> [!NOTE]
> A AWS oferece duas opções para criação de VPC: manual e automática. Na criação manual,  você configura a VPC, subnets, roteadores, gateways e outras opções de rede de forma  personalizada. Já na opção automática, o **VPC wizard** cria a VPC com subnets públicas e  privadas, já anexa um Internet Gateway, configura as tabelas de rotas e inclui um NAT  Gateway, caso seja selecionado. Usaremos a criação automatizada com o VPC wizard. 

### 2.1 Configuração dos Recursos
 
1. No console AWS, acesse o serviço VPC e clique em "**Criar VPC**.

2. Em "**Geração automática de etiqueta de nome**", deixe marcado para gerar os nomes automaticamente.

3. Digite "nginx-monitoring" como prefixo para o nome dos recursos que serão criados.

4. Configure os recursos: 

    - CIDR da VPC: 10.0.0.0/24 (fornece 256 endereços IP, suficiente para o projeto)
    - Número de zonas de disponibilidade (AZs): 1 (precisamos apenas de uma AZ)
    - Número de subnets públicas: 1 (apenas uma subnet pública para nosso servidor)
    - Número de subnets privadas: 0 (não precisaremos de subnets privadas)
    - NAT gateways: None 
    - VPC endpoints: None 

5. Adicione uma tag de projeto:

```
    Key: Project
    Value: nginx-server
```

## 2.2 Criação da VPC 

1. Clique em "**Criar VPC**" e aguarde a criação dos recursos

2. O wizard criará automaticamente:

    - Uma VPC com DNS hostnames habilitado
    - Uma subnet pública na AZ selecionada
    - Um Internet Gateway anexado à VPC
    - Uma Route Table configurada com rota para o Internet Gateway
    - Um Security Group padrão

#### Preview do VPC Workflow

![VPC Workflow](imgs/nginx-vpc-workflow.png)

## 3. Criação e Configuração da Instância EC2

### 3.1 Configuração do Security Group 

### 3.2 Criação da Instância

## 4. Instalação e Configuração do nginx

Abra o terminal do Ubuntu e execute o seguinte comando para garantir a instalação do pacote correto e sua versão mais recente:

```bash
sudo apt update
```

Após isso, instale o nginx:

```bash
sudo apt install nginx
```

Inicie e verifique o status do nginx:

```bash
sudo systemctl start nginx
```

```bash
sudo systemctl status nginx
```

Caso o nginx esteja rodando corretamente, o comando retornará uma saída como esta:

![Status do nginx Ativo](imgs/nginx_status_ativo.jpeg)

Para verificar se o servidor está funcionando, abra o navegador e digite "https://localhost" na barra de endereços. 'Localhost' aponta para o IP do seu próprio computador, permitindo que você acesse o servidor localmente. Se tudo estiver certo, o servidor deve mostrar a página padrão do nginx:

![Página Padrão do nginx](imgs/nginx_via_localhost.jpeg)

## 5. Criação do script de monitoramento do status do nginx

### 5.1 Configuração do Diretório de Logs

Antes de criar o script, criaremos o diretório onde serão armazenados os logs de monitoramento do nginx:

```bash
sudo mkdir /var/log/nginx_status
```

Altere a propriedade do diretório para seu usuário:

```bash
sudo chown seu_usuario:seu_usuario /var/log/nginx_status
```

Em seguida, ajuste as permissões:

```bash
sudo chmod 755 /var/log/nginx_status
```

Essa configuração garante que você tenha acesso total ao diretório, enquanto outros usuários podem apenas ler e executar os arquivos dentro dele.

### 5.2 Criação do Script

Armazenaremos o script dentro do diretório `/usr/local/bin`. Esse diretório já está incluído no PATH por padrão, permitindo a execução do script de qualquer lugar, sem precisar especificar o caminho completo.

Para criar e editar o script, utilize um editor de texto. Utilizando o `nano`:

```bash
sudo nano /usr/local/bin/nginx_status_monitor.sh
```

Digite o script:

```bash
#!/bin/bash

# obtém a data e hora atuais
data_hora=$(date "+%d-%m-%Y %H:%M:%S")

# nome do serviço
servico="nginx"

# status do serviço
status=$(systemctl is-active $servico)

# caminhos para os arquivos de log
log_online="/var/log/nginx_status/nginx_online.log"
log_offline="/var/log/nginx_status/nginx_offline.log"

# verifica o status do serviço nginx e escreve nos arquivos de log
if [ "$status" = "active" ]; then
    echo "Data e Hora: $data_hora | Serviço: $servico | Status: $status | O serviço $servico está ONLINE." >> "$log_online"
else
    echo "Data e Hora: $data_hora | Serviço: $servico | Status: $status | O serviço $servico está OFFLINE" >> "$log_offline"
fi
```

Pressione `CTRL + O` e `ENTER` para salvar e `CTRL + X` para sair. Após isso, garanta permissão de execução ao script para que você possa rodá-lo:

```bash
sudo chmod +x /usr/local/bin/nginx_status_monitor.sh
```

Para verificar se está funcionando corretamente, execute o script manualmente:

```bash
nginx_status_monitor.sh
```

Em seguida, verifique os arquivos de log correspondentes. Exemplo do arquivo de log `nginx_online.log` com os logs do serviço ativos:

![Entrada de Log Online Manual](imgs/manual_online_log_entry.jpeg)

## 6. Automatização do Script

Para automatizar a execução do script, usaremos o serviço **cron**. O cron é um serviço do sistema operacional responsável por agendar e executar tarefas automaticamente em intervalos definidos. Ele usa o arquivo **crontab** para armazenar as configurações das tarefas, que podem ser executadas em horários específicos. Para automatizar nosso script, execute o seguinte comando para abrir e editar o crontab:

```bash
crontab -e
```

Selecione um editor de texto. Adicione a seguinte linha ao arquivo:

```bash
*/5 * * * * /usr/local/bin/nginx_status_monitor.sh
```

Salve e saia do arquivo.

Os arquivos de configuração do cron possuem cinco campos para especificar tempo e data, seguidos pelo comando que será executado. Cada um desses cinco campos é separado por um espaço e não podem haver espaços dentro de cada campo. Eles são os seguintes:

- Minuto (0-59)
- Hora (0-23, onde 0 = meia-noite)
- Dia do mês (1-31)
- Mês (1-12)
- Dia da semana (0-6, onde 0 = domingo)

Um asterisco (\*) pode ser usado para indicar que todas as ocorrências (todas as horas, todos os dias da semana, todos os meses, etc.) de um período de tempo devem ser consideradas.

Sendo assim, no nosso caso, a expressão \*/5 \* \* \* \* faz com que o script seja executado a cada 5 minutos, independentemente da hora, dia do mês, mês ou dia da semana.

### 6.1 Validando a automatização do script

Após salvar as configurações no crontab, a execução do script será iniciada automaticamente a cada cinco minutos. Podemos checar a lista de tarefas agendadas no cron com o seguinte comando:

```bash
crontab -l
```

O comando deve retornar a tarefa configurada anteriormente como na imagem seguinte:

![Crontab -l Output](imgs/crontab-l.jpeg)

Exemplo das saídas no arquivo de log com o serviço online:

![Logs do Cron Online](imgs/online_log_cron_entries.jpeg)

Exemplo das saídas no arquivo de log com o serviço offline:

![Logs do Cron Offline](imgs/offline_log_cron_entries.jpeg)

## 7. Referências

- [Documentação do WSL](https://docs.microsoft.com/en-us/windows/wsl/)
- [Documentação do nginx](https://nginx.org/en/docs/)
- [Guia de Configuração e Uso de Cron Jobs](https://www.pantz.org/software/cron/croninfo)
