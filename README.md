🛡️ Auditoria de Segurança: Ataques de Força Bruta e Password Spraying
Este repositório contém a documentação e execução do desafio de projeto sobre ataques de força bruta, realizado no ambiente Kali Linux (WSL2) utilizando o alvo Metasploitable 2 via Docker.

O objetivo principal foi simular um ataque para identificar credenciais frágeis nos serviços FTP e SMB, superando barreiras comuns de rede em ambientes virtualizados e demonstrando técnicas de mitigação.

🛠️ Ferramentas Utilizadas
Kali Linux: Sistema operacional base para os testes.

Docker: Plataforma de virtualização para hospedar o alvo vulnerável.

Nmap: Ferramenta para mapeamento de portas e descoberta de serviços.

Medusa/Hydra: Ferramentas utilizadas para a execução do ataque de força bruta.

🚀 Passo a Passo da Execução
1. Preparação do Alvo (Docker)
Iniciamos o container do Metasploitable 2 mapeando as portas principais para o host.

Bash
sudo docker run -d --name alvo-metasploitable -p 21:21 -p 445:445 tleemcjr/metasploitable2
Nota: O comando baixa e executa a máquina vulnerável em segundo plano. As portas 21 (FTP) e 445 (SMB) foram expostas para permitir a simulação de ataque externo.

2. Reconhecimento (Nmap)
Antes de iniciar a exploração, validamos se os serviços estavam acessíveis e prontos para conexão.

Bash
nmap -p 21,445 127.0.0.1
Explicação: O Nmap confirmou que as portas estavam no estado , garantindo que o servidor estava operacional para os testes de autenticação.open

3. Criação de Listas de Palavras
Desenvolvemos listas de palavras contendo usuários e senhas comuns para realizar o teste de força bruta.

Bash
echo -e "admin\nuser\nmsfadmin" > users.txt
echo -e "123456\npassword\nadmin\nmsfadmin\nqwerty" > passwords.txt
4. Execução do Ataque (THC-Hydra)
Devido a instabilidades de socket apresentadas pelo Medusa no ambiente WSL2, optamos pelo THC-Hydra pela sua robustez e controle de conexões.

Bash
hydra -t 1 -l msfadmin -P passwords.txt 127.0.0.1 smb -V
Explicação dos Parâmetros:

-t 1: Define apenas uma tentativa por vez (evita bloqueios por excesso de conexões simultâneas).

-l msfadmin: Define o usuário alvo específico.

-P passwords.txt: Define a lista de senhas para o ataque.

smb: Protocolo alvo (Bloco de Mensagem do Servidor).

-V: Modo verboso, exibindo cada tentativa em tempo real.

🔍 Solução de Problemas (Solução de Problemas)
Durante a auditoria, enfrentamos o erro: .all children were disabled due too many connection errors

Causa: O Windows Firewall e a proteção nativa do Metasploitable detectaram o tráfego automatizado e interromperam a conexão.

Solução: Reduzimos o paralelismo para e reiniciamos o serviço de rede do Docker. Esta abordagem "serial" permitiu contornar o rate limiting e concluir a autenticação com sucesso.-t 1

📊 Resultados Obtidos
Os seguintes vetores de acesso foram confirmados após os testes:

Serviço SMB (Porta 445)
Status: Vulnerável.

Credencial Identificada: msfadminmsfadmin

Impacto: Acesso total ao sistema de arquivos compartilhado e potencial escalonamento de privilégios.

Serviço FTP (Porta 21)
Status: Vulnerável.

Credencial Identificada: msfadminmsfadmin

Impacto: Capacidade de transferência de arquivos maliciosos e exfiltração de dados sensíveis.

🛡️ Sugestões de Mitigação (Endurecimento)
Para fortalecer a postura de segurança do ambiente, recomenda-se:

Políticas de Bloqueio (Account Lockout): Configurar o sistema para bloquear automaticamente a conta após 3 tentativas de login falhas.

Autenticação Multifator (MFA): Implementar uma segunda camada de autenticação para todos os acessos administrativos.

Monitoramento Ativo com Fail2Ban: Instalar o serviço para analisar logs em tempo real e banir IPs com comportamento suspeito.

Políticas de Senhas Fortes: Implementar requisitos de complexidade e proibir explicitamente o uso de senhas que coincidam com o nome do usuário.

Conclusão para o GitHub
Laboratório de Cibersegurança (DIO): Simulação de ataques de força bruta em Metasploitable 2. O projeto demonstra a exploração de credenciais no protocolo SMB utilizando THC-Hydra, focando na resolução de conflitos de rede em ambientes virtualizados e na aplicação de boas práticas de segurança defensiva.

<img width="1349" height="340" alt="Screenshot_1" src="https://github.com/user-attachments/assets/1c87a44e-1456-4333-91d5-21db7e1a48c9" />

