# sess-o3

# Guia de Implementação: Administração de Utilizadores, Permissões Seguras e Auditoria em Linux

Este documento descreve, de forma ordenada e cronológica, os procedimentos executados para a gestão de utilizadores, isolamento de diretórios confidenciais e auditoria de segurança no sistema.

---

## Passo 1: Gestão de Utilizadores e Grupos

Com base nas evidências de `sessao3_passo1_utilizadores.png.jpg`, foram provisionados novos utilizadores e configuradas as respetivas credenciais e privilégios.

1. **Criação das contas de utilizador com diretório home e shell Bash padrão:**
```bash
sudo useradd -m -s /bin/bash alice
sudo useradd -m -s /bin/bash bob

```


2. **Validação da criação dos utilizadores (UID, GID e Grupos):**
```bash
id alice
id bob

```


3. **Verificação dos diretórios criados no sistema de ficheiros:**
```bash
ls -la /home/

```


4. **Consulta rápida dos registos no ficheiro de utilizadores do sistema:**
```bash
cat /etc/passwd | grep -E 'alice|bob'

```


5. **Definição de palavras-passe seguras para as novas contas:**
```bash
sudo passwd alice
sudo passwd bob

```


6. **Criação do grupo de segurança e atribuição de privilégios secundários:**
```bash
sudo groupadd seguranca
sudo usermod -aG seguranca alice

```


7. **Auditoria local da correta associação do utilizador ao grupo:**
```bash
groups alice
id alice
cat /etc/group | grep seguranca

```



---

## Passo 2: Verificação do Estado das Contas e Bloqueios

Conforme demonstrado em `sessao3_passo2_shadow.png.jpg`, foi inspecionada a integridade dos hashes de segurança e simulado o bloqueio administrativo de contas.

1. **Auditoria cruzada entre `/etc/passwd` e `/etc/shadow` para validar hashes criptográficos:**
```bash
sudo cat /etc/shadow | grep -E 'alice|bob'

```


2. **Leitura completa do ficheiro shadow para validação de políticas globais:**
```bash
cat /etc/shadow

```


3. **Bloqueio administrativo de conta (Lock) por motivos de segurança:**
```bash
sudo passwd -l bob

```


> **Nota de Engenharia:** O comando `passwd -l` insere um prefixo `!` no hash da palavra-passe no ficheiro `/etc/shadow`, invalidando a autenticação do utilizador `bob` até que seja revertido.



---

## Passo 3: Criação e Restrição do Diretório Confidencial

De acordo com `ssessao3_passo2_chmod750.png.jpg`, foi criado um ambiente restrito para partilha de informação confidencial com base no princípio do privilégio mínimo.

1. **Provisionamento do diretório estruturado:**
```bash
sudo mkdir -p /opt/confidencial

```


2. **Criação de ficheiros simulados com dados sensíveis:**
```bash
sudo bash -c 'echo "Relatorio Anual 2026 - CONFIDENCIAL" > /opt/confidencial/relatorio_anual.txt'
sudo bash -c 'echo "Dados Salariais - RESTRITO" > /opt/confidencial/salarios.txt'

```


3. **Análise das permissões iniciais herdadas (perigosamente permissivas):**
```bash
ls -la /opt/
ls -la /opt/confidencial/

```


4. **Alteração de propriedade e aplicação de máscara de permissão restritiva (`750`):**
```bash
sudo chown root:seguranca /opt/confidencial
sudo chmod 750 /opt/confidencial

```


5. **Validação detalhada dos metadados e permissões finais do nó (Inode):**
```bash
ls -la /opt/
stat /opt/confidencial

```


> **Resultado:** Permissões definidas como `drwxr-x---`. Apenas o `root` possui controlo total (leitura, escrita e execução) e membros do grupo `seguranca` (como a `alice`) possuem permissões de leitura e transição. Outros utilizadores não possuem qualquer acesso.



---

## Passo 4: Teste de Isolamento de Acessos

Em `sessao3_passo3_isolamento.png.jpg`, testou-se a eficácia do isolamento lógico alternando o contexto de execução entre os utilizadores.

1. **Teste de acesso com o utilizador `alice` (Membro do grupo autorizado):**
```bash
su alice
ls /opt/confidencial
exit

```


2. **Teste de acesso com o utilizador `bob` (Não autorizado / Conta bloqueada):**
```bash
su bob
ls /opt/confidencial
exit

```



---

## Passo 5: Geração de Relatório Automatizado de Auditoria

Com base em `auditoria_acessos.txt.jpg`, todos os passos anteriores foram consolidados num único ficheiro de relatório para auditorias futuras.

```bash
# 1. Criação da estrutura de relatórios
mkdir -p ~/relatorios/sessao3

# 2. Inicialização do cabeçalho do relatório
echo "=== AUDITORIA DE ACESSOS - SESSAO 3 ===" > ~/relatorios/sessao3/auditoria_acessos.txt
echo "Data: \$(date)" >> ~/relatorios/sessao3/auditoria_acessos.txt

# 3. Registo de utilizadores validados
echo "--- UTILIZADORES ---" >> ~/relatorios/sessao3/auditoria_acessos.txt
cat /etc/passwd | grep -E 'alice|bob' >> ~/relatorios/sessao3/auditoria_acessos.txt

# 4. Registo de grupos e pertenças
echo "--- GRUPOS ---" >> ~/relatorios/sessao3/auditoria_acessos.txt
groups alice >> ~/relatorios/sessao3/auditoria_acessos.txt
groups bob >> ~/relatorios/sessao3/auditoria_acessos.txt

# 5. Estado das permissões do diretório restrito
echo "--- PERMISSOES DIRETORIO CONFIDENCIAL ---" >> ~/relatorios/sessao3/auditoria_acessos.txt
ls -la /opt/ >> ~/relatorios/sessao3/auditoria_acessos.txt

# 6. Exibição final do relatório gerado
cat ~/relatorios/sessao3/auditoria_acessos.txt

```
