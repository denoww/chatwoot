# Chatwoot

## Development

### 1. Variáveis de ambiente
O arquivo `.env.dev` já está no repositório com as configurações locais. Ajuste o `FRONTEND_URL` se necessário (ex: ngrok).

### 2. Subir
```bash
docker compose -f docker-compose.dev.yml --env-file .env.dev up -d
```

### 3. Primeiro uso — criar banco e rodar migrations
```bash
docker compose -f docker-compose.dev.yml exec chatwoot_web bundle exec rails db:chatwoot_prepare
```

### 4. Logs
```bash
docker compose -f docker-compose.dev.yml logs -f
```

### 5. Derrubar
```bash
docker compose -f docker-compose.dev.yml down
```

---

## Produção

O deploy é feito automaticamente via GitHub Actions a cada push na `main`, ou manualmente em **Actions → Deploy → Run workflow**.

### Fluxo do deploy
1. Upload do `docker-compose.yml` para o S3
2. Na EC2: baixa o compose e o `.env` do S3, roda `docker compose up -d`
3. Registra a EC2 no load balancer

### Configuração inicial

**1. Secrets no GitHub** (`Settings → Secrets and variables → Actions`):
- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

**2. `.env` de produção no S3** — preencha com os dados reais e faça upload:
```bash
aws s3 cp .env s3://sc-environments/chatwoot/.env --acl private
```

Variáveis obrigatórias no `.env`:
```
DATABASE_URL=postgresql://USER:PASSWORD@RDS_ENDPOINT:5432/chatwoot_production
REDIS_URL=redis://ELASTICACHE_ENDPOINT:6379
RAILS_ENV=production
SECRET_KEY_BASE=...
FRONTEND_URL=https://chatwoot.seudominio.com.br
```

**3. EC2** — a instância precisa ter:
- Tag `projetos=chatwoot`
- SSM Agent rodando
- IAM role com permissão de leitura no S3 (`sc-environments`) e acesso ao SSM

**4. Primeiro deploy — criar banco**

Após o primeiro deploy, rode na EC2:
```bash
cd /opt/chatwoot
docker compose exec chatwoot_web bundle exec rails db:chatwoot_prepare
```
