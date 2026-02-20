# 🛡️ Projeto Prático: Simulação de Ataque de Força Bruta e Mitigação

## 🎯 Objetivo do Projeto
Implementar e documentar um laboratório prático de cibersegurança utilizando **Kali Linux**, a ferramenta **Medusa** e ambientes vulneráveis (**Metasploitable 2** e **DVWA**). O foco é simular cenários reais de ataques de força bruta contra diferentes protocolos e, em seguida, propor medidas robustas de prevenção e mitigação.

## 🛠️ Configuração do Ambiente
O laboratório foi construído utilizando o VirtualBox com uma rede interna isolada (Host-only) para garantir um ambiente controlado e seguro.
* **Máquina Atacante:** Kali Linux (IP Fixo: `192.168.10.11`)
* **Máquina Alvo:** Metasploitable 2 (IP Fixo: `192.168.10.10`)
* **Wordlist Utilizada:** Arquivo customizado `senhas.txt` (disponível neste repositório), contendo senhas comuns e credenciais padrão do sistema.

## 🚀 Execução dos Ataques

### 1. Ataque de Força Bruta em FTP (Porta 21)
O objetivo foi comprometer o serviço de transferência de arquivos utilizando o usuário padrão do sistema.
* **Ferramenta:** Medusa
* **Comando Utilizado:** `medusa -h 192.168.10.10 -u msfadmin -P senhas.txt -M ftp`
* **Resultado:** Credencial encontrada com sucesso (`msfadmin:msfadmin`).
* *(Veja a captura de tela na pasta `/imagens/ftp-success.png`)*

### 2. Password Spraying em SMB (Porta 139/445)
O foco foi explorar o serviço de compartilhamento de arquivos e impressoras da rede (Server Message Block).
* **Ferramenta:** Medusa
* **Comando Utilizado:** `medusa -h 192.168.10.10 -u msfadmin -P senhas.txt -M smbnt`
* **Resultado:** Acesso permitido ao compartilhamento `ADMIN$` com sucesso.
* *(Veja a captura de tela na pasta `/imagens/smb-success.png`)*

### 3. Ataque Web em Formulário de Login (DVWA - Porta 80)
O objetivo era realizar bypass na tela de login da aplicação vulnerável DVWA. 
* **Troubleshooting Técnico:** Durante a execução com o Medusa (módulo `web-form`), a ferramenta apresentou falhas contínuas de `error code 302`. Isso ocorre pois o DVWA exige controle de Cookies de Sessão e realiza um redirecionamento (302 Found) após o login, comportamento que o Medusa não processa nativamente.
* **Pivotamento de Ferramenta:** Para concluir o ataque de forma eficiente em aplicações web com redirecionamento, a estratégia foi alterada para o **Hydra**, que lida perfeitamente com requisições HTTP POST e validações de strings.
* **Ferramenta:** Hydra
* **Comando Utilizado:** `hydra -l admin -P senhas.txt 192.168.10.10 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"`
* **Resultado:** Senha de administrador web localizada com sucesso (`admin:password`).
* *(Veja a captura de tela na pasta `/imagens/web-success.png`)*

## 🛡️ Recomendações de Mitigação e Prevenção
Para evitar que ataques como os realizados neste laboratório tenham sucesso em ambientes corporativos reais, as seguintes políticas devem ser implementadas:

1. **Políticas de Bloqueio de Conta (Account Lockout):** Configurar o sistema para bloquear a conta do usuário temporariamente (ex: 15 minutos) após 3 ou 5 tentativas falhas de login. Isso inviabiliza a velocidade da Força Bruta.
2. **Desativação de Credenciais Padrão:** Nunca manter usuários default como `admin`, `msfadmin` ou senhas como `password`. Os sistemas devem forçar a troca de senha no primeiro login.
3. **Senhas Fortes e Complexas:** Exigir senhas longas (mínimo de 12 a 16 caracteres), combinando letras maiúsculas, minúsculas, números e símbolos.
4. **Múltiplos Fatores de Autenticação (MFA/2FA):** A implementação de MFA em serviços de FTP, SMB e páginas Web garante que, mesmo que o atacante descubra a senha via Força Bruta, ele não consiga o acesso final sem o token gerado no celular do usuário.
5. **Monitoramento e Alertas (SIEM/IDS):** Implementar sistemas de detecção de intrusão que alertem a equipe de segurança caso haja um pico anormal de tentativas de login originadas de um mesmo endereço IP (comportamento típico do Medusa/Hydra).
