# Trident Electoral Protocol (TEP)

> *Especificação conceitual e protótipo de referência — versão 0.2*
> *Licença: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) (documentação) · MIT (código)*

---

## O que é o TEP

O TEP é um protocolo eleitoral de código aberto que combina três vetores independentes de registro — urna eletrônica embarcada, blockchain distribuída global e cédula física impressa — exigindo convergência tripla para que qualquer resultado seja considerado válido.

**Nenhum sistema eleitoral conhecido combina simultaneamente os cinco elementos centrais do TEP:**

| Elemento | TEP | Helios | STAR-Vote | Estônia | Alemanha |
|---|:---:|:---:|:---:|:---:|:---:|
| Redundância tripla heterogênea | ✓ | — | — | — | — |
| Hardware unidirecional (TX-only) | ✓ | — | — | — | — |
| UUID atômico com commit-reveal | ✓ | — | — | — | — |
| Regionalização de resultado | ✓ | — | — | — | — |
| Revalidação parcial sem invalidar o todo | ✓ | — | — | — | — |

---

## Motivação

Os sistemas eleitorais modernos falham em garantir simultaneamente auditabilidade humana, eficiência digital e imutabilidade distribuída. A maioria sacrifica uma camada em favor de outra. A falha não é técnica isolada — é **arquitetural**: sistemas são construídos com ponto único de confiança, seja em hardware, software ou instituição.

O TEP parte da premissa de que os três mundos são complementares e que cada camada deve cobrir as fraquezas das demais.

---

## Princípios Fundamentais

1. **A urna é um emissor, nunca um receptor.**
2. **Nenhum resultado é válido sem convergência tripla.**
3. **Todo voto é atômico, rastreável e não repudiável.**
4. **Falhas são localizáveis e corrigíveis sem invalidar o todo.**
5. **Auditabilidade deve ser acessível a humanos comuns e a técnicos.**
6. **O eleitor pode verificar que seu voto foi contabilizado sem revelar em quem votou.**

---

## Arquitetura em uma Página

```
Urna embarcada (ESP8266)
  │
  ├── TX1 (UART, RX fisicamente cortado) ──► Impressora
  │         ├── Cédula: candidato + UUID do voto  ──► Urna física lacrada
  │         └── Recibo: só UUID do voto           ──► Eleitor retém
  │
  └── TX2 (UART para módulo relay) ──► Módulo WiFi (hardware separado)
                                              └── UDP fire-and-forget
                                                      └── Blockchain global
                                                              └── { uuid_voto, hash(uuid_candidato + salt) }

Após encerramento da eleição:
  Revelação pública: { uuid_candidato, salt }
  Qualquer pessoa verifica: hash(uuid_candidato + salt) == entrada_blockchain
```

---

## Por que Open Source

Transparência total é um requisito, não uma escolha. Um sistema eleitoral de código fechado exige confiança cega em seu criador — contradição direta com o objetivo do TEP. Todo componente — firmware da urna, smart contracts, relay, interface de auditoria — é público, auditável e reproduzível por qualquer pessoa com conhecimento técnico.

---

## Origem e TCC

O TEP não nasceu como projeto acadêmico. Nasceu de uma reflexão pessoal sobre espectro político e sobre como melhorar estruturalmente o modelo eleitoral atual — a conclusão foi que confiança em sistemas eleitorais não pode depender de confiança em instituições. Precisa ser técnica e verificável por qualquer pessoa.

O projeto tomou forma própria. Quando chegou a hora de pensar em TCC, a resposta já estava aqui.

O TEP está sendo formalizado como Trabalho de Conclusão de Curso em **Ciência da Computação**. A estrutura acadêmica está em [TCC_ESTRUTURA.md](TCC_ESTRUTURA.md). O desenvolvimento acontece em aberto — o projeto existiria de qualquer forma.

---

## Origem

O TEP nasceu de uma visão política específica: a de que Estado deve ser auditável, eficiente e sem pontos de captura. Confiança em sistemas eleitorais não pode depender de confiança em instituições — precisa ser técnica e verificável por qualquer pessoa. O [manifest.md](manifest.md) documenta o perfil ideológico que originou esse projeto.

---

## Estrutura do Repositório

```
├── README.md                        # este arquivo
├── WHITEPAPER.md                    # especificação técnica completa
├── CHANGELOG.md                     # evolução do protocolo versão a versão
├── TCC_ESTRUTURA.md                 # estrutura acadêmica formal
├── manifest.md                      # visão política e filosófica que originou o TEP
├── architecture/
│   ├── triple-redundancy.md         # os três vetores, UUID, convergência
│   ├── blockchain-layer.md          # commit-reveal, smart contract, nós globais
│   ├── regional-model.md            # regionalização e revalidação parcial
│   └── anomaly-detection.md         # sistema passivo de score por zona
├── threat-model/
│   └── attack-vectors.md            # STRIDE, vetores residuais, custo de ataque
└── references/
    └── cenci-beck-2022-blockchain-sistema-eleitoral-br.pdf  # referência base
```

---

## Contexto Brasileiro

O TSE possui o projeto "Eleições do Futuro" com testes de blockchain desde 2020 (Cenci; Beck, 2022). As propostas existentes focam em **e-voting e voto a distância** — etapa da votação em si. O TEP foca na camada que essas propostas deixam em aberto: **apuração, registro e auditabilidade da contagem**, com redundância física que nenhuma solução digital isolada oferece.

---

## Histórico

| Data | Evento |
|---|---|
| 2026-05-07 | Concepção inicial — discussão informal documentada |
| 2026-05-08 | Especificação v0.2 — arquitetura dual TX, commit-reveal, cerimônia de seed |

---

## Licença

Documentação: [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/)
Código-fonte: MIT License

Contribuições, críticas técnicas e estudos derivados são bem-vindos com atribuição de autoria.
