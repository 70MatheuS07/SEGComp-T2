# SSH Honeypot com Cowrie e Força Bruta com Medusa

Este tutorial explica como configurar um honeypot SSH usando **Cowrie** e realizar ataques de força bruta usando **Medusa**.

## Requisitos

- Sistema baseado em Linux (Ubuntu, Debian, etc.).
- Acesso root ou sudo.
- Python 3 instalado.

## 1. Atualizar o Sistema

Antes de começar, certifique-se de que seu sistema está atualizado:

```bash
sudo apt-get update && sudo apt-get upgrade
```

## 2. Instalar dependências

Instale as dependências necessárias para o Cowrie:

```bash
sudo apt-get install git python3-venv python3-dev libssl-dev libffi-dev build-essential
```

## 3. Clonar o repositório do Cowrie

Clone o repositório oficial do Cowrie no GitHub:

```bash
git clone https://github.com/cowrie/cowrie.git
```

## 4. Criar e Ativar o Ambiente Virtual

Navegue até o diretório do Cowrie e crie um ambiente virtual Python:

```bash
cd cowrie
python3 -m venv cowrie-env
source cowrie-env/bin/activate
```

## 5. Instalar Dependências Python

Atualize o **pip** e instale as dependências:

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

## 6. Configurar o Cowrie

Copie o arquivo de configuração de exemplo:

```bash
cp etc/cowrie.cfg.dist etc/cowrie.cfg
```

## 7. Iniciar o Cowrie

```bash
bin/cowrie start
```

## 8. Verificar os Logs do Cowrie

Monitore os logs do Cowrie para garantir que ele está funcionando corretamente:

```bash
tail -f var/log/cowrie/cowrie.log
```

## 9. Instalar o Medusa

```bash
sudo apt-get install medusa
```

## 10. Preparar o Arquivo de Senhas

Crie um arquivo password_list.txt com as senhas a serem testadas:

```bash
nano password_list.txt
```
Exemplo de conteúdo:

```bash
123456
password
admin
root
Senh@Fort3
```
## 11. Configurar um Usuário com Senha Fixa no Cowrie

Para definir um usuário específico com senha fixa no Cowrie:

### a. Copiar os arquivos

```bash
cp etc/userdb.example etc/userdb.txt
```

### b. Editar o Arquivo userdb.txt

Edite o arquivo para adicionar o usuário matheus com a senha Senh@Fort3:

```bash
nano etc/userdb.txt
```

Modifique o arquivo para ter o seguinte conteúdo:

```bash
# Field #3 contains the password 
# '*' for any username or password
# '!' at the start of a password will not grant this password access
# '/' can be used to write a regular expression 
#
matheus:x:Senh@Fort3
root:x:!root
root:x:!123456
```

Salve e saia (Ctrl + O, Enter, Ctrl + X).

### c. Reiniciar o Cowrie

Reinicie o Cowrie para aplicar as mudanças:

```bash
bin/cowrie stop
```

```bash
bin/cowrie start
```

## 12. Verificar a Configuração do Usuário

Monitore os logs para garantir que o usuário matheus com senha Senh@Fort3 está configurado corretamente:

```bash
tail -f var/log/cowrie/cowrie.log
```

## 13. Executar o Ataque com Medusa para o Usuário "matheus"

```bash
medusa -h 127.0.0.1 -u matheus -P password_list.txt -M ssh -n 2222
```
