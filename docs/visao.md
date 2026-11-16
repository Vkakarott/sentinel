# Sentinel — Visão do Produto

## 1. Problema

O gerenciamento de sistemas pessoais e projetos em produção exige monitoramento constante e notificação imediata de falhas. Atualmente, o ecossistema do dono enfrenta os seguintes gargalos:

1. **Custo e fragilidade das notificações via WhatsApp:**
   - Mensagens ativas em WhatsApp dependem de APIs pagas (Meta Cloud API com cobrança por conversa/template, Twilio) ou de APIs não oficiais/gateways terceiros (Z-API, Baileys, Evolution) sujeitos a instabilidades de conexão, desconexão de QR Code e risco de bloqueio de número.
   - O WhatsApp não distingue níveis de prioridade do sistema operacional (não permite furar o modo "Não Perturbe" ou "Silencioso" de forma nativa e programática).
2. **Fadiga de alertas (Alert Fatigue) e falta de agrupamento:**
   - Um erro recorrente em loop (ex.: falha de conexão com banco de dados) dispara dezenas ou centenas de mensagens idênticas em poucos minutos, inundando a caixa de mensagens e mascarando outros alertas.
3. **Ponto único de falha no monitoramento ("Quem vigia o vigia?"):**
   - Health checks que rodam dentro da própria infraestrutura/VPS monitorada deixam de alertar caso a VPS inteira sofra queda de rede, congelamento de kernel ou falha de hipervisor.
4. **Dispersão de canais de notificação:**
   - E-mails de alerta enviados por provedores de nuvem (limites de cota, faturamento, falhas de scripts de automação/Excel) ficam perdidos na caixa postal comum entre mensagens promocionais e operacionais.
5. **Falta de histórico contextual unificado:**
   - Notificações avulsas de chat não oferecem tela com linha do tempo de incidentes, status atual dos serviços (se ainda estão fora ou se já normalizaram) nem controle de silenciamento temporário (snooze).

---

## 2. Objetivos

O **Sentinel** é um sistema pessoal de monitoramento, agregação de incidentes e entrega de alertas push em tempo real, composto por um backend leve serverless e um aplicativo React Native exclusivo para o smartphone Android do proprietário (distribuído via Firebase App Distribution).

- **Custo zero (~$0,00):** Operar integralmente sobre camadas gratuitas (*free tiers*) resilientes de computação edge, roteamento de e-mails, banco de dados e entrega de push (ex.: Cloudflare Workers, Cloudflare D1/KV, Firebase Cloud Messaging).
- **Push nativo com severidade real no Android:**
  - Suporte a quatro níveis de severidade: `INFO`, `WARNING`, `ERROR` e `CRITICAL`.
  - Alertas `CRITICAL` utilizam canais de notificação Android com prioridade máxima (`IMPORTANCE_MAX`) e permissão de sobreposição de "Não Perturbe" (`bypassDnd: true`), tocando alarme sonoro mesmo no silencioso.
- **Isolamento de infraestrutura (Monitor externo):**
  - O motor de health check ativo roda 100% fora da infraestrutura monitorada (execução distribuída na edge da Cloudflare), garantindo detecção imediata de quedas globais de servidor.
- **Ingestão multicanal:**
  - **Health check ativo:** Verificação periódica de endpoints HTTP/HTTPS de APIs pessoais.
  - **Webhooks HTTP passivos:** Ingestão autenticada de erros e eventos disparados diretamente pelas aplicações do dono.
  - **E-mails de alerta:** Interceptação e parse de e-mails encaminhados para uma caixa dedicada (alertas de billing, quotas de nuvem, falhas de cron/Excel), transformando-os em eventos estruturados.
- **Deduplicação e agrupamento inteligente:**
  - Reconhecimento de eventos repetidos por meio de *fingerprint* com agregação em janela de tempo (ex.: "Erro de DB disparado 15 vezes nos últimos 5 min"), evitando tempestades de push.
- **Controle de silenciamento (Snooze):**
  - Possibilidade de silenciar alertas de um serviço ou de um erro específico por períodos determinados (15m, 1h, 4h, 24h) diretamente via ação rápida no push do Android ou pelo app.
- **Histórico e status no app:**
  - Aplicativo pessoal em React Native exibindo o histórico de incidentes, status em tempo real dos serviços monitorados e detalhes técnicos do payload.
- **Independência tecnológica absoluta:**
  - O sistema é 100% autônomo e isolado, sem qualquer dependência, vínculo ou integração com serviços corporativos legados.

---

## 3. Não-Objetivos

Para manter o escopo enxuto, de manutenção simples e focado no uso pessoal:

- **Não inclui suporte a iOS neste momento:** O foco exclusivo é o ecossistema Android do proprietário. Suporte a iOS e APNs está fora de escopo por ora.
- **Não é um SaaS multi-tenant comercial:** O sistema não terá suporte a múltiplos clientes externos, gestão de assinaturas, cobrança via Stripe ou isolamento corporativo complexo.
- **Não é um substituto de APM/Tracing distribuído:** Não substituirá ferramentas de telemetria contínua profunda (Prometheus, Grafana, Datadog, Sentry, OpenTelemetry). O foco é notificação e status de incidentes, não profiling de código ou armazenamento de gigabytes de logs brutos.
- **Não gerencia escalas de plantão (on-call rotation):** Não haverá roteamento entre múltiplos analistas ou regras complexas de escalonamento para equipes de suporte. Há uma única persona receptora (o dono).
- **Não implementa bots interativos bidirecionais de chat:** O foco primário é o aplicativo móvel dedicado, evitando o desenvolvimento de fluxos conversacionais complexos para WhatsApp ou Telegram.

---

## 4. Personas

### Dono — Engenheiro de Software, DevOps Solo e Proprietário dos Serviços
- **Perfil:** Desenvolvedor experiente, criador e mantenedor de múltiplos microsserviços, APIs, automações de dados (processamento de planilhas/relatórios) e aplicativos móveis Android (como um app mobile existente em React Native).
- **Necessidades:**
  - Ser acordado no meio da noite **apenas** quando uma API de produção realmente cair ou sofrer erro crítico irreversível.
  - Não receber notificações invasivas para avisos rotineiros ou flutuações transitórias.
  - Silenciar rapidamente alertas de um serviço em manutenção programada com um toque na notificação do Android.
  - Ter visão rápida do que está online ou quebrado sem precisar abrir terminais SSH ou consoles de nuvem.

---

## 5. Casos de Uso

| ID | Caso de Uso | Descrição | Severidade Padrão |
| :--- | :--- | :--- | :--- |
| **CU01** | **Health Check detecta API fora do ar** | O monitor externo (Cloudflare Worker cron) faz requisição GET a `/health` de uma API do dono. Resposta com status `>= 500` ou timeout por 2 checagens consecutivas gera evento de queda. | `CRITICAL` |
| **CU02** | **API recupera conectividade (Auto-resolve)** | Após um incidente de queda, o monitor detecta que a API voltou a responder `200 OK`. Dispara push informativo notificando a recuperação e calcula o tempo total de indisponibilidade (*downtime*). | `INFO` |
| **CU03** | **Webhook de exceção não tratada no backend** | Um backend do dono captura erro grave (ex.: falha de transação bancária) e envia payload JSON autenticado via POST `/v1/events`. O Sentinel deduplica se houver repetição e envia push. | `ERROR` / `CRITICAL` |
| **CU04** | **Alerta originado por e-mail (Quotas / Billing / Relatório)** | Um provedor de nuvem (AWS, GCP, Supabase) ou script cron envia e-mail para `alertas@seudominio.com`. O roteador de e-mail intercepta, extrai título, corpo e remetente, classifica e envia push. | `WARNING` / `ERROR` |
| **CU05** | **Deduplicação de tempestade de erros** | Uma falha dispara 100 erros no intervalo de 3 minutos. O Sentinel processa o primeiro imediatamente, agrupa os 99 subsequentes e atualiza o histórico sem disparar 100 toques sonoros no celular. | Variável (mesma do evento) |
| **CU06** | **Silenciamento temporário (Snooze)** | Durante um deploy ou manutenção planejada, o dono pressiona a ação "Silenciar por 1 hora" na notificação recebida ou no app. Novos alertas daquele serviço durante o período não tocam o celular. | N/A |
| **CU07** | **Consulta e auditoria no App Mobile** | O dono abre o app Sentinel e visualiza a lista de incidentes recentes, detalhes de payloads JSON de erro, status de cada serviço monitorado e eventos em andamento. | N/A |
