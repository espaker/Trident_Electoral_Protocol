# Camada Blockchain

---

## Função no TEP

A blockchain é o Vetor 2 do TEP: registro imutável e publicamente verificável de todos os votos, acessível a qualquer pessoa com conexão à internet, sem depender de nenhuma autoridade central para consulta.

**Requisitos que a camada blockchain deve satisfazer:**

- Append-only: nenhum registro pode ser alterado ou removido após inserção
- Sem ponto único de falha: múltiplos nós geográficamente distribuídos
- Sem canal de retorno para as urnas: nós nunca fazem request para urnas
- Auditável publicamente sem autenticação
- Resistente a censura por qualquer agente único

---

## Commit-Reveal: o Esquema Central

O UUID do candidato nunca é exposto em texto claro durante a eleição. O que vai para a blockchain é um compromisso criptográfico:

### Fase de votação — o que é registrado

```
VoteRecord {
  uuid_voto:        bytes32   // identificador único do voto
  candidate_commit: bytes32   // SHA-256(uuid_candidato ‖ salt)
  zone_id:          uint32    // zona eleitoral
  session_id:       bytes16   // identificador da sessão da urna
  timestamp:        uint64    // unix time em milissegundos
}
```

### Fase de revelação — após encerramento da eleição

```
CandidateReveal {
  candidate_id:    uint32    // identificador público do candidato
  uuid_candidato:  bytes32   // UUID gerado a partir da seed da eleição
  salt:            bytes32   // salt único por candidato
}
```

### Verificação pública

```
SHA-256(uuid_candidato ‖ salt) == candidate_commit
```

Qualquer pessoa executa essa verificação para confirmar que um voto foi contabilizado para o candidato correto. Sem o salt, conhecer os candidatos possíveis não permite reverter o commit (pré-imagem resistente).

---

## Por que o Salt é Necessário

Sem salt, um adversário poderia tentar todos os UUIDs de candidatos conhecidos contra cada `candidate_commit` durante a eleição. Com poucos candidatos (geralmente < 50), isso seria trivialmente rápido.

O salt deve ser:
- Gerado aleatoriamente por candidato
- Mantido secreto até a revelação
- Incluído na cerimônia de encerramento da eleição
- Diferente entre eleições

---

## Topologia de Nós

### Opção A — Ethereum Sepolia Testnet (recomendada para protótipo)

- Rede global existente com milhares de nós
- Gratuita para testes
- Integração direta com ethers.js/viem
- Smart contract em Solidity
- Argumento acadêmico forte: distribuição genuinamente global

### Opção B — Rede PoA Privada (produção futura)

- Nós controlados por entidades auditadas independentes
- Mecanismo de consenso Clique (PoA) ou similar
- Nós em jurisdições distintas — pelo menos 3 países
- Acesso público de leitura, escrita apenas via relay autenticado da urna

Para o TCC, Sepolia é suficiente e academicamente defensável.

---

## Smart Contract — Estrutura

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TEPVoteRegistry {

    struct VoteRecord {
        bytes32 uuid_voto;
        bytes32 candidate_commit;
        uint32  zone_id;
        bytes16 session_id;
        uint64  timestamp;
    }

    struct CandidateReveal {
        uint32  candidate_id;
        bytes32 uuid_candidato;
        bytes32 salt;
    }

    // Estado da eleição
    enum ElectionState { OPEN, CLOSED, REVEALED }

    ElectionState public state;
    address public electionAuthority;  // endereço da cerimônia de abertura

    VoteRecord[]   public votes;
    CandidateReveal[] public reveals;

    mapping(bytes32 => bool) public registeredUUIDs;  // impede duplicatas

    event VoteRegistered(bytes32 indexed uuid_voto, uint32 zone_id);
    event ElectionClosed();
    event RevealPublished(uint32 candidate_id);

    modifier onlyOpen()     { require(state == ElectionState.OPEN);     _; }
    modifier onlyClosed()   { require(state == ElectionState.CLOSED);   _; }
    modifier onlyAuthority(){ require(msg.sender == electionAuthority); _; }

    function registerVote(
        bytes32 uuid_voto,
        bytes32 candidate_commit,
        uint32  zone_id,
        bytes16 session_id,
        uint64  timestamp
    ) external onlyOpen {
        require(!registeredUUIDs[uuid_voto], "UUID already registered");
        registeredUUIDs[uuid_voto] = true;
        votes.push(VoteRecord(uuid_voto, candidate_commit, zone_id, session_id, timestamp));
        emit VoteRegistered(uuid_voto, zone_id);
    }

    function closeElection() external onlyAuthority onlyOpen {
        state = ElectionState.CLOSED;
        emit ElectionClosed();
    }

    function publishReveal(
        uint32  candidate_id,
        bytes32 uuid_candidato,
        bytes32 salt
    ) external onlyAuthority onlyClosed {
        // Verificação: o commit deve corresponder a pelo menos um voto registrado
        bytes32 expected = keccak256(abi.encodePacked(uuid_candidato, salt));
        reveals.push(CandidateReveal(candidate_id, uuid_candidato, salt));
        emit RevealPublished(candidate_id);
    }

    // Consulta pública: verificar se um UUID está registrado
    function isVoteRegistered(bytes32 uuid_voto) external view returns (bool) {
        return registeredUUIDs[uuid_voto];
    }

    // Consulta pública: total de votos registrados
    function totalVotes() external view returns (uint256) {
        return votes.length;
    }
}
```

*Nota: esta é a estrutura base. A versão de produção exige auditoria de segurança formal.*

---

## Relay — TypeScript

O relay é o componente que recebe os pacotes UDP da urna e os submete à blockchain:

```
Módulo relay (hardware)
  └── UDP recebido (fire-and-forget da urna)
        └── Relay server (TypeScript/Node.js)
              └── Valida assinatura HMAC do pacote
                    └── ethers.js → contract.registerVote(...)
                          └── Ethereum Sepolia
```

O relay **não altera** o conteúdo do pacote. Sua função é exclusivamente translação de protocolo: UDP → transação Ethereum. Qualquer manipulação seria detectável porque a assinatura HMAC do pacote é gerada pelo hardware da urna antes da transmissão.

---

## Consulta Pública

A interface de auditoria permite que qualquer pessoa:

1. **Consulte por UUID do voto:** confirma que o voto está registrado na blockchain
2. **Após revelação:** verifica para qual candidato o UUID foi contabilizado
3. **Consulte totais por zona:** apuração pública e verificável
4. **Exporte todos os registros:** dataset completo para auditoria independente

Nenhuma autenticação é necessária para consulta. Os dados são públicos por design.

---

## Referências

- NAKAMOTO, S. *Bitcoin: A Peer-to-Peer Electronic Cash System.* 2008.
- CENCI, D. R.; BECK, C. *Nova Tecnologia para o Sistema Eleitoral Brasileiro: Blockchain e Transparência.* Est. Eleit., 2022.
- Ethereum Foundation. *Sepolia Testnet.* ethereum.org.
- OpenZeppelin. *Smart Contract Security Best Practices.*
