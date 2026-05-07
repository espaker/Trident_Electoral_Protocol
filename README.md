# Trident Electoral Protocol (TEP)

> *Documento inicial de especificação conceitual — versão 0.1*

---

## Motivação

Os sistemas eleitorais modernos falham em combinar auditabilidade humana, eficiência digital e imutabilidade distribuída de forma simultânea. A maioria sacrifica uma camada em favor de outra. O TEP nasce da premissa de que os três mundos são complementares, não excludentes, e que cada camada deve cobrir as fraquezas das demais.

---

## Princípios Fundamentais

1. **A urna é um emissor, nunca um receptor.**
2. **Nenhum resultado é válido sem convergência tripla.**
3. **Todo voto é atômico, rastreável e não repudiável.**
4. **Falhas são localizáveis e corrigíveis sem invalidar o todo.**
5. **Auditabilidade deve ser acessível a humanos comuns e a técnicos.**

---

## Arquitetura Geral

O TEP opera sobre três vetores independentes de registro, que devem convergir para que um resultado seja considerado válido:

```
Voto confirmado pelo eleitor
        │
        ├── TX1 ──► Impressora
        │               └── Cédula impressa com QR Code (UUID + hash do voto)
        │                       └── Depositada em urna física pelo eleitor
        │
        └── TX2 ──► Blockchain
                        └── Registro imutável do UUID + hash do voto
```

Os dois transmissores (TX1 e TX2) disparam em paralelo a partir de um único evento atômico — ou ambos transmitem, ou nenhum. Não existe estado intermediário válido.

---

## Os Três Vetores

### Vetor 1 — Urna Eletrônica
- Registro digital local do voto
- Geração do UUID do voto
- Única fonte de geração — nunca recebe dados externos
- Hardware com TX físico apenas, sem pino RX conectado

### Vetor 2 — Blockchain
- Registro distribuído e imutável
- Recebe pacotes assinados da urna via TX2
- Nós da rede são append-only para dados de urna — nunca fazem request de volta
- Validação por consenso distribuído

### Vetor 3 — Cédula Impressa Física
- Impressa via TX1 durante o ato do voto
- Contém QR Code com UUID + hash do voto
- Depositada pelo próprio eleitor em urna física lacrada
- Auditável por qualquer pessoa sem necessidade de equipamento digital

---

## UUID do Voto

Cada voto recebe um UUID único gerado a partir de:

```
UUID = hash(seed_eleição + session_id + timestamp + entropia_local)
```

- **Seed da eleição:** gerada no início de cada eleição, nunca reutilizada
- **Session_id:** identificador da sessão daquela urna específica
- **Timestamp:** momento do voto
- **Entropia local:** dado imprevisível gerado localmente pela urna

O UUID é o fio condutor entre os três vetores — permite comparação atômica voto a voto sem revelar a identidade do eleitor.

O UUID do **candidato** é separado, também gerado por seed, e **só é tornado público após o encerramento da eleição**, impedindo rastreabilidade em tempo real.

---

## Modelo Regional

O resultado nacional **nunca é determinado por volume absoluto de votos.** A eleição é decidida por regiões ganhas, tornando:

- Fraude em massa localmente menos impactante no resultado geral
- Revalidação cirúrgica possível — inconsistência em uma região não invalida o país
- Custo de ataque muito mais alto — exige comprometimento simultâneo de múltiplas regiões

Em caso de inconsistência detectada:

1. A região específica é isolada
2. Nova organização é convocada com membros escolhidos **aleatoriamente**
3. Apenas aquela região refaz o processo
4. O resultado das demais regiões permanece válido

---

## Validação e Convergência

A eleição é considerada válida quando os três vetores convergem dentro de uma **margem de tolerância definida.**

| Vetor | Tolerância | Justificativa |
|---|---|---|
| Urna Eletrônica | 0% | Sistema digital — divergência é anomalia grave |
| Blockchain | 0% | Imutável por design — divergência indica comprometimento |
| Cédula Impressa | Margem pequena (a definir) | Extravio físico orgânico é possível |

Divergência acima da margem dispara revalidação regional automática.

---

## Hardware — Princípio Unidirecional

A urna implementa isolamento físico de recepção:

- Chip TX-only sem pino RX eletricamente conectado
- Impossibilidade de receber dados por design de hardware, não apenas por software
- Elimina toda a classe de ataques remotos: injeção de payload, alteração de firmware via rede, man-in-the-middle
- Equivalente funcional a um **data diode físico**, padrão usado em infraestrutura crítica

---

## Detecção de Anomalias

Separado do fluxo de validação principal, um sistema passivo monitora:

- Taxa de extravio de cédulas por zona eleitoral
- Padrões estatísticos de divergência por região
- Correlações temporais anômalas nos registros da blockchain

Este sistema **não bloqueia resultados** — gera score de confiabilidade por zona, alimenta relatório público pós-eleição e cria histórico auditável para eleições futuras.

---

## Pontos em Aberto — Estudo Futuro

### 🔴 Críticos (necessários para MVP conceitual)

- [ ] **Identidade do eleitor:** como garantir unicidade do voto sem comprometer anonimato? Biometria, título digital, ZK-proof?
- [ ] **Threshold exato de tolerância** para cédulas físicas — baseado em dados históricos de extravio orgânico
- [ ] **Definição formal das regiões** — granularidade (município, zona eleitoral, estado?)
- [ ] **Modelo de consenso da blockchain** — PoA com nós auditados? Qual mecanismo garante que os nós não são capturados?

### 🟡 Importantes (necessários para especificação completa)

- [ ] **Protocolo de transmissão TX2** — UDP? protocolo customizado? como garantir entrega sem ACK de volta?
- [ ] **Gestão da seed da eleição** — quem gera, quando, como é distribuída às urnas sem criar ponto único de comprometimento?
- [ ] **Cerimônia de abertura e encerramento** da urna — procedimentos físicos e digitais
- [ ] **Formato do QR Code** — dados mínimos necessários, resistência a leitura não autorizada em massa

### 🟢 Desejáveis (refinamento)

- [ ] **Score de confiabilidade** — algoritmo de detecção de anomalia estatística por zona
- [ ] **Interface de auditoria pública** — como qualquer cidadão consulta a convergência dos três vetores?
- [ ] **Modelo de seleção aleatória** de organizadores para revalidação regional
- [ ] **Compatibilidade com sistemas legados** — migração gradual ou substituição total?

---

## Referências e Sistemas Relacionados

Sistemas existentes que implementam partes do TEP:

| Sistema | País | O que implementa | O que falta |
|---|---|---|---|
| Helios Voting | Internacional | Auditabilidade criptográfica | Camada física, regionalização |
| STAR-Vote | EUA (proposta) | Urna + impresso + auditoria | Blockchain, regionalização |
| Estônia (i-Voting) | Estônia | Voto digital verificável | Redundância física, TX-only |
| Alemanha | Alemanha | Paper trail auditável | Componente digital |

**Nenhum sistema conhecido combina os cinco elementos centrais do TEP simultaneamente:**
redundância tripla + hardware unidirecional + UUID atômico + regionalização de resultado + revalidação parcial.

---

## Histórico

| Data | Evento |
|---|---|
| 2025-05-07 | Concepção inicial do modelo — discussão informal documentada |

---

*TEP é um protocolo conceitual em desenvolvimento. Contribuições, críticas técnicas e estudos derivados são bem-vindos, desde que com atribuição de autoria conforme licença CC-BY 4.0.*

---

**Licença:** Creative Commons Attribution 4.0 International (CC BY 4.0)
