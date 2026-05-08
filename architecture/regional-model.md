# Modelo Regional

---

## Princípio

O resultado nacional **nunca é determinado por volume absoluto de votos.** A eleição é decidida por regiões ganhas.

Esse princípio não é apenas político — é uma decisão de segurança arquitetural. Ele transforma a natureza do ataque necessário para alterar um resultado: em vez de comprometer volume suficiente de votos em qualquer lugar, um atacante precisa comprometer simultaneamente regiões suficientes para mudar a maioria de regiões ganhas.

---

## Por que Regionalização é uma Decisão de Segurança

### Sistema por volume absoluto (modelo atual)

```
Fraude em região X: +N votos para candidato A
Impacto: N votos a menos de diferença necessária
Custo de ataque: escala linearmente com a diferença de votos
```

### Sistema por regiões ganhas (modelo TEP)

```
Fraude em região X: candidato A ganha região X (1 região)
Para mudar resultado nacional: precisa ganhar regiões suficientes para maioria
Custo de ataque: escala com o número de regiões × custo de comprometer cada região
```

Comprometer uma região inteira — com redundância tripla — é exponencialmente mais custoso do que inserir votos avulsos em um sistema centralizado.

---

## Definição de Região

A granularidade exata das regiões é um ponto em aberto crítico (ver seção de pontos em aberto). Fatores a considerar:

| Granularidade | Vantagem | Desvantagem |
|---|---|---|
| Estado (27) | Simples, já existe juridicamente | Muito grande — revalidação cara |
| Município (~5.570) | Isola bem, revalidação barata | Resultado pode ser distorcido por municípios pequenos |
| Zona eleitoral (~3.000) | Granularidade intermediária | Exige redesenho jurídico |

A definição final deve ser baseada em dados históricos de distribuição eleitoral para garantir que o modelo não crie distorções sistêmicas.

---

## Protocolo de Revalidação Regional

Quando uma inconsistência é detectada na convergência tripla de uma região:

### Passo 1 — Isolamento

```
Região X sinalizada como inconsistente
  │
  ├── Resultado de X excluído do cômputo nacional provisório
  ├── Demais regiões: resultado mantido e válido
  └── Região X: estado → PENDENTE_REVALIDAÇÃO
```

### Passo 2 — Análise de Causa

Equipe técnica independente investiga:
- Qual vetor divergiu (urna, blockchain ou cédula)?
- Divergência é explicável por falha orgânica (hardware, extravio) ou é anomalia?
- Score do sistema de detecção de anomalias para a região

### Passo 3 — Decisão

| Diagnóstico | Ação |
|---|---|
| Falha orgânica identificada e explicável | Correção pontual com auditoria |
| Anomalia não explicável | Revalidação completa da região |
| Fraude confirmada | Processo legal + revalidação |

### Passo 4 — Revalidação

Nova organização constituída com membros escolhidos **aleatoriamente** da população local (modelo análogo ao júri popular):

- Nenhum membro da organização original pode participar
- Processo realizado sob observação de representantes das demais regiões
- Apenas a região X repete o processo
- Resultado das demais regiões permanece intocado

---

## Seleção Aleatória de Organizadores

A seleção aleatória é um mecanismo anti-captura. Se organizadores de revalidação fossem designados politicamente, a revalidação seria vulnerável ao mesmo vetor de ataque da eleição original.

**Requisitos do mecanismo de seleção:**
- Fonte de aleatoriedade verificável e pública (ex: NIST Randomness Beacon no momento da convocação)
- Pool de elegíveis: eleitores registrados na região sem vínculo com partidos ou candidatos envolvidos
- Processo público e documentado
- Número suficiente para representatividade estatística

---

## Pontos em Aberto

- **Granularidade exata das regiões:** requer análise de dados históricos eleitorais brasileiros
- **Número mínimo de regiões para maioria:** depende da granularidade escolhida
- **Prazo de revalidação:** quanto tempo é aceitável para manter resultado nacional em aberto?
- **Peso das regiões:** todas têm peso igual ou proporcional à população?
- **Tratamento de empate:** o que acontece se candidatos empatam em número de regiões ganhas?
