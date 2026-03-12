Este repositório contém a documentação e execução do desafio de projeto sobre ataques de força bruta, realizado no ambiente Kali Linux (WSL2) utilizando o alvo Metasploitable 2 via Docker.

O objetivo principal foi simular um ataque para identificar credenciais frágeis nos serviços FTP e SMB, superando barreiras comuns de rede em ambientes virtualizados.

🛠️ Ferramentas Utilizadas
Kali Linux: Sistema operacional base.
Docker: Para hospedar o alvo vulnerável.
Nmap: Mapeamento de portas e serviços.
Medusa/Hidra: Execução do ataque de força bruta.

🚀 Passo a Passo da Execução
1.Preparação do Alvo (Docker)
Iniciamos o container do Metasploitable 2 mapeando as portas principais para o host.
---------------------------------------------------------------------------------------------
Bash
sudo docker run -d --name alvo-metasploitable -p 21:21 -p 445:445 tleemcjr/metasploitable2
---------------------------------------------------------------------------------------------
Explicação: O comando baixa e executa a máquina vulnerável em segundo plano. Mapeamos a porta 21 (FTP) e 445 (SMB) para permitir o ataque externo.


2. Reconhecimento (Nmap)
Antes de atacar, validamos se os serviços estavam realmente acessíveis.
-----------------------------------------------------------------------------------------
Bash
nmap -p 21,445 127.0.0.1
-----------------------------------------------------------------------------------------
Explicação: O Nmap confirmou que as portas estavam no estado open3, garantindo que o servidor estava pronto para receber conexões. 


3. Criação de Listas de Palavras
 Criamos arquivos de texto com usuários e senhas comuns para o teste.
-----------------------------------------------------------------------------------------
Bash
echo -e "admin\nuser\nmsfadmin" > users.txt
echo -e "123456\npassword\nadmin\nmsfadmin\nqwerty" > passwords.txt
-----------------------------------------------------------------------------------------


8. Execução do Ataque (Hydra)
Após identificar que o Medusa apresentava instabilidade de socket no ambiente WSL2, utilizamos o THC-Hydra com configurações de estabilidade.
-----------------------------------------------------------------------------------------
Bashhydra -t 1 -l msfadmin -P passwords.txt 127.0.0.1 smb -V
-----------------------------------------------------------------------------------------
Explicação dos Parâmetros:
-t 1: Define apenas uma tentativa por vez (evita bloqueios por excesso de conexões).
-l msfadmin: Define o usuário alvo.
-P passwords.txt: Lista de senhas para o ataque.smb: Protocolo alvo (Bloco de Mensagem do Servidor).
-V: Modo verboso, exibindo cada tentativa em tempo real.


🔍 Solução de Problemas (Desafios Encontrados)
Durante a auditoria, enfrentamos o erro: .all children were disabled due too many connection errors
Causa: O Windows Firewall e a proteção nativa do Metasploitable detectaram o tráfego automatizado e cortaram a conexão.
Solução: Reduzimos o paralelismo para -t 1 e reiniciamos o serviço de rede do Docker, o que permitiu concluir a autenticação com sucesso.


📊 Resultados Obtidos
Após a execução dos testes de invasão e análise de tráfego, os seguintes vetores de acesso foram confirmados:

Serviço SMB (Porta 445):
Status: Vulnerável.
Credencial Identificada: Usuário / Senha .msfadminmsfadmin
Impacto: Acesso total ao sistema de arquivos compartilhado e potencial escalonamento de privilégios.

Serviço FTP (Porta 21):
Status: Vulnerável (identificado via persistência de conexão).

Credencial Identificada: Usuário / Senha .msfadminmsfadmin
Impacto: Capacidade de transferência de arquivos maliciosos e exfiltração de dados sensíveis.

🛡️ Sugestões de Mitigação (Endurecimento)
Para corrigir as vulnerabilidades encontradas e fortalecer a postura de segurança do ambiente, recomenda-se a aplicação das seguintes camadas de defesa:

Políticas de Bloqueio (Bloqueio de Contas): Configurar o sistema para bloquear automaticamente a conta após 3 tentativas de login falhas em um curto intervalo de tempo.

Autenticação Multifator (MFA): Implementar uma segunda camada de autenticação para todos os acessos administrativos, garantindo que a posse da senha não seja suficiente para a invasão.

Monitoramento Ativo com Fail2Ban: Instalar e configurar o serviço Fail2Ban para analisar logs em tempo real e banir temporariamente o endereço IP de qualquer origem que realize ataques de força bruta.

Políticas de Senhas Fortes: Implementar requisitos de complexidade (letras maiúsculas, números e símbolos) e proibir explicitamente o uso de senhas que coincidam com o nome do usuário ou termos genéricos (como "admin" ou "123456").
