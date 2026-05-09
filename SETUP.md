# SETUP — Login, Sync e Cobrança

Este guia ativa os modos **Cloud** (login + sync) e **PRO** (cobrança). Sem isso, o app já funciona em modo local.

## 1. Supabase (login + sync) — grátis

1. Crie conta em https://supabase.com → New project (região `sa-east-1`).
2. SQL Editor → cole `supabase/schema.sql` → Run.
3. Settings → API → copie **Project URL** e **anon public key**.
4. App → aba **Conta** → **Configurações** → cole → Salvar.
5. **Conta** → Cadastrar com email/senha → confirme via email.

✅ Login ativo. Teste em outro dispositivo: instale o app, faça login — dados sincronizam.

## 2. Cobrança — Stripe

1. Crie conta em https://stripe.com (modo Test).
2. Products → Add product → PRO → preço recorrente mensal.
3. Copie o **Payment link**.
4. Deploy do webhook:
   ```bash
   supabase login
   supabase link --project-ref <seu-ref>
   supabase functions deploy billing-webhook --no-verify-jwt
   ```
5. Stripe → Developers → Webhooks → Add endpoint:
   - URL: `https://<seu-ref>.functions.supabase.co/billing-webhook`
   - Events: `checkout.session.completed`, `customer.subscription.deleted`
6. Copie o **Signing secret** → Supabase → Edge Functions → Secrets:
   - `STRIPE_WEBHOOK_SECRET=<signing secret>`
7. App → **Conta → Configurações** → Checkout URL → cole o Payment link → Salvar.

## 3. Cobrança — Mercado Pago (alternativa BR)

1. https://www.mercadopago.com.br/developers → Checkout Pro → link de pagamento.
2. Notificações → URL: `https://<seu-ref>.functions.supabase.co/billing-webhook?provider=mp`, evento `payment`.
3. Credenciais → copie **Access Token** → Secret: `MP_ACCESS_TOKEN=<token>`.
4. App → **Conta → Configurações** → Checkout URL → cole o link MP.

## 4. Verificação

| Teste | Esperado |
|---|---|
| App sem chaves | Funciona local |
| Cole chaves → cadastro → login | Aba Conta mostra perfil + sync |
| Outro dispositivo → login | Mesmos dados |
| Tentar exportar CSV (free) | Botão `CSV PRO` aparece bloqueado |
| Pagar via checkout (cartão teste) | ~30s depois plano vira PRO |
| DevTools → Offline → reload | App continua |

## 5. Custo

- Supabase: grátis até 500 MB + 50k req/mês.
- Stripe: 3.99% + R$ 0,39 por transação.
- Mercado Pago: ~4.99% à vista (PIX menor).
- GitHub Pages: grátis ilimitado em repos públicos.

## 6. Problemas comuns

- **"Supabase não configurado"** — chaves vazias. Confira em Configurações.
- **RLS error** — schema não rodou completo. Rode `supabase/schema.sql` de novo.
- **Webhook não dispara** — confirme `client_reference_id` (Stripe) ou `external_reference` (MP) na URL. O app injeta o `user_id` automaticamente.
- **App não atualiza** — service worker cacheia. Force reload (Ctrl+Shift+R) ou desinstale+reinstale.
