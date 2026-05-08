# Sistema de Detecção de Anomalias

---

## Princípio

O sistema de detecção de anomalias é **passivo e separado** do fluxo de validação principal. Ele não bloqueia resultados nem veta regiões — gera scores de confiabilidade por zona eleitoral que alimentam relatórios públicos e constroem histórico auditável para eleições futuras.

A separação é intencional: um sistema que pode bloquear resultados é um vetor de ataque (Denial of Service eleitoral). Um sistema que apenas reporta não pode ser usado para impedir uma eleição — mas cria evidência permanente de qualquer irregularidade.

---

## O que é Monitorado

### 1. Taxa de extravio de cédulas por zona

```
taxa_extravio(zona) = (votos_blockchain - cédulas_contadas) / votos_blockchain
```

- Extravio orgânico esperado: pequena margem positiva (cédulas rasgadas, danificadas, perdidas no transporte)
- Anomalia: taxa negativa (mais cédulas do que votos digitais — indica inserção física) ou taxa muito acima do histórico

### 2. Padrões estatísticos de divergência por região

- Divergência entre urna e blockchain por zona (deve ser 0%)
- Correlação geográfica: anomalias em zonas adjacentes são suspeitas diferentes de anomalias isoladas
- Distribuição temporal: votos registrados fora do horário eleitoral

### 3. Correlações temporais anômalas na blockchain

- Rafagas de registros em timestamps muito próximos (possível replay de pacotes)
- Gaps temporais anormais dentro de uma sessão
- UUIDs com colisão parcial (indica problema na geração de entropia)

---

## Score de Confiabilidade

Cada zona eleitoral recebe um score ao final da eleição:

```
score(zona) = f(
  taxa_extravio,
  divergência_urna_blockchain,
  anomalias_temporais,
  histórico_eleições_anteriores
)
```

O score é:
- **Público:** qualquer pessoa pode consultar o score de qualquer zona
- **Explicado:** o relatório detalha quais métricas contribuíram para o score
- **Histórico:** comparado com eleições anteriores da mesma zona
- **Não-bloqueante:** score baixo gera alerta, não invalida automaticamente

---

## Relatório Pós-Eleição

Ao encerramento, o sistema publica automaticamente:

```
Relatório de Auditoria — Eleição YYYY
├── Resumo nacional: score médio ponderado
├── Mapa de scores por zona (visualização pública)
├── Top anomalias detectadas (ordenadas por severidade)
├── Zonas com score abaixo do threshold de atenção
├── Comparativo histórico por zona
└── Dataset completo para auditoria independente (CSV/JSON)
```

O relatório é publicado na blockchain — imutável e com timestamp verificável.

---

## Histórico como Ferramenta de Detecção

Padrões que são invisíveis em uma eleição isolada tornam-se detectáveis ao longo do tempo:

- Uma zona que consistentemente tem taxa de extravio levemente acima da média pode indicar problema estrutural (ou fraud sistemática de baixa intensidade)
- Melhora repentina de score em uma zona antes considerada problemática pode indicar mudança de organização (positiva ou negativa)
- Correlação entre zonas com score baixo e características demográficas ou geográficas pode revelar vulnerabilidades estruturais do sistema

---

## Implementação

O sistema de detecção roda como processo separado, lendo apenas os dados públicos da blockchain e os dados de contagem de cédulas físicas — sem acesso direto às urnas ou ao processo de apuração.

**Stack:**
- TypeScript (Node.js) — processamento dos dados
- Dados de entrada: blockchain pública (ethers.js) + relatórios de contagem física (JSON)
- Saída: relatório JSON/HTML + registro na blockchain

**Threshold de atenção:** a ser definido empiricamente com base em dados históricos de eleições brasileiras. Ponto em aberto crítico para a especificação completa.
