# 7. Controle de Fluxo e Loops Condicionais

---

## 7.1 Combinando Comandos: A Essência do Shell

Comandos únicos têm capacidade limitada. Só **combinando** eles você pode accomplishing tarefas complexas.

A IA é poderosa em grande parte porque domina essas combinações:

```bash
cat access.log | grep "ERRO" | sort | uniq -c | sort -rn | head -10
```

Isso significa: "De access.log, encontrar erros, contar ocorrências, mostrar top 10"

---

## 7.2 `|` (Pipe): A Arte do Fluxo de Dados

O pipe transforma a **saída** do comando anterior na **entrada** do próximo.

```bash
# Ordenar conteúdo do arquivo
cat nao_ordenado.txt | sort

# Encontrar comandos mais comuns
history | awk '{print $2}' | sort | uniq -c | sort -rn | head -10

# Extrair IPs do log e contar
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -5
```

### Enviando stderr para Pipe

```bash
# Enviar stderr para pipe
comando1 2>&1 | comando2

# Ou Bash 4+
comando1 |& comando2
```

---

## 7.3 `&&`: Executar Próximo Apenas Se Bem-Sucedido

**Apenas se `comando1` succeed (código de saída = 0) `comando2` será executado.**

```bash
# Criar diretório então entrar nele
mkdir -p projeto && cd projeto

# Compilar então executar
gcc -o programa fonte.c && ./programa

# Baixar então extrair
curl -L -o archive.tar.gz http://exemplo.com/arquivo && tar -xzf archive.tar.gz
```

---

## 7.4 `||`: Executar Próximo Apenas Se Falhar

**Apenas se `comando1` falhar (código de saída ≠ 0) `comando2` será executado.**

```bash
# Criar arquivo se não existir
[ -f config.txt ] || echo "Config faltando" > config.txt

# Tentar um caminho, fallback para outro
cd /opt/projeto || cd /home/user/projeto

# Garantir sucesso mesmo em falha (comum em makefiles)
cp arquivo.txt arquivo.txt.bak || true
```

### Combinando `&&` e `||`

```bash
# Expressão condicional
[ -f config ] && echo "Encontrado" || echo "Não encontrado"

# Equivalente a:
if [ -f config ]; then
    echo "Encontrado"
else
    echo "Não encontrado"
fi
```

---

## 7.5 `;`: Executar Independentemente

```bash
# Todos os três executam
mkdir /tmp/test ; cd /tmp/test ; pwd
```

---

## 7.6 `$()`: Substituição de Comando

**Executar comando, substituir `$()` com sua saída.**

```bash
# Uso básico
echo "Hoje é $(date +%Y-%m-%d)"
# Saída: Hoje é 2026-03-22

# Em variáveis
ARQUIVOS=$(ls *.txt)

# Obter nome do diretório
DIR=$(dirname /caminho/para/arquivo.txt)
BASE=$(basename /caminho/para/arquivo.txt)

# Calcular
echo "Resultado é $((10 + 5))"
# Saída: Resultado é 15
```

### vs Backticks

```bash
# Ambos são equivalentes
echo "Hoje é $(date +%Y)"
echo "Hoje é `date +%Y`"

# Mas $() é melhor porque pode aninhar
echo $(echo $(echo aninhado))
```

---

## 7.7 `[[ ]]` e `[ ]`: Testes Condicionais

### Testes de Arquivo

```bash
[[ -f arquivo.txt ]]      # Arquivo regular existe
[[ -d diretorio ]]        # Diretório existe
[[ -e caminho ]]          # Qualquer tipo existe
[[ -L link ]]             # Link simbólico existe
[[ -r arquivo ]]          # Legível
[[ -w arquivo ]]          # Gravável
[[ -x arquivo ]]          # Executável
[[ arquivo1 -nt arquivo2 ]]  # arquivo1 é mais recente que arquivo2
```

### Testes de String

```bash
[[ -z "$str" ]]          # String é vazia
[[ -n "$str" ]]          # String não é vazia
[[ "$str" == "valor" ]]  # Igual
[[ "$str" =~ padrão ]]    # Corresponde regex
```

### Testes Numéricos

```bash
[[ $num -eq 10 ]]        # Igual
[[ $num -ne 10 ]]        # Não igual
[[ $num -gt 10 ]]       # Maior que
[[ $num -lt 10 ]]       # Menor que
```

---

## 7.8 `if`: Declarações Condicionais

```bash
if [[ condição ]]; then
    # fazer algo
elif [[ condição2 ]]; then
    # fazer outra coisa
else
    # fallback
fi
```

### Exemplo Completo

```bash
#!/bin/bash

ARQUIVO="config.yaml"

if [[ ! -f "$ARQUIVO" ]]; then
    echo "Erro: $ARQUIVO não existe"
    exit 1
fi

if [[ -r "$ARQUIVO" ]]; then
    echo "Arquivo é legível"
else
    echo "Arquivo não é legível"
fi
```

---

## 7.9 `for`: Loops

### Sintaxe Básica

```bash
for variável in lista; do
    # usar $variável
done
```

### Padrões Comuns da IA

```bash
# Processar todos os arquivos .txt
for arquivo in *.txt; do
    echo "Processando $arquivo"
done

# Intervalo numérico
for i in {1..10}; do
    echo "Iteração $i"
done

# Array
for cor in vermelho verde azul; do
    echo $cor
done

# Loop estilo C (Bash 3+)
for ((i=0; i<10; i++)); do
    echo $i
done
```

---

## 7.10 `while`: Loops Condicionais

```bash
# Ler linhas
while IFS= read -r linha; do
    echo "Lido: $linha"
done < arquivo.txt

# Loop de contagem
contador=0
while [[ $contador -lt 10 ]]; do
    echo $contador
    ((contador++))
done
```

---

## 7.11 `case`: Correspondência de Padrão

```bash
case $AÇÃO in
    start)
        echo "Iniciando serviço..."
        ;;
    stop)
        echo "Parando serviço..."
        ;;
    restart)
        $0 stop
        $0 start
        ;;
    *)
        echo "Uso: $0 {start|stop|restart}"
        exit 1
        ;;
esac
```

### Padrões com Wildcards

```bash
case "$nome_arquivo" in
    *.txt)
        echo "Arquivo de texto"
        ;;
    *.jpg|*.png|*.gif)
        echo "Arquivo de imagem"
        ;;
    *)
        echo "Tipo desconhecido"
        ;;
esac
```

---

## 7.12 Referência Rápida

| Símbolo | Nome | Descrição |
|---------|------|-------------|
| `\|` | Pipe | Passar saída para próxima entrada |
| `&&` | E | Executar próximo apenas se anterior succeed |
| `\|\|` | OU | Executar próximo apenas se anterior falha |
| `;` | Ponto e vírgula | Executar independentemente |
| `$()` | Substituição de comando | Executar, substituir com saída |
| `[[ ]]` | Teste condicional | sintaxe de teste recomendada |
| `if` | Condicional | ramificação baseada em condição |
| `for` | Loop de contagem | iterar através de lista |
| `while` | Loop condicional | repetir enquanto condição for verdadeira |
| `case` | Correspondência de padrão | ramificação multi-caminho |

---

## 7.13 Exercícios

1. Use `|` para combinar `ls`, `grep`, `wc` para contar arquivos `.log`
2. Use `&&` para garantir que `cd` succeed antes de continuar
3. Use loop `for` para criar 10 diretórios (dir1 a dir10)
4. Use `while read` para ler e exibir /etc/hosts
5. Escreva uma calculadora simples com `case` (somar, subtrair, multiplicar, dividir)
