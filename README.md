# Monitoramento Automatizado do nginx no Ubuntu 20.04 LTS

## Sobre o projeto

A prática consiste na instalação do WSL (Subsistema do Windows para Linux) no Windows e na criação de uma instância Ubuntu 20.04 LTS. Inclui também a configuração de um servidor nginx, o monitoramento do status do serviço por meio de um script personalizado e a automatização da execução deste script a cada 5 minutos. O script deve conter a data, hora, o nome do serviço, o status e uma mensagem personalizada de ONLINE ou OFFLINE. O objetivo é garantir a continuidade e a disponibilidade do serviço, registrando seu status em arquivos separados.


### Índice

1. [Pré-requisitos](#1-pré-requisitos)
2. [Ativação e configuração do WSL](#2-ativação-e-configuração-do-wsl)
3. [Instalação e Configuração do Ubuntu 20.04 LTS](#3-instalação-e-configuração-do-ubuntu-2004-lts)
4. [Instalação e Configuração do nginx](#4-instalação-e-configuração-do-nginx)

## 1. Pré-requisitos

- Windows 10 versão 2004 e superior ou Windows 11
- Conhecimento básico do terminal Linux

## 2. Ativação e configuração do WSL

Clique com o botão direito do mouse sobre o PowerShell ou o Prompt de Comando do Windows e selecione **"Executar como administrador"**.

Depois, execute o comando:

```powershell
wsl --install
```

Após executado o comando e terminado a instalação do WSL, reinicie o computador.

## 3. Instalação e Configuração do Ubuntu 20.04 LTS

Abra o PowerShell como administrador e execute o comando:

```powershell
wsl --install -d Ubuntu-20.04
```

Alternativamente, você pode abrir a Microsoft Store, buscar por "Ubuntu 20.04 LTS", clicar em adquirir e instalar a distribuição.

Terminado o processo de instalação do Ubuntu no WSL, você será solicitado a criar um nome de usuário e senha. Esta conta será o **usuário padrão e administrador da distribuição**, com permissões para executar comandos de super usuário (`sudo`).

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

Para verificar se o servidor está funcionando, abra o navegador e digite http://localhost na barra de endereços. Se tudo estiver certo, o servidor vai mostrar a página padrão do Nginx:

![Página Padrão do nginx](imgs/nginx_via_localhost.jpeg)