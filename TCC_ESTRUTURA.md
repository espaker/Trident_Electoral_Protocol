# TCC — Estrutura Formal
## Trident Electoral Protocol: Um Protocolo de Redundância Tripla para Eleições Auditáveis

> Documento de estruturação acadêmica — versão 0.2

---

## Título Provisório

**"TEP — Trident Electoral Protocol: Arquitetura de Redundância Tripla para Sistemas Eleitorais Auditáveis, Distribuídos e Resistentes a Captura"**

---

## Problema Central

Os sistemas eleitorais modernos falham em garantir simultaneamente:

1. **Auditabilidade humana** — qualquer cidadão sem conhecimento técnico pode verificar
2. **Eficiência digital** — apuração rápida, consistente e imune a erro humano em escala
3. **Imutabilidade distribuída** — resultado não pode ser alterado por nenhum agente centralizado

A hipótese central é que a falha não é técnica isolada — é **arquitetural**: sistemas são desenhados com ponto único de confiança, seja em hardware, software ou instituição.

---

## Objetivo

Propor uma arquitetura de protocolo eleitoral que elimine pontos únicos de falha e de confiança, através de redundância tripla com vetores heterogêneos e convergência verificável.

---

## Estrutura Acadêmica

### 1. Introdução

- 1.1 Contextualização — crise de confiança em sistemas eleitorais contemporâneos
- 1.2 Motivação — por que redundância tripla e não aperfeiçoamento incremental
- 1.3 Problema de pesquisa — "É possível um sistema eleitoral auditável sem ponto único de confiança?"
- 1.4 Objetivos gerais e específicos
- 1.5 Justificativa
- 1.6 Estrutura do trabalho

---

### 2. Referencial Teórico

#### 2.1 Teoria Democrática e Confiança Institucional
- Legitimidade eleitoral como fundamento da democracia representativa
- Confiança procedimental vs. confiança em resultados
- Vulnerabilidade de sistemas centralizados à captura institucional
- Referências: Dahl (Poliarquia), Przeworski (Democracia e Mercado), Lijphart

#### 2.2 Criptografia Aplicada a Sistemas Eleitorais
- Funções hash e propriedades de imutabilidade
- UUID e unicidade de registros atômicos
- Commit-reveal: compromisso criptográfico com revelação diferida
- Zero-Knowledge Proofs — verificação sem revelação de identidade
- Blockchain: consenso distribuído, append-only, resistência a revisão
- Referências: Nakamoto (Bitcoin whitepaper), Ben-Sasson (zk-SNARKs)

#### 2.3 Geração de Entropia Verificável
- Entropia verdadeira vs. pseudoaleatória em contextos de alta confiança
- LavaRand (Cloudflare): fontes físicas imprevisíveis como base de aleatoriedade
- NIST Randomness Beacon: beacon público de entropia verificável
- Cerimônias públicas de geração de seed: auditabilidade do próprio processo aleatório
- Referências: Cloudflare LavaRand whitepaper, NIST SP 800-90

#### 2.4 Hardware de Segurança
- Princípio unidirecional: data diodes em infraestrutura crítica
- Air gap e suas limitações práticas
- Hardware TX-only de hardware vs. TX-only de aplicação: distinção formal e threat model de cada nível
- Referências: padrões NIST, literatura de ICS/SCADA security

#### 2.5 Verificabilidade em Sistemas Eleitorais — Propriedades Formais
- Cast-as-intended: o voto registrado corresponde à intenção do eleitor
- Recorded-as-cast: o voto registrado foi corretamente armazenado
- Counted-as-recorded: o voto armazenado foi corretamente contabilizado
- O recibo de UUID como mecanismo de verificação recorded-as-cast pelo próprio eleitor
- Referências: Ryan et al. (Prêt à Voter), Benaloh (Simple Verifiable Elections)

#### 2.6 Sistemas Eleitorais Existentes — Estado da Arte
- Helios Voting — auditabilidade criptográfica, sem camada física
- STAR-Vote (Travis County, EUA) — urna + impresso + auditoria, sem blockchain
- i-Voting da Estônia — voto digital verificável, sem redundância física
- Sistema alemão — paper trail auditável, sem componente digital
- Análise comparativa: o que cada um resolve e o que deixa em aberto

---

### 3. Metodologia

- 3.1 Abordagem — pesquisa aplicada de design de sistemas
- 3.2 Modelagem de ameaças (threat modeling) — STRIDE aplicado a sistemas eleitorais
- 3.3 Critérios de avaliação do protocolo proposto
  - Resistência a ataque remoto
  - Auditabilidade por cidadão comum
  - Localização e contenção de falhas
  - Custo de comprometimento do resultado
  - Verificabilidade individual pelo eleitor sem comprometer sigilo
- 3.4 Análise comparativa com sistemas existentes
- 3.5 Protótipo de referência — escopo e limitações

---

### 4. O Protocolo TEP

#### 4.1 Princípios de Design
- A urna é um emissor, nunca um receptor
- Nenhum resultado é válido sem convergência tripla
- Todo voto é atômico, rastreável e não repudiável
- Falhas são localizáveis sem invalidar o todo
- Auditabilidade acessível a humanos comuns e técnicos
- O eleitor pode verificar que seu voto foi contabilizado sem revelar em quem votou

#### 4.2 Arquitetura de Hardware da Urna

A urna opera com dois canais de transmissão fisicamente separados e independentes, ambos unidirecionais por hardware:

```
Urna (microcontrolador embarcado)
  │
  ├── TX1 (UART, pino RX fisicamente cortado)
  │     └── Impressora térmica
  │           ├── Via 1 — Cédula completa: candidato + UUID do voto
  │           │     └── Depositada pelo eleitor na urna física lacrada
  │           └── Via 2 — Recibo do eleitor: apenas UUID do voto
  │                 └── Eleitor retém para auditoria pessoal na blockchain
  │
  └── TX2 (UART para módulo relay, pino RX fisicamente cortado)
        └── Módulo de transmissão dedicado (hardware separado)
              └── UDP fire-and-forget (sem ACK) → Nós da blockchain
```

**Distinção formal entre os dois níveis de TX-only:**
- **TX-only de hardware (TX1 e TX2):** pino RX eletricamente desconectado — injeção de dados fisicamente impossível
- **TX-only de aplicação (UDP):** protocolo fire-and-forget sem recv() — injeção remota sem superfície de recepção no código

A separação entre urna e módulo relay garante que a urna nunca tem conhecimento do que acontece após TX2 — o relay é descartável e substituível sem comprometer a integridade do registro local.

#### 4.3 Geração de Seed e Cerimônia de Abertura

A seed da eleição é a raiz de toda a cadeia de confiança criptográfica. Sua geração exige:

- **Entropia física verificável:** múltiplas fontes imprevisíveis e publicamente observáveis (ex: ruído atmosférico de estações distribuídas, dados sísmicos, câmeras de locais públicos) — modelo análogo ao LavaRand da Cloudflare
- **Cerimônia pública documentada:** geração realizada em ato público com observadores independentes, filmada e transmitida ao vivo
- **Contribuição de múltiplas partes (MPC):** nenhum participante individual conhece a seed completa antes da combinação — elimina ponto único de comprometimento
- **Registro imutável:** seed final publicada na blockchain antes do início da eleição

A seed nunca é reutilizada entre eleições.

#### 4.4 UUID Atômico do Voto

```
UUID_voto = hash(seed_eleição + session_id + timestamp + entropia_local)
```

- `seed_eleição`: gerada na cerimônia pública, nunca reutilizada
- `session_id`: identificador único da urna naquela sessão
- `timestamp`: momento exato do voto
- `entropia_local`: dado imprevisível gerado localmente pelo hardware da urna

O UUID do voto é o fio condutor entre os três vetores — permite comparação atômica voto a voto sem revelar a identidade do eleitor.

#### 4.5 Commit-Reveal para UUID do Candidato

O UUID de cada candidato é derivado da seed da eleição e **nunca exposto durante o processo eleitoral:**

```
Durante a eleição — blockchain armazena:
  { uuid_voto, hash(uuid_candidato + salt) }

Após encerramento — publicado:
  { uuid_candidato, salt }

Verificação por qualquer observador:
  hash(uuid_candidato + salt) == entrada_blockchain  →  voto válido e contabilizado
```

O `salt` impede brute-force durante a eleição — mesmo conhecendo os candidatos possíveis, é computacionalmente inviável reverter o hash sem o salt. O resultado da eleição só pode ser determinado após a revelação simultânea de todos os UUIDs de candidatos.

#### 4.6 Verificação Individual pelo Eleitor

O recibo (Via 2) contém apenas o `uuid_voto`. Com ele, o eleitor pode:

1. Consultar a blockchain publicamente: confirmar que o UUID está registrado
2. Após encerramento: confirmar que o UUID foi corretamente contabilizado para o candidato escolhido

O eleitor **não consegue provar a terceiros em quem votou** — o UUID do voto não está vinculado ao UUID do candidato de forma legível antes da revelação, e mesmo após a revelação a ligação é técnica, não trivialmente apresentável como "prova de voto" a observadores externos. Isso preserva o sigilo do voto e protege contra coerção.

#### 4.7 Arquitetura dos Três Vetores

- **Vetor 1 — Urna Eletrônica:** registro digital local, fonte única de geração, hardware embarcado
- **Vetor 2 — Blockchain distribuída:** registro imutável por consenso, nós globais, append-only
- **Vetor 3 — Cédula Impressa Física (Via 1):** auditável sem equipamento digital, depositada pelo eleitor

#### 4.8 Modelo Regional

- Resultado nacional determinado por regiões ganhas, não volume absoluto de votos
- Inconsistência em uma região não invalida o país inteiro
- Custo de ataque proporcional ao número de regiões comprometidas simultaneamente
- Revalidação parcial e cirúrgica: apenas a região afetada repete o processo
- Organizadores substitutos selecionados aleatoriamente

#### 4.9 Validação e Convergência

| Vetor | Tolerância | Justificativa |
|---|---|---|
| Urna Eletrônica | 0% | Sistema digital — divergência é anomalia grave |
| Blockchain | 0% | Imutável por design — divergência indica comprometimento |
| Cédula Impressa | Margem pequena (a definir) | Extravio físico orgânico é possível |

Divergência acima da margem dispara revalidação regional automática.

#### 4.10 Detecção de Anomalias

Sistema passivo separado do fluxo de validação:
- Score de confiabilidade por zona eleitoral
- Padrões estatísticos de divergência por região
- Correlações temporais anômalas nos registros da blockchain
- Não bloqueia resultados — gera relatório público auditável pós-eleição

---

### 5. Protótipo de Referência

#### 5.1 Stack Tecnológico

| Camada | Tecnologia | Justificativa |
|---|---|---|
| Firmware da urna | C++ (Arduino/ESP8266) | Único ambiente viável para embarcado com UART TX-only |
| Módulo relay | C++ (ESP8266 dedicado) | Hardware separado, UDP fire-and-forget |
| Blockchain | Ethereum Sepolia testnet | Distribuição global real, gratuita, ethers.js nativo |
| Smart contract | Solidity | Registro append-only verificável publicamente |
| Backend/relay server | TypeScript (Node.js) | Stack principal, integração com ethers.js/viem |
| Interface de auditoria pública | TypeScript (React) | Consulta aberta à blockchain por qualquer cidadão |

#### 5.2 Escopo do Protótipo

**Inclui:**
- Urna simulada em ESP8266: geração de UUID, HMAC-SHA256, TX1 serial, TX2 UDP
- Smart contract append-only na Sepolia com commit-reveal
- Script de revelação de UUIDs pós-eleição e contagem verificável
- Interface pública de auditoria: consulta por uuid_voto

**Fora do escopo:**
- Impressora física real (substituída por log serial)
- Cerimônia de geração de seed (substituída por seed mockada documentada)
- Identidade do eleitor / unicidade do voto (anotado como trabalho futuro)

---

### 6. Análise de Segurança

#### 6.1 Modelagem de Ameaças — STRIDE

| Ameaça | Mitigação no TEP |
|---|---|
| Spoofing | UUID único + HMAC — duplicação computacionalmente inviável |
| Tampering | Blockchain append-only + cédula física como âncora independente |
| Repudiation | UUID não repudiável, voto atômico registrado em três vetores |
| Information Disclosure | Commit-reveal: candidatos ocultos até encerramento; salt impede brute-force |
| Denial of Service | Falha localizada por região, não invalida o todo |
| Elevation of Privilege | Hardware TX-only elimina superfície de ataque remoto na urna |

#### 6.2 Vetores de Ataque Residuais
- Comprometimento físico da urna antes da eleição (pré-implantação de firmware malicioso)
- Captura de maioria dos nós da blockchain antes da eleição
- Falsificação em massa de cédulas físicas (exige acesso à impressora e ao formato exato)
- Comprometimento da cerimônia de geração de seed

#### 6.3 Análise de Custo de Ataque
- Custo comparativo vs. sistemas tradicionais — ataque exige comprometimento simultâneo de múltiplas camadas heterogêneas
- Detecção inevitável: convergência tripla expõe qualquer divergência

---

### 7. Pontos em Aberto — Agenda de Pesquisa Futura

#### Críticos (necessários para especificação completa)
- **Identidade do eleitor:** como garantir unicidade do voto sem comprometer anonimato? ZK-proof de título eleitoral?
- **Threshold exato de tolerância** para cédulas físicas — requer dados históricos de extravio orgânico
- **Definição formal das regiões** — granularidade ótima (município? zona eleitoral?)
- **Protocolo MPC para geração da seed** — especificação formal da cerimônia

#### Importantes
- Formato do QR Code — dados mínimos, resistência a leitura em massa não autorizada
- Modelo de consenso da blockchain em rede própria — PoA com nós auditados?
- Cerimônia de encerramento da urna — procedimentos físicos e digitais formais
- Receipt-freeness: aprofundar análise de coerção via recibo de UUID

#### Desejáveis
- Algoritmo formal de score de anomalia estatística por zona
- Compatibilidade ou migração gradual a partir de sistemas legados
- Formalização matemática do protocolo (notação criptográfica formal)

---

### 8. Conclusão

- 8.1 Síntese do protocolo proposto
- 8.2 Contribuições originais — cinco elementos combinados sem precedente em sistemas conhecidos
- 8.3 Limitações do trabalho e do protótipo
- 8.4 Trabalhos futuros

---

## Referências Base (a expandir)

### Sistemas Eleitorais
- ADIDA, B. et al. *Helios: Web-based Open-Audit Voting.* USENIX Security, 2008.
- BELL, S. et al. *STAR-Vote: A Secure, Transparent, Auditable, and Reliable Voting System.* USENIX EVT/WOTE, 2013.
- HEIBERG, S.; WILLEMSON, J. *Verifiable Internet Voting in Estonia.* 2014.
- BENALOH, J. *Simple Verifiable Elections.* USENIX EVT, 2006.
- RYAN, P. et al. *Prêt à Voter: a Voter-Verifiable Voting System.* IEEE Transactions on Information Forensics and Security, 2009.

### Criptografia
- NAKAMOTO, S. *Bitcoin: A Peer-to-Peer Electronic Cash System.* 2008.
- BEN-SASSON, E. et al. *Succinct Non-Interactive Zero Knowledge for a von Neumann Architecture.* USENIX Security, 2014.
- NIST SP 800-90A — *Recommendation for Random Number Generation Using Deterministic Random Bit Generators.*

### Entropia e Aleatoriedade
- CLOUDFLARE. *LavaRand in Production: The Nitty-Gritty Technical Details.* Blog técnico, 2017.
- NIST. *Randomness Beacon.* beacon.nist.gov — especificação e API pública.

### Teoria Democrática
- DAHL, R. *Poliarquia: Participação e Oposição.* Edusp, 2005.
- PRZEWORSKI, A. *Democracia e Mercado.* Relume-Dumará, 1994.

### Hardware de Segurança
- BYRES, E. *The Myths and Facts behind Cyber Security Risks for Industrial Control Systems.* VDE Congress, 2004.
- NIST SP 800-82 — *Guide to Industrial Control Systems Security.*

---

## Próximos Passos

- [ ] Confirmar orientador e normas ABNT da instituição
- [ ] Escrever introdução completa (~3 páginas)
- [ ] Implementar smart contract Solidity (append-only + commit-reveal)
- [ ] Implementar firmware ESP8266: UUID, HMAC-SHA256, TX1, TX2
- [ ] Implementar relay em TypeScript + integração ethers.js
- [ ] Implementar interface de auditoria pública
- [ ] Escrever seção 2.5 (verificabilidade formal) com referências Ryan e Benaloh

---

*Versão 0.2 — incorpora arquitetura dual TX, impressão em duas vias, commit-reveal para UUID de candidato, geração de seed com entropia física verificável e protótipo de referência com stack tecnológico definido.*
