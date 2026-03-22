# 4. Geração e Escrita de Texto

---

## 4.1 Por Que a IA Não Precisa de um Editor

O fluxo de trabalho da maioria dos engenheiros humanos para escrever código:
1. Abrir editor (VS Code, Vim, Emacs...)
2. Digitar código
3. Salvar arquivo
4. Fechar editor

Para a IA:
```
"Escrever um programa Python" = gerar algum texto
"Salvar este programa em um arquivo" = escrever texto no disco
```

O processo de geração de código da IA é um **processo de geração de texto**. Então a IA usa as ferramentas de texto do Shell:

- `echo`: exibir linha única de texto
- `printf`: saída formatada
- `heredoc`: exibir texto multi-linha (mais importante!)

---

## 4.2 `echo`: Saída Mais Simples

### Uso Básico

```bash
# Exibir string
echo "Olá, Mundo!"

# Exibir variável
nome="Alice"
echo "Olá, $nome!"

# Exibir múltiplos valores
echo "Hoje é $(date +%Y-%m-%d)"
```

### Armadilhas do `echo`

```bash
# echo adiciona nova linha por padrão
echo -n "Carregando: "  # Sem nova linha
```

### Escrevendo Arquivos com `echo`

```bash
# Sobrescrever
echo "Olá, Mundo!" > arquivo.txt

# Anexar
echo "Segunda linha" >> arquivo.txt
```

**Nota**: Usar `echo` para arquivos multi-linha é doloroso, então a IA quase nunca usa para código. `heredoc` é a estrela.

---

## 4.3 `printf`: Saída Formatada Mais Poderosa

### Comparação com `echo`

```bash
# printf suporta formatação estilo C
printf "Valor: %.2f\n" 3.14159
# Saída: Valor: 3.14

printf "%s\t%s\n" "Nome" "Idade"
```

### Criando Tabelas

```bash
printf "%-15s %10s\n" "Nome" "Preço"
printf "%-15s %10.2f\n" "iPhone" 999.99
printf "%-15s %10.2f\n" "MacBook" 1999.99
```

---

## 4.4 heredoc: A Arma Central da IA para Escrever Código

### O que é heredoc?

heredoc é uma sintaxe especial do Shell para **exibir texto multi-linha literalmente**.

```bash
cat << 'EOF'
Todo este conteúdo
será exibido literalmente
incluindo novas linhas, espaços
EOF
```

### Escrevendo Arquivos (Uso Mais Comum da IA)

```bash
cat > programa.py << 'EOF'
#!/usr/bin/env python3

def hello(name):
    print(f"Olá, {name}!")

if __name__ == "__main__":
    hello("Mundo")
EOF
```

### Por Que Usar `'EOF'` (Aspas Simples)?

```bash
# EOF com aspas simples: não expandir nada
cat << 'EOF'
HOME é: $HOME
Hoje é: $(date)
EOF
# Saída: HOME é: $HOME (não expandido)

# EOF com aspas duplas ou sem aspas: vai expandir
cat << EOF
HOME é: $HOME
EOF
# Saída: HOME é: /home/ai (expandido)
```

**Escolha da IA**: quase sempre usar `'EOF'` (aspas simples). Porque código geralmente não precisa de expansão de variável Shell.

---

## 4.5 IA Escreve Vários Arquivos com heredoc

### Escrever Programa Python

```bash
cat > src/main.py << 'EOF'
#!/usr/bin/env python3
"""Ponto de entrada principal"""

import sys
import os

def main():
    print("Projeto Híbrido Python + C")
    print(f"Diretório de trabalho: {os.getcwd()}")

if __name__ == "__main__":
    main()
EOF
```

### Escrever Script Shell

```bash
cat > scripts/deploy.sh << 'EOF'
#!/bin/bash
set -e

DEPLOY_HOST="server.example.com"

echo "🚀 Fazendo deploy para $DEPLOY_HOST"
rsync -avz --delete ./dist/ "$DEPLOY_HOST:/var/www/app/"
ssh "$DEPLOY_HOST" "systemctl restart myapp"
echo "✅ Deploy completo!"
EOF

chmod +x scripts/deploy.sh
```

### Escrever Arquivo de Configuração

```bash
cat > config.json << 'EOF'
{
    "site_name": "Meu Blog",
    "author": "Anônimo",
    "theme": "minimal"
}
EOF
```

### Escrever Dockerfile

```bash
cat > Dockerfile << 'EOF'
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "-m", "http.server", "8000"]
EOF
```

---

## 4.6 Armadilhas do heredoc e Soluções

### Armadilha 1: Contém Aspas Simples

```bash
# Problema: EOF com aspas simples não permite aspas simples
cat << 'EOF'
Ele disse: 'Olá'.
EOF
# Saída: erro de sintaxe

# Solução: usar EOF com aspas duplas
cat << "EOF"
Ele disse: 'Olá'.
EOF
```

### Armadilha 2: Contém `$` (Mas Não Quer Expansão)

```bash
# Problema: EOF com aspas duplas expande $
cat << "EOF"
Preço é $100
EOF
# Saída: Preço é (vazio)

# Solução: escapar individualmente
cat << "EOF"
Preço é $$100
EOF
# Saída: Preço é $100
```

---

## 4.7 Prática: Construindo um Projeto Completo do Zero

```bash
# 1. Criar estrutura de diretórios
mkdir -p myblog/{src,themes,content}

# 2. Criar arquivo de configuração
cat > myblog/config.json << 'EOF'
{
    "site_name": "Meu Blog",
    "author": "Anônimo",
    "theme": "minimal"
}
EOF

# 3. Criar programa principal Python
cat > myblog/src/blog.py << 'EOF'
#!/usr/bin/env python3
"""Gerador de blog simples"""
import json
from pathlib import Path

def load_config():
    config_path = Path(__file__).parent.parent / "config.json"
    with open(config_path) as f:
        return json.load(f)

if __name__ == "__main__":
    config = load_config()
    print(f"Gerando: {config['site_name']}")
EOF

# 4. Criar script de build
cat > myblog/build.sh << 'EOF'
#!/bin/bash
set -e
echo "🔨 Construindo blog..."
cd "$(dirname "$0")"
python3 src/blog.py
echo "✅ Build completo!"
EOF

chmod +x myblog/build.sh

# 5. Verificar estrutura
find myblog -type f | sort
```

---

## 4.8 Referência Rápida

| Ferramenta | Propósito | Exemplo |
|------------|-----------|---------|
| `echo` | Exibir linha única | `echo "Olá"` |
| `echo -n` | Sem nova linha | `echo -n "Carregando..."` |
| `printf` | Saída formatada | `printf "%s: %d\n" "idade" 25` |
| `>` | Sobrescrever arquivo | `echo "oi" > arquivo.txt` |
| `>>` | Anexar ao arquivo | `echo "oi" >> arquivo.txt` |
| `<< 'EOF'` | heredoc (sem expansão) | preferido para código |
| `<< "EOF"` | heredoc (expandir) | raramente usado |

---

## 4.9 Exercícios

1. Crie uma animação de Carregamento com `echo -n` e loop `for`
2. Crie uma tabela formatada (nome, idade, emprego) com `printf`
3. Escreva um programa Python de 20 linhas com heredoc
4. Escreva um docker-compose.yml com heredoc
