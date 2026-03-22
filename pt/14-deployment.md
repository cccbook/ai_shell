# 14. Scripts de Implantacao

---

## 14.1 Elementos Essenciais de um Bom Script de Implantacao

Um bom script de implantacao deve:
1. Suportar modo de teste (dry-run)
2. Ter saida de log clara
3. Fazer rollback em caso de falha
4. Lidar com interrupcoes
5. Ter controle de versao

---

## 14.2 Script de Implantacao Simples

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

HOST_IMPLANTACAO="servidor.exemplo.com"
CAMINHO_IMPLANTACAO="/var/www/meuapp"
USUARIO_IMPLANTACAO="deploy"

VERMELHO='\033[0;31m'
VERDE='\033[0;32m'
AMARELO='\033[1;33m'
SEM_COR='\033[0m'

log() { echo -e "${VERDE}[INFO]${SEM_COR} $@"; }
warn() { echo -e "${AMARELO}[AVISO]${SEM_COR} $@"; }
erro() { echo -e "${VERMELHO}[ERRO]${SEM_COR} $@" >&2; }

verificar() {
    command -v rsync &>/dev/null || { erro "rsync necessario"; exit 1; }
    if ! ssh -o ConnectTimeout=5 "$USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO" "exit 0" 2>/dev/null; then
        erro "Nao e possivel conectar a $HOST_IMPLANTACAO"
        exit 1
    fi
    log "Pre-verificacoes aprovadas"
}

backup() {
    log "Criando backup..."
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    ssh "$USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO" "cp -r $CAMINHO_IMPLANTACAO $CAMINHO_IMPLANTACAO.backup.$TIMESTAMP"
    log "Backup feito: $CAMINHO_IMPLANTACAO.backup.$TIMESTAMP"
}

implantar() {
    log "Iniciando implantacao..."
    rsync -avz --delete \
        --exclude='.git' \
        --exclude='node_modules' \
        --exclude='.env' \
        ./ "$USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO:$CAMINHO_IMPLANTACAO/"
    log "Arquivos sincronizados"
}

reiniciar() {
    log "Reiniciando servico..."
    ssh "$USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO" "systemctl restart meuapp"
    sleep 2
    
    if ssh "$USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO" "systemctl is-active meuapp" | grep -q "active"; then
        log "Servico iniciado com sucesso"
    else
        erro "Servico falhou ao iniciar"
        exit 1
    fi
}

principal() {
    log "🚀 Implantacao iniciando"
    log "Alvo: $USUARIO_IMPLANTACAO@$HOST_IMPLANTACAO:$CAMINHO_IMPLANTACAO"
    
    verificar
    backup
    implantar
    reiniciar
    
    log "✅ Implantacao concluida!"
}

principal
EOF

chmod +x deploy.sh
```

---

## 14.3 Suportar Dry-run

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

DRY_RUN=false
HOST_IMPLANTACAO="servidor.exemplo.com"

while [[ $# -gt 0 ]]; do
    case "$1" in
        -n|--dry-run)
            DRY_RUN=true
            shift
            ;;
        *)
            shift
            ;;
    esac
done

log() { echo "[INFO] $@"; }

implantar() {
    if $DRY_RUN; then
        log "[DRY-RUN] Sincronizaria arquivos para $HOST_IMPLANTACAO"
        rsync -avz --dry-run ./ "$HOST_IMPLANTACAO:/tmp/teste/"
    else
        log "Iniciando implantacao..."
        rsync -avz ./ "$HOST_IMPLANTACAO:/var/www/app/"
    fi
}

implantar
EOF
```

---

## 14.4 Implantacao Multi-Ambiente

```bash
cat > deploy.sh << 'EOF'
#!/bin/bash
set -euo pipefail

AMBIENTE="${1:-staging}"

case "$AMBIENTE" in
    staging)
        HOST="staging.exemplo.com"
        CAMINHO="/var/www/staging"
        ;;
    production)
        HOST="prod.exemplo.com"
        CAMINHO="/var/www/production"
        CONFIRMAR=true
        ;;
    *)
        echo "Ambiente desconhecido: $AMBIENTE"
        exit 1
        ;;
esac

log() { echo "🚀 [$AMBIENTE] $@"; }

if [[ "$CONFIRMAR" == "true" ]]; then
    echo "Sobre implantacao em producao: $HOST"
    read -p "Continuar? (s/n) " -n 1 -r
    echo
    [[ $REPLY =~ ^[Ss]$ ]] || exit 0
fi

log "Iniciando implantacao para $HOST"

rsync -avz --delete \
    --exclude='.env' \
    ./ "$HOST:$CAMINHO/"

ssh "$HOST" "cd $CAMINHO && npm install --production"

log "Implantacao concluida"
EOF
```

---

## 14.5 Exercicios

1. Escreva um script de implantacao que suporte opcao `--dry-run`
2. Adicione funcionalidade de backup ao script de implantacao
3. Escreva um script de implantacao multi-ambiente (dev/staging/production)
4. Crie um script de rollback que liste e permita escolher qual versao restaurar
