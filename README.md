# Projeto de Prática em Cibersegurança: Ataque de Força Bruta em Serviço FTP

Este repositório contém a documentação prática do Cenário A do desafio de Cibersegurança (Bootcamp DIO / Santander). O objetivo principal é demonstrar o ciclo de vida de uma auditoria de segurança em um ambiente simulado e controlado: desde a validação da topologia de rede e reconhecimento do alvo até a execução de um ataque de força bruta contra o serviço FTP.

# 📌 1. Visão Geral do Ambiente e Topologia

O laboratório foi montado em um ambiente de virtualização isolado usando o Oracle VM VirtualBox, garantindo a segurança dos testes e a separação da rede host/pública.

Máquina Atacante: Kali Linux (IP: 192.168.100.20/24)

Máquina Alvo: Metasploitable 2 (IP: 192.168.100.10/24)

Modo de Rede: Rede Interna (CLIENTE-LAN)

Endereçamento: Atribuição estática de IPs no mesmo segmento de rede sem servidor DHCP ativo.

    Print Recomendado 1: images/01_config_rede_virtualbox.png

    Captura de tela das Configurações de Rede da VM no VirtualBox mostrando a opção "Rede Interna" e o nome CLIENTE-LAN.

# 🛠️ 2. Ferramentas Utilizadas

Ferramenta          Finalidade no Laboratório
ping                Teste básico de alcance de rede e conectividade ICMP entre as máquinas.
nmap                Reconhecimento de portas abertas e detecção de versão de serviços (-sV).
medusa              Teste de força bruta paralelo e modular contra o serviço FTP.
Wordlists           Dicionários de senhas (rockyou.txt e wordlist direcionada) para quebra de credenciais.

# 🚀 3. Passo a Passo do Procedimento Executado

# Etapa 1: Validação de Conectividade de Rede

Após realizar a configuração manual dos IPs nas interfaces virtuais (eth0) de ambas as VMs na sub-rede 192.168.100.0/24, foi validada a comunicação de rede enviando pacotes ICMP da máquina atacante (Kali Linux) para o alvo (Metasploitable 2).

ping -c 4 192.168.100.10

Flags explicadas:

-c 4: Define o limite exacto de envio de 4 pacotes ICMP antes de finalizar a execução.

Print Recomendado 2: images/02_ping_conectividade.png

Captura do terminal do Kali Linux exibindo a resposta dos 4 pacotes (0% packet loss).

# Etapa 2: Mapeamento e Reconhecimento com Nmap

Com a conectividade confirmada, foi executado um escaneamento no alvo para verificar o estado da porta 21 (FTP) e identificar a versão exata do software em execução.

nmap -sV -p 21 -n 192.168.100.10

Flags explicadas:

-sV: Solicita a sondagem do serviço para determinar a versão exata do software (vsftpd 2.3.4).

-p 21: Restringe o escaneamento especificamente à porta padrão do FTP.

-n: Desativa a resolução DNS para evitar erros de timeout ou asserções de biblioteca.

Resultado da varredura:

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4

Print Recomendado 3: images/03_nmap_reconhecimento_ftp.png

Captura do terminal mostrando o relatório do Nmap com a porta 21/tcp aberta e a versão vsftpd 2.3.4.

# Etapa 3: Ataque de Força Bruta em FTP com Medusa

Com a confirmação do serviço FTP ativo, foi utilizada a ferramenta Medusa para simular uma tentativa de autenticação por força bruta baseada em dicionário contra o usuário padrão msfadmin.

medusa -h 192.168.100.10 -u msfadmin -P /tmp/senha.txt -M ftp

Flags explicadas:

-h 192.168.100.10: Especifica o IP do host alvo.

-u msfadmin: Define o nome de usuário a ser testado.

-P /tmp/senha.txt: Aponta para a wordlist/dicionário com as senhas a serem validadas.

-M ftp: Carrega o módulo específico para ataques ao protocolo FTP.

Resultado obtido:

ACCOUNT FOUND: [ftp] Host: 192.168.100.10 User: msfadmin Password: msfadmin [SUCCESS]

Print Recomendado 4: images/04_medusa_sucesso_account_found.png

Captura do terminal do Kali Linux com o Medusa exibindo a mensagem "ACCOUNT FOUND" em verde/destaque.

# 4 🛡️. Mitigações e Boas Práticas de Segurança

Para defender o serviço FTP contra ataques de força bruta semelhantes ao demonstrado:

Políticas de Senhas Fortes: Exigir senhas complexas e de grande comprimento para evitar a quebra via dicionários conhecidos (como a rockyou.txt).

Bloqueio de Conta (Account Lockout): Implementar mecanismos como o fail2ban no servidor Linux para bloquear temporariamente endereços IP após um número específico de tentativas mal-sucedidas.

Uso de Protocolos Seguros: Substituir o FTP não criptografado por alternativas com suporte a criptografia, como SFTP (SSH File Transfer Protocol) ou FTPS.

Documentação gerada para fins educacionais e de portfólio profissional.