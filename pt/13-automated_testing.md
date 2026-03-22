# 13. Testes Automatizados

---

## 13.1 Por que Usar Shell para Testes

- Execucao rapida de verificacoes simples
- Testar comandos de sistema e scripts
- Testes de integracao em CI/CD
- Testes de regressao de scripts existentes

---

## 13.2 Framework Basico de Testes

```bash
#!/bin/bash
set -euo pipefail

TESTES_EXECUTADOS=0
TESTES_APROVADOS=0
TESTES_REPROVADOS=0

teste() {
    local nome=$1
    local comando=$2
    local esperado=$3
    
    ((TESTES_EXECUTADOS++))
    
    local atual
    atual=$(eval "$comando" 2>/dev/null)
    
    if [[ "$atual" == "$esperado" ]]; then
        echo "✓ $nome"
        ((TESTES_APROVADOS++))
    else
        echo "✗ $nome"
        echo "  Esperado: $esperado"
        echo "  Atual: $atual"
        ((TESTES_REPROVADOS++))
    fi
}

teste "Adicao" "echo \$((1 + 1))" "2"
teste "Strings iguais" "echo ola" "ola"

echo ""
echo "Resultados: $TESTES_APROVADOS/$TESTES_EXECUTADOS aprovado(s)"
[[ $TESTES_REPROVADOS -eq 0 ]] || exit 1
```

---

## 13.3 Testar Arquivo e Diretorio

```bash
#!/bin/bash

teste_arquivo_existe() {
    local arquivo=$1
    [[ -f "$arquivo" ]] || { echo "✗ Arquivo nao encontrado: $arquivo"; return 1; }
}

teste_diretorio_existe() {
    local dir=$1
    [[ -d "$dir" ]] || { echo "✗ Diretorio nao encontrado: $dir"; return 1; }
}

teste_arquivo_contem() {
    local arquivo=$1
    local padrao=$2
    grep -q "$padrao" "$arquivo" || { echo "✗ Arquivo faltando: $padrao"; return 1; }
}
```

---

## 13.4 Testar Saida de Comando

```bash
#!/bin/bash

teste_sucesso() {
    local nome=$1
    shift
    if "$@"; then
        echo "✓ $nome"
    else
        echo "✗ $nome (codigo de saida: $?)"
        return 1
    fi
}

teste_falha() {
    local nome=$1
    shift
    if ! "$@" 2>/dev/null; then
        echo "✓ $nome (falhou corretamente)"
    else
        echo "✗ $nome (deveria falhar mas succeeded)"
        return 1
    fi
}

teste_saida() {
    local nome=$1
    local esperado=$2
    shift 2
    
    local saida=$("$@" 2>/dev/null)
    if [[ "$saida" == *"$esperado"* ]]; then
        echo "✓ $nome"
    else
        echo "✗ $nome"
        return 1
    fi
}
```

---

## 13.5 Script de Teste Amigo de CI

```bash
cat > scripts/test.sh << 'EOF'
#!/bin/bash
set -euo pipefail

VERMELHO='\033[0;31m'
VERDE='\033[0;32m'
SEM_COR='\033[0m'

FALHOU=0

executar_teste() {
    local nome=$1
    local cmd=$2
    
    echo -n "Teste: $nome ... "
    if eval "$cmd" &>/dev/null; then
        echo -e "${VERDE}✓${SEM_COR}"
    else
        echo -e "${VERMELHO}✗${SEM_COR}"
        ((FALHOU++))
    fi
}

executar_teste "Script init existe" "[[ -x scripts/init-projeto.sh ]]"
executar_teste "Script build existe" "[[ -f scripts/build.sh ]]"
executar_teste "Script deploy existe" "[[ -f scripts/deploy.sh ]]"

echo ""
if [[ $FALHOU -eq 0 ]]; then
    echo -e "${VERDE}✅ Todos os testes aprovados${SEM_COR}"
    exit 0
else
    echo -e "${VERMELHO}❌ $FALHOU teste(s) falhou(ram)${SEM_COR}"
    exit 1
fi
EOF

chmod +x scripts/test.sh
```

---

## 13.6 Exercicios

1. Escreva um framework de testes com funcoes `test_eq`, `test_contem`, `test_sucesso`
2. Escreva 5 casos de teste para um dos seus scripts
3. Crie um script de teste amigavel para CI que produza formato TAP
4. Teste um script de processamento em lote para garantir que manuseia todos os arquivos corretamente
