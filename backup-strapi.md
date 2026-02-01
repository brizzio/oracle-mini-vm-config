Fazer o backup do Strapi com SQLite é muito simples, porque toda a inteligência do seu banco de dados está em um único arquivo dentro do seu servidor. Diferente do MySQL, que exige comandos complexos de exportação, com o SQLite basta copiar o arquivo.

Aqui estão as três formas de garantir que você não perca nada:

1. Backup Manual via Terminal (O mais rápido)
Como mapeamos um volume na sua Stack do Docker, os arquivos do Strapi estão salvos em uma pasta dentro do seu servidor. Você pode criar uma cópia de segurança com este comando:

``` bash
# Entra na pasta onde os dados do Strapi estão (ajuste o caminho se necessário)
# Se você usou o caminho relativo ./app na Stack:
cp ~/app/.tmp/data.db ~/app/.tmp/data_backup_$(date +%F).db
```

Isso criará um arquivo como data_backup_2026-02-01.db.

2. Backup pelo Portainer (Interface Visual)
Você pode baixar os arquivos diretamente para o seu computador pessoal:

No Portainer, vá em Containers.

Clique no ícone de Console (>_) do container do Strapi.

Clique em Connect (usando /bin/sh ou bash).

Navegue até a pasta do banco: cd .tmp.

Dica: O Portainer não tem um botão direto de "Download" dentro do console, mas você pode usar a aba Volumes no menu lateral, localizar o volume criado para o Strapi e ver os arquivos lá.

3. Automatizando com o Script de Backup
Para não esquecer, você pode adicionar uma tarefa ao seu servidor que faz o backup toda noite.

Crie o script de backup: nano backup_strapi.sh

Cole este conteúdo:

```bash
#!/bin/bash
DESTINO="/home/ubuntu/backups"
ORIGEM="/home/ubuntu/app/.tmp/data.db" # Verifique se este é o caminho da sua Stack
mkdir -p $DESTINO
cp $ORIGEM $DESTINO/strapi_db_$(date +%Y%m%d_%H%M).db
# Apaga backups com mais de 7 dias para não encher o disco de 1GB
find $DESTINO -type f -mtime +7 -name "*.db" -delete
```


Agende no Linux (Crontab):

Digite 

``` bash
crontab -e.
```

Adicione esta linha ao final (roda todo dia às 03:00 da manhã): 0 3 * * * /bin/bash /home/ubuntu/backup_strapi.sh

Importante: Onde estão as imagens?
O banco de dados (.db) guarda os textos e configurações. Mas as fotos e uploads ficam na pasta public/uploads. Se você quiser um backup completo, você deve copiar a pasta app inteira:

Bash
tar -czvf backup_completo_strapi.tar.gz /home/ubuntu/app
