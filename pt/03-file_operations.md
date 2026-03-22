# 3. Operações de Arquivos

---

## 3.1 Modelo Mental da IA do Sistema de Arquivos

Antes de mergulhar em cada comando, entenda como a IA vê o sistema de arquivos.

Engenheiros humanos tipicamente veem o sistema de arquivos **visualmente**—como o Explorador de Arquivos do Windows ou o Finder do macOS, entendendo através de ícones e formas de pastas.

A visão da IA é completamente diferente:

```
path          = localização absoluta /home/user/project/src/main.py
relative      = caminhar para baixo a partir da localização atual
nodes         = cada arquivo ou diretório é um "nó"
attributes    = permissões, tamanho, timestamps, dono
type          = arquivo regular(-), diretório(d), link(l), dispositivo(b/c)
```

Quando a IA executa `ls -la`, ela vê:

```
drwxr-xr-x  5 ai  staff  170 Mar 22 10:30 .
drwxr-xr-x  3 ai  staff  102 Mar 22 09:00 ..
-rw-r--r--  1 ai  staff  4096 Mar 22 10:30 .env
-rw-r--r--  1 ai  staff  8192 Mar 22 10:31 README.md
drwxr-xr-x  3 ai  staff   96 Mar 22 10:30 src
drwxr-xr-x  2 ai  staff   64 Mar 22 10:30 tests
-rw-r--r--  1 ai  staff  2048 Mar 22 10:32 package.json
```

A IA pode ler imediatamente disso:
- Quais são diretórios (`d`)
- Quais são arquivos ocultos (começando com `.`)
- Quem tem quais permissões
- Tamanhos de arquivo (determinando se são grandes)
- Última data de modificação

---

## 3.2 `ls`: O Primeiro Comando Mais Usado pela IA

Quase antes de cada operação, a IA executa `ls` para confirmar o estado atual.

### Combinações Comuns de `ls` da IA

```bash
# Lista básica
ls

# Mostrar arquivos ocultos (muito importante!)
ls -a

# Formato longo (informação detalhada)
ls -l

# Formato longo + arquivos ocultos (mais comum)
ls -la

# Ordenar por data de modificação (mais recente primeiro)
ls -lt

# Ordenar por data de modificação (mais antigo primeiro)
ls -ltr

# Tamanhos legíveis por humanos (K, M, G)
ls -lh

# Mostrar apenas diretórios
ls -d */

# Mostrar todos os arquivos recursivamente
ls -R

# Mostrar números de inode (útil para links rígidos)
ls -li
```

### Fluxo de Trabalho Real da IA

```bash
cd ~/project && ls -la

# Resultado:
# drwxr-xr-x  5 ai  staff  170 Mar 22 10:30 .
# drwxr-xr-x  3 ai  staff  102 Mar 22 09:00 ..
# -rw-r--r--  1 ai  staff  4096 Mar 22 10:30 .env
# -rw-r--r--  1 ai  staff  8192 Mar 22 10:31 README.md
# drwxr-xr-x  3 ai  staff   96 Mar 22 10:30 src
# drwxr-xr-x  2 ai  staff   64 Mar 22 10:30 tests
# -rw-r--r--  1 ai  staff  2048 Mar 22 10:32 package.json

# Análise da IA: há um arquivo .env, diretórios src e tests, package.json
# Este é um projeto Node.js
```

---

## 3.3 `cd`: O Diretório que a IA Nunca Esquece

### Hábitos de `cd` da IA

```bash
# Ir para diretório home
cd ~

# Ir para diretório anterior (muito útil!)
cd -

# Ir para diretório pai
cd ..

# Entrar em subdiretório
cd src

# Navegar caminhos profundos (crédito do Tab completion)
cd ~/project/backend/api/v2/routes
```

### Padrão `cd` + `&&` da IA

Este é um dos padrões mais comuns da IA:

```bash
# Primeiro cd, só executa próximo comando após confirmar sucesso
cd ~/project && ls -la
```

### Erros Comuns

```bash
# Erro: não confirmar se diretório existe
cd nonexistent
# Saída: bash: cd: nonexistent: No such file or directory

# Abordagem da IA: verificar primeiro
[ -d "nonexistent" ] && cd nonexistent || echo "Diretório não existe"
```

---

## 3.4 `mkdir`: A Arte de Criar Diretórios

### Uso Básico

```bash
# Criar diretório único
mkdir myproject

# Criar múltiplos diretórios
mkdir src tests docs

# Criar diretórios aninhados (-p é a chave!)
mkdir -p project/src/components project/tests
```

### Por Que a IA Quase Sempre Usa `-p`

A flag `-p` (parents) significa:
1. Se o diretório existe, **nenhum erro**
2. Se o pai não existe, **criar automaticamente**

### Padrão Típico de Criação de Projeto da IA

```bash
# Criar uma estrutura padrão de projeto
mkdir -p project/{src,tests,docs,scripts,config}
```

---

## 3.5 `rm`: A Arte da Delecção

**Aviso**: Este é um dos comandos mais perigosos no Shell.

### Uso Básico

```bash
# Deletar arquivo
rm arquivo.txt

# Deletar diretório (precisa de -r)
rm -r diretorio/

# Deletar diretório e tudo dentro (perigoso!)
rm -rf diretorio/
```

### O Perigo de `rm -rf`

```bash
# Nunca execute isso como root!
# rm -rf /

# Se você acidentalmente adicionar um espaço extra:
rm -rf * 
# (espaço) = rm -rf deleta tudo no diretório atual
```

---

## 3.6 `cp`: Copiando Arquivos e Diretórios

### Uso Básico

```bash
# Copiar arquivo
cp fonte.txt destino.txt

# Copiar diretório (precisa de -r)
cp -r source_directory/ destination_directory/

# Mostrar progresso durante cópia (-v verbose)
cp -v arquivo_grande.iso /backup/

# Modo interativo (perguntar antes de sobrescrever)
cp -i *.py src/
```

### Poder dos Wildcards

```bash
# Copiar todos os arquivos .txt
cp *.txt backup/

# Copiar todos os arquivos de imagem
cp *.{jpg,png,gif,webp} images/
```

---

## 3.7 `mv`: Mover e Renomear

### Uso Básico

```bash
# Mover arquivo
mv arquivo.txt backup/

# Mover e renomear
mv nome_antigo.txt nome_novo.txt

# Renomear em lote
for f in *.txt; do
    mv "$f" "${f%.txt}.md"
done
```

---

## 3.8 Referência Rápida

| Comando | Uso Básico | Flags Comuns | Nota da IA |
|---------|------------|--------------|------------|
| `ls` | `ls [caminho]` | `-l` longo, `-a` oculto, `-h` humano | `ls -la` é sempre bom |
| `cd` | `cd [caminho]` | `-` anterior, `..` pai | `cd xxx &&` é bom hábito |
| `mkdir` | `mkdir [dir]` | `-p` aninhado | quase sempre use `-p` |
| `rm` | `rm [arquivo]` | `-r` recursivo, `-f` forçar | cuidado com `rm -rf /*` |
| `cp` | `cp [src] [dst]` | `-r` diretório, `-i` perguntar, `-p` preservar | use `-i` para segurança |
| `mv` | `mv [src] [dst]` | `-i` perguntar, `-n` não-sobrescrever | é renomear |

---

## 3.9 Exercícios

1. Use `mkdir -p` para criar um diretório aninhado em três níveis, depois confirme com `tree` ou `find`
2. Copie um arquivo grande com `cp -v` e veja a saída
3. Renomeie em lote 10 arquivos `.txt` para `.md` com `mv`
4. Delete um arquivo de teste com `rm -i` para experimentar o prompt
