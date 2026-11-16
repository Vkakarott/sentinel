# Sentinel — Comparativo de Soluções e Alternativas

> **Status da Decisão:** O proprietário decidiu formalmente pelo desenvolvimento de um **App próprio em React Native focado exclusivamente em Android** com distribuição via **Firebase App Distribution**. As ferramentas avaliadas neste documento permanecem catalogadas estritamente como benchmark e referência comparativa histórica. *(Suporte a iOS está fora de escopo por ora).*

---

## 1. Visão Geral das Ferramentas Avaliadas

| Solução | Categoria Primária | Custo | Self-Host? | Plataforma Alvo | Critical Alert / Bypass DND | Ingestão E-mail Nativa? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ntfy.sh** | Push Notification Service | Grátis (hosted limit) / Self-host grátis | Sim (Go / Docker) | Android / iOS | Parcial (Prioridade máxima com som no Android) | Sim (plano pago ou self-host SMTP) |
| **Gotify** | Push Notification Service | Grátis (Open Source) | Sim (Go / Docker) | Apenas Android e Web | Sim no Android (Canais de alta prioridade) | Não (requer bridge externa) |
| **Pushover** | Push Service Proprietário | $5 one-time por plataforma | Não (SaaS fechado) | Android / iOS | Sim (Priority 2 / Emergency fura DND e repete até ack) | Sim (Endereço `@pomail.net` por usuário) |
| **Uptime Kuma** | Health Checker / Uptime Monitor | Grátis (Open Source) | Sim (Node.js / Docker) | Web / PWA (Sem app nativo) | Depende do canal de saída configurado | Não (foco em ping HTTP) |
| **Healthchecks.io**| Cron / Dead Man's Switch | Grátis até 20 checks / Self-host | Sim (Python / Django) | Web / E-mail / Webhooks | Depende do canal de saída configurado | Sim (pings via e-mail) |
| **Apprise** | Multi-channel Dispatcher Gateway | Grátis (Open Source) | Sim (Python / Docker) | N/A (biblioteca emissora) | N/A (depende do destino final) | Não diretamente |
| **Telegram Bot** | Mensageria / Bot API | Grátis | Não (Nuvem Telegram) | Android / iOS / Desktop | Não (Mensagem comum de chat; DND bloqueia) | Não (requer bot intermediário) |
| **App Próprio (Sentinel)** | App Pessoal Dedicado (React Native + FCM)| ~$0 (Free tiers Cloudflare + Firebase) | Sim (Controle total) | **Android** *(iOS fora de escopo por ora)* | **Sim (Android Notification Channels com `bypassDnd: true`)** | **Sim (via Cloudflare Email Routing + Worker)** |

---

## 2. Análise Detalhada por Ferramenta (Benchmark)

### 2.1. ntfy.sh
O [ntfy.sh](https://ntfy.sh) é uma das soluções de código aberto mais populares para envio de notificações push via HTTP `POST`/`PUT`.
- **Como funciona:** Publicação de mensagens em tópicos (`https://ntfy.sh/meu-topico`) e subscrição via app Android.
- **Custos e Limites:**
  - *Instância pública gratuita:* Até 250 mensagens/dia por IP, anexos até 2 MB, tópicos públicos a menos que autenticado.
  - *Self-hosted:* 100% grátis, sem limites artificiais.
- **Prós:** Simples de disparar via cURL, suporta prioridades de 1 a 5, tags, botões de ação na notificação.
- **Por que foi descartado como caminho principal:** O servidor público não possui deduplicação de alertas nem agrupamento dinâmico em janela deslizante; rodar self-hosted exigiria manter uma VPS adicional fora da infraestrutura monitorada, gerando custo desnecessário.
- **Documentação e Fontes:**
  - [ntfy.sh Documentation](https://docs.ntfy.sh/)
  - [ntfy GitHub Repository](https://github.com/binwiederhier/ntfy)

---

### 2.2. Gotify
O [Gotify](https://gotify.net) é um servidor de notificações push auto-hospedado simples e enxuto escrito em Go.
- **Como funciona:** Expõe API REST para criação de aplicações e clientes; app Android mantém WebSocket ou push.
- **Custos e Limites:** 100% open-source e gratuito.
- **Prós:** Leve, canais de prioridade no Android, interface web limpa.
- **Por que foi descartado como caminho principal:** Exige servidor Docker/Linux dedicado 24/7 (dependência de VPS externa), não faz deduplicação inteligente e não possui ingestão nativa de e-mails de alerta.
- **Documentação e Fontes:**
  - [Gotify Documentation](https://gotify.net/docs/)
  - [Gotify GitHub Repository](https://github.com/gotify/server)

---

### 2.3. Pushover
O [Pushover](https://pushover.net) é uma plataforma proprietária voltada para envio de notificações críticas a desenvolvedores e sysadmins.
- **Como funciona:** Disparos via API REST simples ou envio de e-mails para um endereço exclusivo `@pomail.net`.
- **Custos e Limites:** Licença vitalícia de $5 USD (pagamento único por plataforma), 10.000 mensagens/mês gratuitas.
- **Prós:** Suporte excelente a Emergency Priority que fura Não Perturbe no Android; ingestão direta de e-mail; cliente móvel maduro.
- **Por que foi descartado como caminho principal:** Plataforma proprietária de código fechado; não possui motor de deduplicação na origem (loops de erro inundam o aparelho); não fornece tela de status ou painel customizado das APIs do dono.
- **Documentação e Fontes:**
  - [Pushover API Documentation](https://pushover.net/api)
  - [Pushover Pricing & FAQ](https://pushover.net/pricing)

---

### 2.4. Uptime Kuma
O [Uptime Kuma](https://uptime.kuma.pet) é um monitor de uptime auto-hospedado com interface web completa.
- **Como funciona:** Aplicação Node.js com SQLite que executa health checks contínuos e alerta via canais terceiros.
- **Custos e Limites:** 100% gratuito e open-source.
- **Prós:** Interface web intuitiva, status pages públicas, retentativas automáticas antes de declarar queda.
- **Por que foi descartado como caminho principal:** Ponto único de falha se hospedado na mesma VPS; exige manter processo Node.js rodando 24/7 (não roda na edge serverless com custo zero); não é um app móvel nem ingere e-mails de relatórios.
- **Documentação e Fontes:**
  - [Uptime Kuma GitHub](https://github.com/louislam/uptime-kuma)

---

### 2.5. Healthchecks.io
O [Healthchecks.io](https://healthchecks.io) é especializado em monitoramento passivo (*dead man's switch*) de cron jobs e backups.
- **Como funciona:** Seus serviços enviam pings HTTP periódicos; a ausência de ping dispara alerta.
- **Custos e Limites:** Grátis até 20 checks no plano em nuvem.
- **Por que não atende sozinho:** Excelente para checagens passivas de cron, mas não atua como receptor genérico de webhooks de erros de aplicação nem possui app Android próprio com priorização de som.
- **Documentação e Fontes:**
  - [Healthchecks.io Documentation](https://healthchecks.io/docs/)
  - [Healthchecks.io GitHub](https://github.com/healthchecks/healthchecks)

---

### 2.6. Apprise
O [Apprise](https://github.com/caronc/apprise) é uma biblioteca de despacho universal para mais de 90 serviços de notificação.
- **Por que não atende sozinho:** É apenas um despachante de envio, não um app receptor ou motor de agregação.
- **Documentação e Fontes:**
  - [Apprise GitHub Repository](https://github.com/caronc/apprise)

---

### 2.7. Bots de Telegram
- **Como funciona:** Envio de mensagens HTTP POST para a API de bots do Telegram.
- **Por que foi descartado:** Não fura o modo "Não Perturbe" ou silencioso no Android (mensagens são tratadas como chat comum); alertas misturam-se com grupos e canais pessoais; ausência de dashboard de status.
- **Documentação e Fontes:**
  - [Telegram Bot API Documentation](https://core.telegram.org/bots/api)

---

## 3. Conclusão e Diretriz Definitiva

A avaliação comparativa consolidou a decisão do proprietário:
1. **Descarte de soluções prontas:** Nenhuma ferramenta pronta combina custo zero contínuo, motor de deduplicação na edge, ingestão transparente de e-mails, tela personalizada de status de serviços e controle granular de canais Android com bypass de Não Perturbe.
2. **App Próprio em React Native (Android):** Aproveita a experiência do dono com um app mobile existente, viabiliza interface sob medida e garante controle total do ciclo de vida da notificação no Android via Firebase Cloud Messaging e Firebase App Distribution.
3. **Escopo de Plataforma:** O desenvolvimento foca **estritamente em Android**. *(Suporte a iOS, certificados APNs e TestFlight estão fora de escopo por ora).*
