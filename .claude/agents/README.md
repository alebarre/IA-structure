# Fábrica de Software — Kit de Agentes

Time de desenvolvimento multi-agente para Claude Code. Cinco papéis
especializados, cada um com contexto isolado, prompt próprio e permissões de
ferramenta restritas ao seu papel. Stack: **Java/Spring Boot** (back) +
**Angular** (front).

## Instalação

Coloque os arquivos `.md` em `.claude/agents/` na raiz do projeto (ou em
`~/.claude/agents/` para reaproveitar em vários projetos):

```
.claude/agents/
├── po.md          # Product Owner — requisitos, arquitetura, dados
├── dev.md         # Dev full-stack — Spring Boot + Angular (único que coda)
├── qa.md          # QA — smoke tests, gate 100% green
├── security.md    # Segurança estática (SAST) — revisa código parado
└── pentester.md   # Segurança dinâmica (DAST) — ataca a app em staging
```

## Fluxo de trabalho

```
PO  →  Dev  →  QA  →  Security  →  Pentester  →  (CI/CD) produção
│      │       │       │            │
│      │       │       │            └─ ataca staging rodando (OWASP ZAP)
│      │       │       └─ revisa código parado (segredos, CORS, autz)
│      │       └─ mvn verify / ng test — tudo verde ou reprova
│      └─ implementa back+front, escreve testes, corrige achados
└─ especifica em docs/ (fonte da verdade para todos)
```

1. **PO** traduz a visão do produto em `docs/requirements.md`,
   `docs/architecture.md` e `docs/data-model.md`.
2. **Dev** implementa a fatia e escreve os testes.
3. **QA** roda os testes; só passa com 100% verde.
4. **Security** revisa o código estático.
5. **Pentester** ataca a app implantada em staging.
6. Com todos os vereditos APROVADO, o merge/deploy para produção corre no
   **CI/CD** — é lá que mora a trava dura.

## Princípios de projeto

- **Gates mecânicos, não declarativos.** QA, Security e Pentester executam
  comandos reais (testes, scanners) e decidem pelo resultado — não por parecer.
- **Separação de permissões.** Só o **Dev** edita código. PO escreve docs; QA,
  Security e Pentester são read-only sobre a aplicação. Ninguém aprova o próprio
  trabalho.
- **Fonte única da verdade.** Os documentos em `docs/` que o PO mantem são lidos
  por todos os agentes antes de agir.
- **Duas camadas de segurança.** Security (estática/SAST) e Pentester
  (dinâmica/DAST) são complementares, não redundantes.

## Observações importantes

- **A trava de produção é o CI/CD.** Os agentes preparam e validam; a barreira
  que impede merge com build/scan reprovado deve viver no GitHub Actions /
  GitLab CI (fail on High/Critical). O pipeline é o gate final.
- **Ponto de entrada.** Quando houver um produto a definir, a primeira conversa
  é com o **PO**.
