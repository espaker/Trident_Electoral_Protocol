# TEP Whitepaper — Especificação Técnica

> Trident Electoral Protocol · versão 0.2 · 2026-05-08

---

## 1. Problema

Os sistemas eleitorais modernos falham em garantir simultaneamente:

- **Auditabilidade humana:** qualquer cidadão sem conhecimento técnico pode verificar o resultado
- **Eficiência digital:** apuração rápida, consistente e imune a erro humano em escala
- **Imutabilidade distribuída:** resultado não pode ser alterado por nenhum agente centralizado

A falha é arquitetural: sistemas são desenhados com ponto único de confiança. A urna eletrônica brasileira, por exemplo, é auditável e segura dentro de seus próprios termos, mas centraliza confiança no TSE e no software proprietário. O TSE reconhece isso e busca soluções complementares via blockchain (projeto "Eleições do Futuro", 2020), mas as propostas existentes focam em e-voting — não na camada de apuração e registro distribuído.

O TEP resolve a camada que nenhum sistema atual resolve: **redundância tripla heterogênea com convergência verificável publicamente.**

---

## 2. Princípios de Design

1. A urna é um emissor, nunca um receptor
2. Nenhum resultado é válido sem convergência tripla
3. Todo voto é atômico, rastreável e não repudiável
4. Falhas são localizáveis e corrigíveis sem invalidar o todo
5. Auditabilidade acessível a humanos comuns e técnicos
6. O eleitor pode verificar que seu voto foi contabilizado sem revelar em quem votou

---

## 3. Arquitetura de Hardware da Urna

A urna opera com dois canais de transmissão fisicamente independentes. O pino RX é eletricamente cortado em ambos — injeção de dados é fisicamente impossível, não apenas bloqueada por software.

```
Microcontrolador embarcado (ESP8266)
  │
  ├── TX1 — UART GPIO1
  │     pino RX fisicamente cortado/removido
  │     └── Impressora térmica
  │           ├── Cédula (Via 1): candidato + UUID do voto
  │           │     └── Eleitor deposita na urna física lacrada
  │           └── Recibo (Via 2): apenas UUID do voto
  │                 └── Eleitor retém para auditoria pessoal
  │
  └── TX2 — UART para módulo relay dedicado
        pino RX fisicamente cortado/removido
        └── Módulo ESP8266 separado (hardware distinto)
              └── WiFi UDP — fire-and-forget, sem recv()
                      └── Nós da blockchain global
```

### Distinção formal entre níveis de TX-only

| Nível | Mecanismo | Classe de ataque eliminada |
|---|---|---|
| TX-only de hardware (TX1, TX2) | Pino RX eletricamente removido | Injeção física direta |
| TX-only de aplicação (UDP) | Fire-and-forget, código nunca chama recv() | Injeção remota via rede |

A urna não tem conhecimento do que acontece após TX2. O módulo relay é hardware descartável — substituível sem comprometer nenhum registro já realizado.

---

## 4. Geração de Seed e Cerimônia de Abertura

A seed é a raiz de toda a cadeia de confiança criptográfica da eleição.

### Requisitos

- **Entropia física verificável:** múltiplas fontes imprevisíveis e publicamente observáveis — ruído atmosférico de estações distribuídas geograficamente, dados sísmicos, câmeras de locais públicos transmitidas ao vivo. Modelo análogo ao LavaRand da Cloudflare.
- **Multi-Party Computation (MPC):** nenhum participante individual conhece a seed completa. Cada parte contribui com entropia independente; a seed final é derivada da combinação — elimina ponto único de comprometimento.
- **Cerimônia pública documentada:** realizada em ato aberto, transmitida ao vivo, com observadores independentes. O processo de geração é auditável por qualquer pessoa.
- **Registro imutável pré-eleição:** seed publicada na blockchain antes do início da votação. Qualquer alteração posterior seria detectável.

A seed nunca é reutilizada entre eleições.

---

## 5. UUID Atômico do Voto

```
UUID_voto = SHA-256(seed_eleição ‖ session_id ‖ timestamp ‖ entropia_local)
```

- `seed_eleição`: gerada na cerimônia pública, nunca reutilizada
- `session_id`: identificador único da urna naquela sessão eleitoral
- `timestamp`: momento exato do voto em unix time com precisão de milissegundos
- `entropia_local`: saída de `esp_random()` — TRNG do hardware do ESP8266

O UUID do voto é o fio condutor entre os três vetores. Permite comparação atômica voto a voto sem revelar identidade do eleitor.

---

## 6. Commit-Reveal para UUID do Candidato

O UUID de cada candidato é derivado da seed da eleição e permanece opaco durante todo o processo eleitoral.

### Durante a eleição — blockchain armazena

```
{
  uuid_voto:        "a3f7...",
  candidate_commit: SHA-256(uuid_candidato ‖ salt)
}
```

### Após encerramento — revelação pública simultânea

```
{
  uuid_candidato: "b9c2...",
  salt:           "e1d4..."
}
```

### Verificação por qualquer observador

```
SHA-256(uuid_candidato ‖ salt) == candidate_commit  →  voto válido e contabilizado
```

O `salt` impede brute-force durante a eleição: mesmo conhecendo todos os candidatos possíveis, reverter o commit é computacionalmente inviável sem o salt. O resultado só pode ser determinado após revelação simultânea de todos os UUIDs de candidatos.

---

## 7. Verificação Individual pelo Eleitor

O recibo (Via 2) contém apenas o `uuid_voto`. Com ele, o eleitor pode:

1. **Durante ou após a eleição:** consultar a blockchain publicamente e confirmar que o UUID está registrado (prova de que o voto foi *recorded-as-cast*)
2. **Após a revelação:** confirmar que o UUID foi contabilizado para o candidato escolhido (prova de *counted-as-recorded*)

O eleitor **não consegue provar a terceiros em quem votou** de forma trivial. O UUID do voto não revela o candidato sem a chave de revelação — e mesmo após a revelação, estabelecer a ligação exige conhecimento técnico que impede coerção simples ("mostre seu recibo, eu confirmo seu voto"). Isso preserva o sigilo e protege contra coerção.

---

## 8. Os Três Vetores

### Vetor 1 — Urna Eletrônica

- Registro digital local do voto
- Única fonte de geração de UUID — nunca recebe dados externos
- Hardware embarcado com TX físico apenas, pino RX eletricamente removido

### Vetor 2 — Blockchain Global

- Registro imutável por consenso distribuído
- Nós geograficamente distribuídos — sem autoridade central
- Append-only: nós de urna nunca fazem request de volta
- Dados de urna chegam via UDP unidirecional do módulo relay

### Vetor 3 — Cédula Impressa Física

- Impressa via TX1 durante o ato do voto
- Cédula completa (Via 1): candidato + UUID — depositada pelo eleitor
- Auditável por qualquer pessoa sem equipamento digital
- Âncora física independente de qualquer sistema digital

---

## 9. Modelo Regional

O resultado nacional **nunca é determinado por volume absoluto de votos.** A eleição é decidida por regiões ganhas.

**Consequências:**
- Fraude em massa em uma região tem impacto limitado no resultado geral
- Revalidação é cirúrgica — inconsistência em uma região não invalida o país
- Custo de ataque escala com o número de regiões comprometidas simultaneamente

Ver [architecture/regional-model.md](architecture/regional-model.md) para protocolo completo.

---

## 10. Validação e Convergência

| Vetor | Tolerância | Justificativa |
|---|---|---|
| Urna Eletrônica | 0% | Sistema digital — qualquer divergência é anomalia grave |
| Blockchain | 0% | Imutável por design — divergência indica comprometimento |
| Cédula Impressa | Margem pequena (a definir) | Extravio físico orgânico é possível |

Divergência acima da margem dispara revalidação regional automática.

---

## 11. Stack do Protótipo de Referência

| Camada | Tecnologia |
|---|---|
| Firmware urna | C++ (Arduino/ESP8266) |
| Módulo relay | C++ (ESP8266 dedicado) |
| Blockchain | Ethereum Sepolia testnet |
| Smart contract | Solidity — append-only registry com commit-reveal |
| Backend/relay server | TypeScript (Node.js) + ethers.js |
| Interface de auditoria pública | TypeScript (React) |

---

## 12. Análise de Segurança

Ver [threat-model/attack-vectors.md](threat-model/attack-vectors.md) para análise STRIDE completa e vetores de ataque residuais.

---

## 13. Pontos em Aberto

### Críticos
- **Identidade do eleitor:** unicidade do voto sem comprometer anonimato — ZK-proof de título eleitoral?
- **Threshold exato** para tolerância de extravio de cédulas físicas — requer dados históricos
- **Granularidade regional** — município, zona eleitoral, estado?
- **Protocolo MPC formal** para cerimônia de geração de seed

### Importantes
- Formato do QR Code — dados mínimos + resistência a leitura em massa não autorizada
- Modelo de consenso para rede blockchain própria — PoA com nós auditados?
- Cerimônia formal de encerramento da urna
- Receipt-freeness: análise aprofundada de coerção via recibo de UUID

### Desejáveis
- Algoritmo formal de score de anomalia estatística por zona
- Compatibilidade ou migração a partir de sistemas legados brasileiros
- Formalização matemática do protocolo em notação criptográfica

---

## Referências

- ADIDA, B. et al. *Helios: Web-based Open-Audit Voting.* USENIX Security, 2008.
- BELL, S. et al. *STAR-Vote: A Secure, Transparent, Auditable, and Reliable Voting System.* USENIX EVT/WOTE, 2013.
- BENALOH, J. *Simple Verifiable Elections.* USENIX EVT, 2006.
- BEN-SASSON, E. et al. *Succinct Non-Interactive Zero Knowledge for a von Neumann Architecture.* USENIX Security, 2014.
- BYRES, E. *The Myths and Facts behind Cyber Security Risks for Industrial Control Systems.* VDE Congress, 2004.
- CENCI, D. R.; BECK, C. *Nova Tecnologia para o Sistema Eleitoral Brasileiro: Blockchain e Transparência.* Est. Eleit., Brasília, v. 16, n. 1, p. 285-334, jan./jul. 2022.
- CLOUDFLARE. *LavaRand in Production: The Nitty-Gritty Technical Details.* 2017.
- HEIBERG, S.; WILLEMSON, J. *Verifiable Internet Voting in Estonia.* 2014.
- NAKAMOTO, S. *Bitcoin: A Peer-to-Peer Electronic Cash System.* 2008.
- NIST SP 800-82 — *Guide to Industrial Control Systems Security.*
- NIST SP 800-90A — *Recommendation for Random Number Generation Using Deterministic Random Bit Generators.*
- RYAN, P. et al. *Prêt à Voter: a Voter-Verifiable Voting System.* IEEE TIFS, 2009.
