# JM Transportes — Financeiro

App de controle financeiro instalável (PWA) com login, sincronização em nuvem e plano PRO.

- **Funciona offline** (service worker + localStorage)
- **Instalável** no celular (Android/iOS) e desktop como app nativo
- **Login opcional** via Supabase — sem login, app roda 100% local
- **Plano PRO** com exportação CSV, cobrança via Stripe ou Mercado Pago

## Setup rápido

1. Habilite GitHub Pages: **Settings → Pages → Source: GitHub Actions**
2. Faça push em `main` → o workflow publica automaticamente
3. Acesse `https://<seu-usuario>.github.io/<repo>/`
4. Para login + sync + cobrança: siga [SETUP.md](./SETUP.md)

## Estrutura

```
index.html                     SPA principal (React inline)
sw.js                          Service worker offline-first
manifest.webmanifest           Manifest PWA
icons/                         Ícones
lib/cloud.js                   Bridge Supabase + sync + billing
supabase/schema.sql            Schema SQL (rodar 1 vez)
supabase/functions/            Edge Functions (webhook)
.github/workflows/pages.yml    Deploy automático
```

## Modos

| Modo | Precisa | Funciona |
|---|---|---|
| **Local** | Nada | Tudo local |
| **Cloud** | Supabase configurado em Conta→Config | Login + sync entre dispositivos |
| **PRO** | Cloud + link checkout | Exportação CSV + extras |

## Roadmap

- [x] PWA instalável + offline
- [x] Login Supabase + sync
- [x] Plano PRO com gates
- [x] Webhook Stripe / Mercado Pago
- [ ] Importação OFX
- [ ] Múltiplas empresas
- [ ] Relatório anual
- [ ] Open Finance (Belvo / Pluggy)
