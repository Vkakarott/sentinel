# Sentinel — Registro de Decisões e Perguntas em Aberto

Este documento registra as decisões de arquitetura e produto do **Sentinel**, diferenciando as deliberações já **tomadas e consolidadas pelo proprietário** daquelas que permanecem **em aberto**.

---

## 1. Decisões Tomadas

### D01 — Abordagem do Cliente: App Próprio em React Native
- **Status:** **TOMADA (2026-11-16)**
- **Decisão:** Desenvolver um aplicativo móvel próprio em **React Native**.
- **Justificativa:** Ferramentas prontas (Pushover, ntfy.sh) foram descartadas como caminho principal devido à falta de deduplicação nativa na origem, ausência de visualização customizada de status das APIs e restrições de controle sobre agrupamento. O know-how prévio do dono com um app mobile existente viabiliza a entrega rápida de uma experiência 100% personalizada e com custo zero. As ferramentas prontas permanecem catalogadas apenas como referência comparativa em [docs/alternativas.md](alternativas.md).

---

### D02 — Plataforma Alvo: Exclusivamente Android
- **Status:** **TOMADA (2026-11-16)**
- **Decisão:** Foco **exclusivo no ecossistema Android**. *(Suporte a Apple iOS, certificados APNs e Time-Sensitive/Critical Alerts estão formalmente fora de escopo por ora).*
- **Justificativa:** O smartphone de uso pessoal do dono opera com Android, permitindo controle irrestrito sobre `NotificationChannels`, personalização de sons, vibrações dedicadas e sobreposição direta do modo Não Perturbe (`setBypassDnd: true`), sem necessidade de autorizações prévias da Apple ou pagamento de anuidades.

---

### D03 — Estratégia de Distribuição: Firebase App Distribution
- **Status:** **TOMADA (2026-11-16)**
- **Decisão:** Distribuir o aplicativo Android via **Firebase App Distribution** (não utilizar sideload manual avulso de APK por cabo/chat nem publicação em lojas públicas).
- **Justificativa:** O Firebase App Distribution é gratuito, integra perfeitamente com a stack Firebase já utilizada para o FCM e oferece o aplicativo **App Tester** para o dono, garantindo atualizações instantâneas no celular em um toque, controle de versões de release e histórico limpo.

---

### D04 — Relação com Serviços Corporativos Existentes
- **Status:** **TOMADA (2026-11-16)**
- **Decisão:** Sem dependência de serviços corporativos existentes.
- **Justificativa:** Garantir isolamento arquitetural total, evitando que qualquer instabilidade, dependência de schema ou manutenção de serviços corporativos afete os alertas pessoais do proprietário.

---

## 2. Decisões em Aberto (Aguardando Aprovação)

### D05 — Hospedagem do Backend & Monitor Externo (Cloudflare Workers vs. VPS Dedicada)
- **Status:** **EM ABERTO**
- **Contexto:** O monitor e a ingestão de eventos precisam obrigatoriamente rodar **fora** da VPS monitorada para evitar o ponto único de falha (*single point of failure*).
- **Opções:**
  - *Opção A (Recomendada):* **Cloudflare Workers + Cron Triggers + Email Routing.**
    - Execução serverless na edge global, custo $0 no Free Tier (100k req/dia), cron triggers de 1 min nativos e ingestão de e-mails em domínio próprio sem subir servidor SMTP.
  - *Opção B:* **Segunda VPS em outro provedor (ex.: Hetzner ou Oracle Cloud Free Tier).**
    - Ambiente Linux tradicional, mas exige manutenção contínua de SO, patches de segurança e custos adicionais (ou risco de cancelamento em tiers Always Free).
- **Recomendação do Agente:** **Opção A (Cloudflare Workers).** Elimina manutenção de infraestrutura e resolve o health check com custo zero garantido.

---

### D06 — Persistência do Histórico de Incidentes (Cloudflare D1 + KV vs. Postgres Externo)
- **Status:** **EM ABERTO**
- **Contexto:** É necessário persistir a lista de serviços cadastrados, status de uptime, histórico de eventos e janelas de deduplicação/snooze.
- **Opções:**
  - *Opção A (Recomendada):* **Cloudflare D1 (SQLite na Edge) + Cloudflare KV.**
    - Totalmente integrado aos Workers via bindings nativos (`env.DB` e `env.KV`), 5M de leituras e 100k escritas gratuitas/dia no D1; o KV gerencia janelas de dedup e mute que expiram sozinhas via TTL sem necessidade de cron de expurgo.
  - *Opção B:* **Supabase ou Neon (Postgres externo gratuito).**
    - Banco Postgres completo, mas introduz latência de rede externa e risco de suspensão (*pausing*) de projetos inativos em tiers gratuitos.
- **Recomendação do Agente:** **Opção A (Cloudflare D1 + KV).** Mantém todo o ecossistema backend unificado sob a mesma conta da Cloudflare com performance ideal e zero atrito de rede.
