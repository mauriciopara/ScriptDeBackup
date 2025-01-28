# ScriptDeBackup

#!/bin/bash

# Verifica se o script está sendo executado como root
if [ "$(id -u)" -ne 0 ]; then
    echo "Este script deve ser executado como root." >&2
    exit 1
fi

# Verifica se foram fornecidos dois parâmetros
if [ "$#" -ne 2 ]; then
    echo "Uso: $0 <pasta_origem> <pasta_destino>" >&2
    exit 1
fi

PASTA_ORIGEM="$1"
PASTA_DESTINO="$2"

# Verifica se a pasta de origem existe
if [ ! -d "$PASTA_ORIGEM" ]; then
    echo "Erro: A pasta de origem não existe." >&2
    exit 1
fi

# Criação das pastas de backup se não existirem
mkdir -p "$PASTA_DESTINO"

# Mantendo a propriedade dos arquivos
rsync -a --delete "$PASTA_ORIGEM"/ "$PASTA_DESTINO/backup.1"/

# Rotaciona os backups antigos
rm -rf "$PASTA_DESTINO/backup.3"
mv "$PASTA_DESTINO/backup.2" "$PASTA_DESTINO/backup.3" 2>/dev/null
mv "$PASTA_DESTINO/backup.1" "$PASTA_DESTINO/backup.2" 2>/dev/null

# Cria o backup mais recente
cp -a "$PASTA_ORIGEM" "$PASTA_DESTINO/backup.1"

# Configuração do cron job para execução automática às 4h da manhã
CRON_JOB="0 4 * * * /bin/bash $PWD/$0 $PASTA_ORIGEM $PASTA_DESTINO"
(crontab -l 2>/dev/null; echo "$CRON_JOB") | crontab -

echo "Backup concluído e agendado para ser executado diariamente às 4h da manhã."
