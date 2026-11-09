# Projeto de Prática em Cibersegurança: Auditoria e Força Bruta em Laboratório

Este repositório contém a documentação prática das atividades desenvolvidas no âmbito de treinamento de Cibersegurança. O objetivo principal é demonstrar o ciclo de vida de auditoria de segurança em um ambiente de teste isolado e controlado: desde a validação de topologia de rede e reconhecimento até a execução de testes de força bruta e password spraying contra os serviços FTP, HTTP/Web (DVWA) e SMB.

## 📌 1. Visão Geral do Ambiente e Topologia

O laboratório foi montado em um ambiente de virtualização isolado usando o **Oracle VM VirtualBox**, garantindo a segurança dos testes e a separação da rede host/pública.

* **Máquina Atacante:** Kali Linux (IP: `192.168.100.20/24`)

* **Máquina Alvo:** Metasploitable 2 (IP: `192.168.100.10/24`)

* **Modo de Rede:** Rede Interna (`CLIENTE-LAN`)

* **Endereçamento:** Atribuição estática de IPs no mesmo segmento de rede sem servidor DHCP ativo.

> **Print Recomendado 1:** `images/01_config_rede_virtualbox.png, images/02_config_rede_metasploitable2.png`
> *Captura de tela das Configurações de Rede da VM no VirtualBox mostrando a opção "Rede Interna" e o nome `CLIENTE-LAN`.*

## 🛠️ 2. Ferramentas Utilizadas

| Ferramenta | Finalidade no Laboratório | 
| ----- | ----- | 
| **`ping`** | Teste básico de alcance de rede e conectividade ICMP entre as máquinas. | 
| **`nmap`** | Mapeamento de portas abertas e detecção de versão de serviços (`-sV`). | 
| **`medusa`** | Teste de autenticação por força bruta em redes paralelas contra o serviço FTP. | 
| **Burp Suite** | Proxy de interceptação HTTP e módulo **Intruder** para análise de formulários web. | 
| **`enum4linux` / `rpcclient`** | Enumeração de usuários, compartilhamentos e SID via sessões nulas em SMB/RPC. | 
| **`hydra`** | Execução de ataques de Password Spraying e testes de autenticação paralela contra SMB. | 
| **Wordlists** | Dicionários de senhas (`rockyou.txt` e dicionários customizados) para testes de resiliência de credenciais. | 

## 🚀 3. Cenário A: Ataque de Força Bruta em Serviço FTP

### Etapa 1: Validação de Conectividade de Rede

Após a configuração manual dos IPs nas interfaces virtuais (`eth0`) na sub-rede `192.168.100.0/24`, validou-se a comunicação de rede enviando pacotes ICMP da máquina atacante para o alvo.

```bash
ping -c 4 192.168.100.10
```

* **Flags explicadas:**

  * `-c 4`: Define o limite de envio de 4 pacotes ICMP antes de finalizar a execução.

> **Print Recomendado 2:** `images/03_ping_conectividade.png`
> *Captura do terminal do Kali Linux exibindo a resposta dos 4 pacotes (0% packet loss).*

### Etapa 2: Mapeamento e Reconhecimento com Nmap

Com a conectividade confirmada, executou-se um escaneamento no alvo para verificar a porta 21 (FTP) e identificar a versão exata do software em execução.

```bash
nmap -sV -p 21 -n 192.168.100.10
```

* **Flags explicadas:**

  * `-sV`: Solicita a sondagem do serviço para determinar a versão exata do software (`vsftpd 2.3.4`).

  * `-p 21`: Restringe o escaneamento à porta padrão do FTP.

  * `-n`: Desativa a resolução DNS.

**Resultado da varredura:**

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
```

> **Print Recomendado 3:** `images/04_nmap_reconhecimento_ftp.png`
> *Captura do terminal mostrando o relatório do Nmap com a porta 21/tcp aberta e a versão vsftpd 2.3.4.*

### Etapa 3: Ataque de Força Bruta em FTP com Medusa

Com a confirmação do serviço FTP ativo, utilizou-se a ferramenta **Medusa** para simular uma tentativa de autenticação por força bruta baseada em dicionário contra o usuário padrão `msfadmin`.

```bash
medusa -h 192.168.100.10 -u msfadmin -P /tmp/senha.txt -M ftp
```

* **Flags explicadas:**

  * `-h 192.168.100.10`: Especifica o IP do host alvo.

  * `-u msfadmin`: Define o nome de usuário a ser testado.

  * `-P /tmp/senha.txt`: Aponta para o dicionário com as senhas a serem validadas.

  * `-M ftp`: Carrega o módulo específico para o protocolo FTP.

**Resultado obtido:**

```text
ACCOUNT FOUND: [ftp] Host: 192.168.100.10 User: msfadmin Password: msfadmin [SUCCESS]
```

> **Print Recomendado 4:** `images/05_medusa_sucesso_account_found.png`
> *Captura do terminal do Kali Linux com o Medusa exibindo a mensagem "ACCOUNT FOUND".*

## 🚀 4. Cenário B: Ataque de Força Bruta Web no DVWA (Burp Suite)

### Etapa 1: Acesso ao Ambiente e Interceptação

1. **Acesso à Aplicação:** Navegou-se até a aplicação vulnerável **DVWA (Damn Vulnerable Web Application)** hospedada na VM Metasploitable 2 (`http://192.168.100.10/dvwa/`).

2. **Nível de Segurança:** O parâmetro de segurança foi ajustado para `Low` e acessou-se a secção **Brute Force**.

3. **Interceptação:** Com o **Burp Suite Proxy** ativado no navegador, enviou-se uma tentativa de login de teste (`username: admin`, `password: janaina`). A requisição HTTP `GET` foi retida no proxy.

```http
GET /dvwa/vulnerabilities/brute/?username=admin&password=janaina&Login=Login HTTP/1.1
Host: 192.168.100.10
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36
Cookie: security=low; PHPSESSID=8f7e7ab9c8d85b3db206933e7256eef7
```

> **Print Recomendado 5:** `images/06_burp_proxy_intercept.png`
> *Captura da requisição interceptada na aba Proxy > Intercept do Burp Suite.*

### Etapa 2: Configuração e Execução no Burp Intruder

1. **Envio ao Intruder:** Clicou-se com o botão direito na requisição e selecionou-se **Send to Intruder** (`Ctrl + I`).

2. **Posicionamento de Payloads:** Na sub-aba **Positions**, configurou-se o ataque como **Sniper** e definiu-se a marcação no valor do parâmetro de senha: `password=§janaina§`.

3. **Carregamento da Wordlist:** Na aba lateral **Payloads**, carregou-se o arquivo de dicionário:

   ```bash
   /usr/share/wordlists/rockyou.txt
   ```

4. **Execução:** Acionou-se o botão **Start attack** para iniciar os testes automatizados.

> **Print Recomendado 6:** `images/07_intruder_positions_payloads.png`
> *Painel do Intruder com o parâmetro delimitado e a lista rockyou.txt carregada.*

### Etapa 3: Análise dos Resultados via Métrica de Resposta (Length)

Na janela de progresso do ataque, monitoraram-se as colunas `Status code` e `Length`:

* **Tentativas Incorretas:** Retornaram o código HTTP `200` com um tamanho fixo (ex: ~`4923` bytes) correspondente à página de erro.

* **Tentativa Bem-sucedida:** A linha contendo a credencial correta gerou uma variação no tamanho da resposta (*Length*) devido ao carregamento da mensagem de boas-vindas da aplicação.

> **Print Recomendado 7:** `images/08_intruder_attack_results.png`
> *Janela de resultados do Intruder destacando a resposta com valor destoante na coluna Length.*

## 🚀 5. Cenário C: Password Spraying em SMB com Enumeração

Diferente de um ataque tradicional de força bruta (que testa múltiplas senhas em uma única conta, correndo alto risco de bloqueio de senha), o **Password Spraying** testa uma única senha comum contra uma lista de múltiplos nomes de usuários válidos.

### Etapa 1: Enumeração de Usuários (SMB / RPC)

Antes de iniciar os testes de autenticação, executou-se a fase de reconhecimento para levantar nomes de contas válidas no alvo (`192.168.100.10`).

1. **Identificação do Serviço SMB:**
   A varredura com Nmap confirmou a presença dos serviços SMB ativos nas portas TCP 139 e 445.

2. **Enumeração via `enum4linux`:**
   Através de conexões nulas (*Null Sessions*), extraíram-se dados do sistema remoto e a lista de contas de usuário:
   ```bash
   enum4linux -U 192.168.100.10
   ```
   *Alternativa via rpcclient:*
   ```bash
   rpcclient -U "" -N 192.168.100.10 -c "enumdomusers"
   ```

3. **Criação do Dicionário de Usuários (`users.txt`):**
   Os usuários identificados no sistema (ex: `msfadmin`, `user`, `postgres`, `service`, `root`) foram organizados em um arquivo texto local.

> **Print Recomendado 8:** `images/09_enumera_usuarios_smb.png`
> *Terminal exibindo o retorno da enumeração de contas via enum4linux/rpcclient e a criação da lista users.txt.*

### Etapa 2: Ataque de Password Spraying com Hydra

Com o arquivo de usuários pronto, utilizou-se a ferramenta **Hydra** para testar uma única senha padrão (ex: `service`) contra todos os usuários enumerados.

```bash
hydra -L users.txt -p "service" 192.168.100.10 smb
```

* **Flags explicadas:**

  * `-L users.txt`: Aponta para a lista contendo múltiplos nomes de usuário.

  * `-p "service"`: Define a senha única utilizada no teste direcionado (*Password Spraying*).

  * `192.168.100.10 smb`: Define o IP de destino e o serviço alvo.

**Resultado Obtido:**

```text
[445][smb] host: 192.168.100.10   login: service   password: service [SUCCESS]
```

> **Print Recomendado 9:** `images/10_password_spraying_sucesso.png`
> *Terminal do Kali Linux mostrando o sucesso da autenticação na conta "service" via Password Spraying em SMB.*

## 🛡️ 6. Mitigações e Boas Práticas de Segurança

### Para o Serviço FTP:

1. **Políticas de Senhas Fortes:** Exigir senhas complexas e longas para mitigar a efetividade de dicionários como `rockyou.txt`.

2. **Bloqueio por Tentativas (Account Lockout):** Utilizar ferramentas de defesa como `fail2ban` no host para banir temporariamente IPs que excedam a taxa de tentativas incorretas.

3. **Migração para Protocolos Criptografados:** Substituir o FTP plano por **SFTP** ou **FTPS**.

### Para Aplicações Web (Formulários de Login):

1. **Proteção Anti-Automação (CAPTCHA):** Implementar mecanismos de validação humana em formulários de autenticação para inviabilizar ferramentas automatizadas (como o Burp Intruder).

2. **Rate Limiting e Bloqueio de IP:** Aplicar limites de requisições por IP/Sessão e bloqueios temporários de conta após sucessivas falhas de login.

3. **Respostas Genéricas de Autenticação:** Evitar mensagens de erro detalhadas que diferenciem erros de utilizador e erros de senha.

### Para o Serviço SMB e Prevenção de Password Spraying:

1. **Desabilitar Sessões Nulas (Null Sessions):** Restringir acessos anônimos para evitar a enumeração de contas por usuários não autenticados no protocolo SMB.

2. **Múltiplo Fator de Autenticação (MFA):** Adotar MFA para impedir que o comprometimento de uma senha simples garanta acesso aos recursos do sistema.

3. **Monitoramento e SIEM:** Configurar alertas no SIEM para identificar picos de tentativas de autenticação oriundos de uma mesma origem testando a mesma senha contra múltiplas contas em um curto intervalo de tempo.

*Documentação gerada para fins estritamente educacionais e de construção de portfólio profissional em segurança da informação.*