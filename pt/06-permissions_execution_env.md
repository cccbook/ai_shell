# 6. Permissões, Execução, Ambiente e Configuração

---

## 6.1 `chmod`: A Arte das Permissões

### Noções Básicas de Permissões Linux/Unix

```
drwxr-xr-x  3 ai  staff   96 Mar 22 10:30 .
-rw-r--r--  1 ai  staff  4096 Mar 22 10:30 README.md
-rwxr-xr-x  1 ai  staff   128 Mar 22 10:30 script.sh
```

Os 9 caracteres estão em três grupos:
- `rwx` (dono): ler, escrever, executar
- `r-x` (grupo): ler, executar
- `r--` (outros): apenas ler

### Duas Representações do chmod

**Numérico (octal)**:

```
r = 4, w = 2, x = 1

rwx = 7, r-x = 5, r-- = 4
```

Combinações comuns:
- `777` = rwxrwxrwx (perigoso!)
- `755` = rwxr-xr-x
- `644` = rw-r--r--
- `700` = rwx------
- `600` = rw-------

**Simbólico**:

```bash
chmod u+x script.sh    # Dono adiciona executar
chmod g-w arquivo.txt  # Grupo remove escrever
chmod +x script.sh     # Todos adicionam executar
```

### Uso Comum de chmod da IA

```bash
# Tornar script executável (quase todo script)
chmod +x script.sh

# Tornar diretório navegável
chmod +x ~/projects

# Diretórios com escrita para grupo
chmod -R g+w projeto/
```

---

## 6.2 Executando Scripts Shell

### Métodos de Execução

```bash
# Método 1: Execução por caminho (precisa de permissão de execução)
./script.sh

# Método 2: Usando bash (não precisa de permissão de execução)
bash script.sh

# Método 3: Usando source (executa no shell atual)
source script.sh
```

### Quando Usar Qual?

| Método | Quando Usar | Características |
|--------|-------------|-----------------|
| `./script.sh` | Execução padrão | precisa `chmod +x`, subshell |
| `bash script.sh` | Especificar shell | não precisa permissão de execução |
| `source script.sh` | Definir ambiente | executa no shell atual |

### Diferença Chave: `source` vs `./script`

```bash
# Conteúdo de script.sh: export MINHA_VAR="olá"

# Executar com ./
./script.sh
echo $MINHA_VAR  # Saída: (vazio) ← no subshell, variável se foi

# Executar com source
source script.sh
echo $MINHA_VAR  # Saída: olá ← no shell atual, variável permanece
```

---

## 6.3 `export`: Variáveis de Ambiente

```bash
# Definir variável de ambiente
export NOME="Alice"
export PATH="$PATH:/novo/diretorio"

# Mostrar todas as variáveis de ambiente
export

# Variáveis comuns
echo $HOME      # Diretório home
echo $USER       # Nome de usuário
echo $PATH      # Caminho de busca
echo $PWD       # Diretório atual
```

### Persistindo Variáveis de Ambiente

```bash
# Adicionar ao ~/.bashrc
echo 'export EDITOR=vim' >> ~/.bashrc

# Aplicar alterações
source ~/.bashrc
```

---

## 6.4 `source`: Carregando Arquivos

Equivalente a: **colar** diretamente o conteúdo do arquivo na posição atual e executar.

### Usos Comuns

```bash
# Carregar ambiente virtual
source venv/bin/activate

# Carregar arquivo .env
source .env

# Carregar biblioteca
source ~/scripts/comum.sh
```

### Prático: Arquivos de Configuração Modulares

```bash
# config.sh
export DB_HOST="localhost"
export DB_PORT="5432"

# Usar em outros scripts
source config.sh
psql -h $DB_HOST -p $DB_PORT
```

---

## 6.5 `env`: Gerenciamento de Ambiente

```bash
# Mostrar todas as variáveis de ambiente
env

# Executar com ambiente limpo
env -i HOME=/tmp PATH=/bin sh

# Definir variáveis e executar
env VAR1=valor1 VAR2=valor2 ./meu_programa
```

### Encontrando Comandos

```bash
which python      # Encontrar localização do comando
type cd           # Encontrar builtins do shell
whereis gcc       # Encontrar todos os arquivos relacionados
```

---

## 6.6 `sudo`: Escalando Privilégios

```bash
# Executar como root
sudo rm /var/log/old.log

# Executar como usuário específico
sudo -u postgres psql

# Mostrar o que você pode fazer
sudo -l
```

### Aviso de Perigo

```bash
# Nunca execute isso!
sudo rm -rf /

# Nunca faça isso!
sudo curl http://site-desconhecido.com | sh
```

---

## 6.7 Referência Rápida

| Comando | Propósito | Flags Comuns |
|---------|-----------|--------------|
| `chmod` | Mudar permissões de arquivo | `+x` adicionar exec, `755` octal, `-R` recursivo |
| `chown` | Mudar dono | `usuário:grupo`, `-R` recursivo |
| `./script` | Executar script (precisa x) | - |
| `bash script` | Executar script (não precisa x) | - |
| `source` | Executar no shell atual | - |
| `export` | Definir variáveis de ambiente | `-n` remover |
| `env` | Exibir/gerenciar ambiente | `-i` limpar |
| `sudo` | Executar como root | `-u usuário` especificar usuário |

---

## 6.8 Exercícios

1. Crie um script, defina permissões com `chmod 755`, depois execute-o
2. Execute o mesmo script de variável de ambiente com `source` e `./`, observe a diferença
3. Use `env -i` para criar um ambiente limpo, execute `python --version`
4. Crie um arquivo `.env`, use `source` para carregar suas variáveis
