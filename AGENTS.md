# Sentinel

## Visão Geral do Projeto
O **Sentinel** é um sistema pessoal de monitoramento, agregação de incidentes e entrega de notificações push em tempo real no smartphone Android do proprietário. O objetivo principal é substituir notificações ativas de WhatsApp por push móvel nativo com custo aproximadamente zero, garantindo que alertas críticos toquem mesmo em modo silencioso/Não Perturbe, agrupando tempestades de erro e operando com um monitor externo independente da infraestrutura monitorada.

O sistema é 100% autônomo e independente, composto por um backend serverless e um aplicativo React Native para Android distribuído via Firebase App Distribution.

Para mais detalhes da arquitetura, comparativo e especificações, consulte a pasta `docs/`:
- `docs/visao.md`: Visão do produto, problema, objetivos, não-objetivos e casos de uso.
- `docs/alternativas.md`: Comparativo de soluções existentes (ntfy, Gotify, Pushover, Uptime Kuma, etc.) vs. app próprio.
- `docs/arquitetura.md`: Arquitetura técnica (Cloudflare Edge, FCM, deduplicação, modelo de dados, severidade no Android).
- `docs/roadmap.md`: Fases de evolução, critérios de pronto e matriz de riscos.
- `docs/decisoes.md`: Histórico de decisões tomadas e itens remanescentes.

---

## Diretrizes para Agentes e Desenvolvedores

1. **Idioma da Documentação:** Toda a documentação e discussões de projeto devem ser redigidas em português do Brasil (`pt-BR`).
2. **Idioma do Código e Commits:** Código-fonte, comentários técnicos, nomes de variáveis, classes, schemas e mensagens de commit devem ser estritamente em inglês.
3. **Padrão de Commits:** Utilizar estritamente a convenção **Conventional Commits** (ex.: `feat: ...`, `fix: ...`, `docs: ...`, `chore: ...`).
4. **Sem Co-Author:** Não inclua assinaturas ou trailers de co-autoria (como `Co-authored-by:...`) nas mensagens de commit.
5. **Proibido Envio Remoto:** Nenhuma alteração, branch, tag ou commit deve ser enviada para repositório remoto (`git push` ou criação de `remote`) sem solicitação e autorização prévia e explícita do proprietário.
