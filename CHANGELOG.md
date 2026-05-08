# Changelog

Todas as mudanças relevantes do projeto são documentadas aqui.
Formato: `## [versão] — YYYY-MM-DD` com seções `Adicionado`, `Alterado`, `Removido`.

---

## [0.2] — 2026-05-08

### Adicionado
- **Arquitetura dual TX na urna:** dois canais UART independentes (TX1 → impressora, TX2 → módulo relay), ambos com pino RX eletricamente cortado. Módulo relay como hardware fisicamente separado da urna.
- **Impressão em duas vias:** Via 1 (cédula completa com candidato + UUID) depositada na urna física; Via 2 (somente UUID) retida pelo eleitor para auditoria pessoal na blockchain.
- **Commit-reveal para UUID do candidato:** UUID do candidato nunca exposto durante a eleição — blockchain registra `hash(uuid_candidato ‖ salt)`, revelado apenas após encerramento. Salt por candidato impede brute-force.
- **Geração de seed com entropia física verificável:** modelo análogo ao LavaRand da Cloudflare. Cerimônia pública documentada com Multi-Party Computation (MPC) — nenhuma parte individual conhece a seed completa antes da combinação.
- **Verificação individual pelo eleitor (recorded-as-cast):** eleitor pode confirmar via recibo que seu UUID está na blockchain sem revelar em quem votou.
- **Protótipo de referência definido:** stack tecnológico — C++ (Arduino/ESP8266) para firmware, TypeScript (Node.js + ethers.js) para relay e backend, Solidity para smart contract, Ethereum Sepolia testnet como blockchain.
- **Estrutura de repositório:** pastas `architecture/`, `threat-model/`, `references/` criadas.
- **Documentos:** `WHITEPAPER.md`, `architecture/triple-redundancy.md`, `architecture/blockchain-layer.md`, `architecture/regional-model.md`, `architecture/anomaly-detection.md`, `threat-model/attack-vectors.md`.
- **Referência:** Cenci; Beck (2022) — *Nova Tecnologia para o Sistema Eleitoral Brasileiro: Blockchain e Transparência* adicionada e contextualizada no README.
- **Declaração open source:** projeto declarado aberto (CC BY 4.0 para documentação, MIT para código) com justificativa técnica — transparência total é requisito arquitetural.
- **TCC:** projeto definido como base para Trabalho de Conclusão de Curso em Ciência da Computação. `TCC_ESTRUTURA.md` criado com estrutura acadêmica formal.

### Alterado
- **README** reescrito para v0.2: tabela comparativa com sistemas existentes, diagrama de arquitetura completo, seção de contexto brasileiro, seção TCC.
- **Distinção formal entre níveis de TX-only:** hardware (pino RX removido) vs. aplicação (UDP fire-and-forget) — documentada com threat model de cada nível.
- **Modelo regional** reposicionado como decisão de segurança arquitetural, não apenas modelo político.

### Removido
- Nada removido nesta versão.

---

## [0.1] — 2026-05-07

### Adicionado
- Concepção inicial do protocolo TEP em discussão informal documentada.
- Três vetores: urna eletrônica, blockchain, cédula impressa física.
- Princípio hardware TX-only (data diode físico).
- UUID atômico: `hash(seed_eleição + session_id + timestamp + entropia_local)`.
- Modelo regional: resultado por regiões ganhas, não volume absoluto.
- Revalidação parcial: inconsistência em uma região não invalida o todo.
- Detecção de anomalias: sistema passivo de score por zona.
- Tabela comparativa com Helios, STAR-Vote, Estônia, Alemanha.
- `README.md` inicial com especificação conceitual v0.1.
