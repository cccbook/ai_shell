# 5. Processamento de Texto

---

## 5.1 Filosofia de Processamento de Texto da IA

No mundo da IA, **tudo é texto**.

- Código é texto
- Arquivos de configuração são texto
- Logs são texto
- JSON, HTML, Markdown são todos texto

Então comandos de processamento de texto são o núcleo do toolkit da IA.

Quando engenheiros humanos encontram problemas: "Preciso de uma ferramenta para lidar com isso..."
Quando a IA encontra problemas: "Isso pode ser resolvido com `grep | sed | awk` em uma linha."

---

## 5.2 `cat`: A Arte de Ler Arquivos

### Uso Básico

```bash
# Exibir conteúdo do arquivo
cat arquivo.txt

# Combinar arquivos
cat parte1.txt parte2.txt > inteiro.txt

# Mostrar números de linha
cat -n script.sh
```

### Propósito Real: Combinar e Criar

```bash
cat << 'EOF' > novo_arquivo.txt
Conteúdo do arquivo
Pode escrever muitas linhas
EOF
```

---

## 5.3 `head` e `tail`: Vendo Apenas o Que Você Precisa

### `head`: Olhar o Começo

```bash
# Primeiras 10 linhas (padrão)
head arquivo.txt

# Primeiras 5 linhas
head -n 5 arquivo.txt

# Primeiros 100 bytes
head -c 100 arquivo.txt
```

### `tail`: Olhar o Fim

```bash
# Últimas 10 linhas (padrão)
tail arquivo.txt

# Últimas 5 linhas
tail -n 5 arquivo.txt

# Seguir arquivo em tempo real (mais comum!)
tail -f /var/log/syslog

# Seguir e filtrar
tail -f app.log | grep --line-buffered ERROR
```

### Visualizar Intervalo Específico de Linhas

```bash
# Visualizar linhas 100-150
tail -n +100 arquivo.txt | head -n 50
```

---

## 5.4 `wc`: Ferramenta de Contagem

```bash
# Contar linhas
wc -l arquivo.txt

# Contar linhas de múltiplos arquivos
wc -l *.py

# Contar arquivos no diretório
ls | wc -l
```

---

## 5.5 `grep`: Rei da Busca de Texto

### Uso Básico

```bash
# Buscar linhas com "erro"
grep "erro" log.txt

# Ignorar maiúsculas/minúsculas
grep -i "erro" log.txt

# Mostrar números de linha
grep -n "erro" log.txt

# Mostrar apenas nomes de arquivos
grep -l "TODO" *.md

# Inverter (linhas não correspondentes)
grep -v "debug" log.txt

# Corresponder palavra inteira
grep -w "erro" log.txt
```

### Expressões Regulares

```bash
# Corresponder início
grep "^Erro" log.txt

# Corresponder fim
grep "concluído.$" log.txt

# Qualquer caractere
grep "e.or" log.txt

# Intervalo
grep -E "[0-9]{3}-" log.txt
```

### Técnicas Avançadas

```bash
# Busca recursiva
grep -r "TODO" src/

# Apenas extensão específica
grep -r "TODO" --include="*.py" src/

# Mostrar contexto de linhas
grep -B 2 -A 2 "ERRO" log.txt

# Múltiplas condições (OU)
grep -E "erro|aviso|fatal" log.txt
```

---

## 5.6 `sed`: Ferramenta de Substituição de Texto

### Substituição Básica

```bash
# Substituir primeira correspondência
sed 's/antigo/novo/' arquivo.txt

# Substituir todas as correspondências
sed 's/antigo/novo/g' arquivo.txt

# Substituição no local
sed -i 's/antigo/novo/g' arquivo.txt

# Fazer backup e substituir
sed -i.bak 's/antigo/novo/g' arquivo.txt
```

### Deletar Linhas

```bash
# Deletar linhas vazias
sed '/^$/d' arquivo.txt

# Deletar linhas de comentário
sed '/^#/d' arquivo.txt

# Deletar espaços em branco à direita
sed 's/[[:space:]]*$//' arquivo.txt
```

### Exemplos Práticos

```bash
# Renomear extensão em lote (.txt → .md)
for f in *.txt; do
    mv "$f" "$(sed 's/.txt$/.md/' <<< "$f")"
done

# Remover finais de linha Windows
sed -i 's/\r$//' arquivo.txt
```

---

## 5.7 `awk`: Canivete Suíço do Processamento de Texto

### Conceito Básico

`awk` processa texto linha por linha, automaticamente dividindo em campos ($1, $2, $3...), executando ações especificadas para cada linha.

### Uso Básico

```bash
# Divisão padrão por espaços em branco
awk '{print $1}' arquivo.txt

# Especificar delimitador
awk -F: '{print $1}' /etc/passwd

# Mostrar múltiplos campos
awk -F: '{print $1, $3, $7}' /etc/passwd
```

### Processamento Condicional

```bash
# Processar apenas linhas correspondentes
awk -F: '$3 > 1000 {print $1}' /etc/passwd

# BEGIN e END
awk 'BEGIN {print "Início"} {print} END {print "Fim"}' arquivo.txt
```

### Exemplos Práticos

```bash
# Somar uma coluna CSV
awk -F, '{soma += $3} END {print soma}' dados.csv

# Encontrar máximo
awk 'NR==1 {max=$3} $3>max {max=$3} END {print max}' dados.csv

# Saída formatada
awk '{printf "%-20s %10.2f\n", $1, $2}' dados.txt
```

---

## 5.8 Prática: Combinando Todas as Ferramentas

### Cenário: Analisar Logs do Servidor

```bash
# 1. Encontrar mensagens de erro
grep -i "erro" access.log

# 2. Contar erros
grep -ci "erro" access.log

# 3. Encontrar erros mais comuns
grep "erro" access.log | awk '{print $NF}' | sort | uniq -c | sort -rn | head

# 4. Contar requisições por hora
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c
```

### Cenário: Modificar Código em Lote

```bash
# Mudar "print" para "logger.info" em todos os arquivos .py
find . -name "*.py" -exec sed -i 's/print(/logger.info(/g' {} +

# Mudar var para const
find . -name "*.js" -exec sed -i 's/\bvar\b/const/g' {} +
```

---

## 5.9 Referência Rápida

| Comando | Propósito | Flags Comuns |
|---------|-----------|--------------|
| `cat` | Exibir/combinar arquivos | `-n` números de linha |
| `head` | Ver início do arquivo | `-n` linhas, `-c` bytes |
| `tail` | Ver fim do arquivo | `-n` linhas, `-f` seguir |
| `wc` | Contagem | `-l` linhas, `-w` palavras, `-c` bytes |
| `grep` | Busca de texto | `-i` ignorar, `-n` número linha, `-r` recursivo, `-c` contar |
| `sed` | Substituição de texto | `s/antigo/novo/g`, `-i` no local |
| `awk` | Processamento de campos | `-F` delimitador, `{print}` ação |

---

## 5.10 Exercícios

1. Use `head` e `tail` para ver linhas 100-120
2. Use `grep` para encontrar todos os usuários com `/bin/bash` em /etc/passwd
3. Use `sed` para substituir todos `\r\n` com `\n`
4. Use `awk` para calcular máximo, mínimo e média de um arquivo numérico
