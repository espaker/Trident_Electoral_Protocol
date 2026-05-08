# Arquitetura de Redundância Tripla

---

## Os Três Vetores

O TEP opera sobre três vetores independentes e heterogêneos. Independência é o requisito crítico: os vetores devem usar tecnologias distintas, ter cadeias de custódia distintas e falhar por razões distintas — para que um atacante precise comprometer simultaneamente três sistemas de natureza completamente diferente.

```
Voto confirmado pelo eleitor
        │
        ├── TX1 ──► Impressora (UART, RX cortado)
        │               ├── Via 1: candidato + UUID ──► Urna física lacrada
        │               └── Via 2: só UUID           ──► Eleitor retém
        │
        └── TX2 ──► Módulo relay (hardware separado)
                        └── UDP fire-and-forget
                                └── Blockchain global
                                        └── { uuid_voto, commit(uuid_candidato) }
```

Os dois transmissores disparam em paralelo a partir de um único evento atômico. Não existe estado intermediário válido: ou ambos transmitem, ou nenhum.

---

## Vetor 1 — Urna Eletrônica

**Função:** registro digital local, fonte única e autoritativa de geração de UUID.

**Propriedades críticas:**
- Nunca recebe dados externos — pino RX eletricamente removido
- Gera o UUID do voto a partir de entropia local + seed da eleição
- Armazena log local de todos os votos da sessão
- Não conhece o UUID do candidato em texto claro — apenas seu commit

**Por que é necessário:**
A urna é o ponto de captura do voto — sem ela, não existe registro. Mas sozinha é um ponto único de confiança. Os outros dois vetores são âncoras independentes do registro gerado aqui.

---

## Vetor 2 — Blockchain Global

**Função:** registro imutável e distribuído, verificável por qualquer pessoa com acesso à internet.

**Propriedades críticas:**
- Append-only por design de protocolo
- Nós geograficamente distribuídos — sem autoridade central
- Dados chegam via UDP unidirecional — nós nunca fazem request para urnas
- Commit do UUID do candidato registrado — candidato real revelado apenas após encerramento

**Por que é necessário:**
A urna pode ser comprometida fisicamente. A blockchain garante que o registro do voto existe em múltiplos lugares independentes antes que qualquer agente físico possa interferir.

---

## Vetor 3 — Cédula Impressa Física

**Função:** âncora física auditável sem nenhum equipamento digital.

**Propriedades críticas:**
- Impressa pelo próprio eleitor no momento do voto
- Depositada pelo próprio eleitor na urna física lacrada
- Contém QR Code com UUID do voto — verificável por qualquer leitor de QR
- Auditável por cidadão comum sem conhecimento técnico

**Por que é necessário:**
Sistemas digitais podem falhar ou ser comprometidos de forma coordenada. A cédula física é a âncora que qualquer pessoa pode contar, manusear e verificar — sem depender de nenhum software.

---

## UUID Atômico

O UUID do voto é o fio condutor entre os três vetores:

```
UUID_voto = SHA-256(seed_eleição ‖ session_id ‖ timestamp ‖ entropia_local)
```

| Campo | Origem | Propriedade garantida |
|---|---|---|
| seed_eleição | Cerimônia pública MPC | Imprevisível e verificável |
| session_id | Configuração da urna | Unicidade por urna/sessão |
| timestamp | Relógio local da urna | Ordem temporal dos votos |
| entropia_local | TRNG do hardware (esp_random) | Imprevisibilidade local |

O mesmo UUID aparece nos três vetores. Qualquer divergência é detectável voto a voto.

---

## Recibo do Eleitor

A impressora gera duas vias:

**Via 1 — Cédula oficial:**
```
┌─────────────────────────────────────┐
│ CANDIDATO: [nome + número]          │
│ UUID: a3f7b2c1-...                  │
│ [QR Code]                           │
│ Zona: 042 · Seção: 0127 · 14:23:07 │
└─────────────────────────────────────┘
Depositada pelo eleitor na urna física lacrada
```

**Via 2 — Recibo do eleitor:**
```
┌─────────────────────────────────────┐
│ SEU COMPROVANTE DE VOTO             │
│ UUID: a3f7b2c1-...                  │
│ [QR Code]                           │
│ Consulte: tep.eleitoral/verificar   │
└─────────────────────────────────────┘
Eleitor retém — permite verificação pessoal na blockchain
```

O recibo não revela o candidato. Não é coercível: mostrar o recibo a terceiros prova que o voto foi registrado, mas não prova em quem se votou (o UUID do candidato só é revelado após encerramento e a ligação exige consulta técnica).

---

## Convergência

A eleição é válida quando os três vetores convergem dentro da margem de tolerância definida por vetor.

| Vetor | Tolerância | Ação em caso de divergência |
|---|---|---|
| Urna Eletrônica | 0% | Isolamento regional + revalidação |
| Blockchain | 0% | Isolamento regional + revalidação |
| Cédula Impressa | Margem pequena (a definir empiricamente) | Log + score de anomalia |

Divergência acima da margem dispara o protocolo de revalidação regional descrito em [regional-model.md](regional-model.md).
