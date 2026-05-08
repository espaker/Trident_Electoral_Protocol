# Modelo de Ameaças — Vetores de Ataque

> Metodologia: STRIDE aplicado ao TEP

---

## STRIDE

| Categoria | Descrição | Mitigação no TEP |
|---|---|---|
| **S**poofing | Forjar identidade de voto ou urna | UUID único derivado de entropia irreproduzível + HMAC assinado pelo hardware da urna |
| **T**ampering | Alterar registro de voto após criação | Blockchain append-only + cédula física como âncora independente |
| **R**epudiation | Negar que um voto foi registrado | UUID não repudiável registrado em três vetores heterogêneos simultaneamente |
| **I**nformation Disclosure | Revelar em quem o eleitor votou | Commit-reveal: UUID do candidato opaco durante eleição; salt impede brute-force |
| **D**enial of Service | Impedir contagem ou invalidar eleição | Falha localizada por região — uma zona comprometida não invalida o país |
| **E**levation of Privilege | Tomar controle da urna remotamente | Hardware TX-only com pino RX eletricamente removido — superfície de ataque remoto inexistente |

---

## Vetores de Ataque Detalhados

### 1. Comprometimento físico da urna pré-eleição

**Descrição:** atacante com acesso físico à urna antes da eleição implanta firmware malicioso que gera UUIDs previsíveis ou duplicados, ou altera o candidato registrado antes de transmitir.

**Por que é difícil:**
- Requer acesso físico durante cadeia de custódia
- Firmware é verificado por hash antes da eleição (cerimônia de auditoria)
- Urnas comprometidas gerariam UUIDs inconsistentes com a seed pública da eleição — detectável na convergência

**Mitigação atual:** verificação de hash de firmware em cerimônia pública antes do lacramento das urnas.

**Mitigação ausente:** secure boot verificável por terceiros no hardware — ponto em aberto.

---

### 2. Captura de nós da blockchain

**Descrição:** atacante controla nós suficientes da rede para tentar reescrever histórico ou censurar transações de zonas específicas.

**Por que é difícil:**
- Requer captura de maioria dos nós (51% em PoW, supermaioria em PoA)
- Nós em jurisdições distintas exigem comprometimento multi-jurisdicional simultâneo
- Cédula física é âncora independente: mesmo com blockchain comprometida, contagem física detecta divergência

**Mitigação atual:** distribuição geográfica dos nós + redundância tripla.

**Mitigação ausente:** mecanismo formal de auditoria de nós e critérios de exclusão — ponto em aberto.

---

### 3. Falsificação em massa de cédulas físicas

**Descrição:** atacante produz cédulas físicas falsas com UUIDs inventados para inflar contagem do Vetor 3.

**Por que é difícil:**
- UUID da cédula deve estar registrado na blockchain para passar na convergência
- Gerar UUIDs válidos exige conhecer o HMAC da urna e a seed da eleição
- Inserção de cédulas físicas exige acesso às urnas lacradas durante a votação ou transporte

**Mitigação atual:** convergência com blockchain — cédula sem UUID correspondente é descartada.

**Mitigação ausente:** autenticação adicional na cédula (ex: tinta especial, marca d'água criptográfica) — ponto em aberto desejável.

---

### 4. Comprometimento da cerimônia de geração de seed

**Descrição:** atacante controla a geração da seed da eleição, tornando-a previsível. Com a seed, pode pré-computar os UUIDs de candidatos e potencialmente reverter commits durante a eleição.

**Por que é difícil:**
- MPC exige comprometimento simultâneo de todas as partes contribuintes
- Fontes físicas de entropia são publicamente observáveis — manipulação seria detectável
- Seed publicada na blockchain antes da eleição — qualquer alteração posterior é rastreável

**Mitigação atual:** MPC com múltiplas partes independentes + cerimônia pública documentada.

**Mitigação ausente:** protocolo MPC formal especificado — ponto em aberto crítico.

---

### 5. Ataque ao relay (interceptação do UDP)

**Descrição:** atacante intercepta os pacotes UDP entre o módulo relay e a blockchain, alterando o conteúdo antes do registro.

**Por que é difícil:**
- Pacotes UDP são assinados com HMAC pelo hardware da urna antes da transmissão
- HMAC usa chave derivada da seed da eleição — chave não é extraível do hardware sem acesso físico
- Pacote modificado falha na verificação HMAC no relay server antes de ser submetido à blockchain

**Mitigação atual:** HMAC de hardware no pacote UDP.

**Mitigação ausente:** especificação formal do protocolo de transmissão TX2 — ponto em aberto importante.

---

### 6. Coerção via recibo do eleitor

**Descrição:** agente coercitivo exige que o eleitor apresente seu recibo de UUID como prova de voto em determinado candidato.

**Por que é difícil:**
- O recibo contém apenas o UUID do voto — não revela o candidato diretamente
- Durante a eleição, o UUID do candidato está protegido pelo commit (hash + salt)
- Após a revelação, estabelecer a ligação exige consulta técnica à blockchain — não é apresentável como "prova simples" a um coercitivo não-técnico
- Eleitor pode alegar que votou em qualquer candidato sem forma de refutação imediata

**Mitigação atual:** separação entre UUID do voto e UUID do candidato + commit-reveal.

**Mitigação ausente:** análise formal de receipt-freeness — o TEP não implementa receipt-freeness criptográfica completa (como em Prêt à Voter), mas oferece proteção prática razoável. Ponto em aberto para pesquisa futura.

---

### 7. Negação de serviço na blockchain durante a eleição

**Descrição:** atacante satura a rede blockchain com transações lixo para impedir o registro de votos válidos.

**Por que é relevante em redes públicas:** em Ethereum mainnet/testnet, gas wars são possíveis.

**Mitigação para rede própria (PoA):** nós são permissionados — transações de fontes não autorizadas são rejeitadas. Apenas relays autenticados submetem votos.

**Mitigação para rede pública (Sepolia/mainnet):** prioridade de gas configurável no relay + mecanismo de retry com fila local no relay server. Votos não perdidos — ficam na fila até confirmação.

---

## Custo de Ataque — Comparativo

| Ataque | Sistema centralizado | TEP |
|---|---|---|
| Alterar resultado de uma zona | Comprometer 1 servidor central | Comprometer 3 vetores heterogêneos + passar pela convergência |
| Alterar resultado nacional | Comprometer 1 sistema de totalização | Comprometer maioria das regiões × 3 vetores cada |
| Ocultar ataque | Possível — logs controláveis | Impossível — blockchain pública é permanente e auditável |
| Ataque remoto à urna | Possível se urna tem pino RX | Impossível — pino RX eletricamente removido |

---

## Superfície de Ataque Residual

O TEP não é invulnerável — nenhum sistema é. Os vetores residuais após todas as mitigações:

1. **Comprometimento físico de urnas na cadeia de custódia** — mitigação parcial via auditoria de firmware
2. **Captura da totalidade dos nós blockchain** — improvável com distribuição geográfica suficiente
3. **Coerção em escala regional** — fora do escopo técnico; problema político/social
4. **Vulnerabilidade zero-day no hardware do ESP8266** — aceita como risco de componente; mitigável com hardware customizado em produção

A filosofia do TEP é que o custo de comprometimento deve exceder o benefício em qualquer cenário realista de ataque.
