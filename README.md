# Trabalho 2 - Segurança em Computação
Este trabalho tem como objetivo instalar e configurar uma ferramenta de segurança, simular ataques cibernéticos de força bruta e documentar, criando um tutorial do processo.

### Participantes:
- Lucas Gabriel de Oliveira Costa
- Matheus de Oliveira Lima
- Matheus Lopes Ferreira Lima

# SSH Honeypot com Cowrie e Força Bruta com Medusa

Este tutorial explica como configurar um honeypot SSH usando **Cowrie** e realizar ataques de força bruta usando **Medusa**. Ele serve para criar um ambiente controlado onde possíveis invasores podem ser atraídos, e suas tentativas de acesso são registradas para análise posterior. O uso do Medusa permite simular ataques de força bruta para testar a eficácia da configuração do honeypot.

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

> Cowrie é um honeypot projetado para registrar tentativas de login SSH (e Telnet), armazenando detalhes das tentativas, como credenciais usadas e comandos executados.

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

> O arquivo de configuração do Cowrie (`cowrie.cfg`) define como o honeypot irá operar. Ao copiar o arquivo de exemplo para etc/cowrie.cfg, estamos criando uma configuração inicial, que pode ser personalizada conforme necessário.
>
> Dentro do arquivo de configuração, alguns parâmetros que podem ser configurados incluem:
> - Porta que o honeypot SSH irá usar.
> - Caminho para os logs.
> - Endereço do servidor que receberá os dados capturados (se aplicável).

## 7. Iniciar o Cowrie

```bash
bin/cowrie start
```

> Aqui, iniciamos o honeypot. Ele começa a escutar por conexões SSH na porta especificada e registra todas as interações.

## 8. Verificar os Logs do Cowrie

Monitore os logs do Cowrie para garantir que ele está funcionando corretamente:

```bash
tail -f var/log/cowrie/cowrie.log
```

> Os logs são um aspecto fundamental de um honeypot, pois registram todos os detalhes das tentativas de invasão. O comando acima monitora o arquivo de log do Cowrie em tempo real.

## 9. Instalar o Medusa

```bash
sudo apt-get install medusa
```

> Medusa é uma ferramenta de força bruta que pode ser usada para testar a resistência de servidores a ataques de login repetidos. Ao instalar o Medusa, estamos preparando o ambiente para simular ataques contra o honeypot.

## 10. Preparar o Arquivo de Senhas

Crie um arquivo `password_list.txt` com as senhas a serem testadas:

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

> Aqui, criamos um arquivo contendo uma lista de senhas que o Medusa usará para tentar acessar o honeypot. Este arquivo simula um ataque de força bruta, testando várias combinações de senha.

## 11. Configurar um Usuário com Senha Fixa no Cowrie

Para simular um ataque de força bruta eficaz, é necessário configurar um usuário específico no honeypot com uma senha fixa.

### a. Copiar os arquivos

```bash
cp etc/userdb.example etc/userdb.txt
```

> O arquivo `userdb.txt` é onde os usuários e suas respectivas senhas para o honeypot são definidos. Primeiro, copiamos o arquivo de exemplo para um arquivo real que será utilizado pelo Cowrie.

### b. Editar o Arquivo userdb.txt

Edite o arquivo para adicionar o usuário matheus com a senha `Senh@Fort3`:

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

> Dentro do arquivo userdb.txt, definimos um usuário chamado matheus com a senha `Senh@Fort3`. O formato é:
>
> - O primeiro campo é o nome de usuário.
> - O terceiro campo é a senha.
> - Um * pode ser usado para aceitar qualquer combinação de usuário ou senha.
> - Um ! no início da senha bloqueia o acesso com essa senha.
> - ⚠️ O segundo campo (`:x:`) não é utilizado pelo Cowrie e é um remanescente dos arquivos de senha Unix tradicionais (como `/etc/passwd`). Ele não tem função real aqui, mas precisa estar presente para seguir a estrutura esperada. No contexto do Unix, ele indicaria a presença de uma senha armazenada em outro local (geralmente no arquivo `/etc/shadow`), mas para o Cowrie, esse campo é apenas um marcador e pode ser ignorado.

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

Monitore os logs para garantir que o usuário matheus com senha `Senh@Fort3` está configurado corretamente:

```bash
tail -f var/log/cowrie/cowrie.log
```

## 13. Executar o Ataque com Medusa para o Usuário "matheus"

Finalmente, usamos o Medusa para realizar um ataque de força bruta contra o usuário matheus, tentando todas as senhas do arquivo `password_list.txt`.

```bash
medusa -h 127.0.0.1 -u matheus -P password_list.txt -M ssh -n 2222
```

> - -h 127.0.0.1: define o endereço IP do alvo, que aqui é o próprio localhost.
> - -u matheus: especifica o usuário alvo do ataque.
> - -P password_list.txt: define o arquivo contendo a lista de senhas a serem testadas.
> - -M ssh: define o módulo (neste caso, SSH) para o ataque.
> - -n 2222: define a porta onde o Cowrie está escutando (a padrão é 2222).

## Conclusão

Com este tutorial, você configurou um honeypot SSH usando o Cowrie e simulou ataques de força bruta com o Medusa. Essa configuração ajuda a entender melhor como invasores tentam acessar sistemas e permite monitorar essas tentativas com segurança. A partir dos logs gerados, você pode analisar essas atividades e melhorar a segurança do seu servidor.
