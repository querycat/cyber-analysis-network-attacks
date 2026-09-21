# Relatório de Incidente de Cibersegurança

## Seção 1: Que tipo de ataque que pode ter causado esta interrupção na rede

Uma possível explicação para a mensagem de erro de tempo esgotado na conexão do site é: o servidor web está sobrecarregado a ponto de não conseguir responder a novas requisições dentro do tempo esperado, fazendo com que a conexão expire (_timeout_) antes de ser estabelecida.

**Os logs mostram que**: um volume anormalmente alto de pacotes **TCP SYN** foi enviado repetidamente ao servidor a partir de um único endereço IP não reconhecido, sem que o handshake de conexão fosse completado com o ACK final esperado.

**Este evento pode ser**: um ataque de **negação de serviço (DoS)**, do tipo **TCP SYN flood** — como todo o tráfego malicioso partiu de uma única origem, e não de múltiplas origens distribuídas, não se trata de um **DDoS (Distributed Denial of Service)**.

## Seção 2: Como o ataque está causando o mau funcionamento do site

Quando os visitantes do site tentam estabelecer uma conexão com o servidor web, ocorre um handshake de três vias usando o protocolo TCP. Explique os três passos do handshake:

1. O cliente envia um pacote **SYN** ao servidor, solicitando o início de uma conexão.
2. O servidor responde com um pacote **SYN-ACK**, confirmando o recebimento e reservando recursos (memória e uma entrada na fila de conexões) para essa conexão em potencial.
3. O cliente responde com um pacote **ACK**, confirmando o recebimento do SYN-ACK e completando o handshake. A partir daí a conexão é considerada estabelecida.

### O que acontece quando um ator malicioso envia um grande número de pacotes SYN de uma só vez:

O servidor responde a cada SYN com um SYN-ACK e reserva recursos para a conexão, aguardando o ACK final. Como o atacante nunca envia esse ACK, cada conexão fica "meio aberta" (_half-open_) na fila do servidor até expirar por tempo limite. Ao repetir esse envio em alto volume e velocidade, o atacante consegue esgotar os recursos disponíveis (memória e slots de conexão) do servidor muito mais rápido do que eles são liberados.

### O que os logs indicam e como isso afeta o servidor:

Os logs indicam um número muito acima do normal de pacotes SYN vindos de um único IP, sem os correspondentes handshakes completos. Isso confirma que o servidor estava recebendo mais solicitações de conexão do que sua capacidade de processá-las, esgotando seus recursos e impedindo-o de responder a requisições legítimas — tanto de clientes acessando o site quanto de funcionários consultando pacotes de viagem internamente. O resultado observado foi a indisponibilidade total do serviço, com erro de tempo esgotado na conexão para qualquer usuário.