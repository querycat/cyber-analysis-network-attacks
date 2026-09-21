# Análise do Ataque de Rede

## Objetivo

Com base na evidência coletada no log de tráfego do Wireshark (ver [`wireshark-TCP-HTTP-log.xlsx`](docs/wireshark-TCP-HTTP-log.xlsx)), este documento identifica o tipo de ataque de rede sofrido pelo servidor web da agência de viagens, explica como o ataque ocorreu tecnicamente e descreve seu impacto negativo no site e nos usuários.

## Identificação do ataque

O padrão observado no log — um volume anormalmente alto de pacotes TCP SYN enviados repetidamente por um único endereço IP de origem, sem o correspondente handshake completo (SYN → SYN-ACK → ACK) — é característico de um ataque de **negação de serviço (Denial of Service, DoS),** mais especificamente um **TCP SYN flood**.

Como todo o tráfego malicioso partiu de um único IP (e não de múltiplas origens distribuídas), trata-se de um DoS convencional, e não de um DDoS (Distributed Denial of Service).

## Como o ataque ocorreu

O TCP SYN flood explora o processo de estabelecimento de conexão do protocolo TCP, conhecido como _three-way handshake_:

O cliente envia um pacote **SYN** ao servidor, solicitando o início de uma conexão.
O servidor responde com um pacote **SYN-ACK**, reservando recursos (memória e uma entrada na fila de conexões) para essa conexão em potencial.
Em uma conexão legítima, o cliente responderia com um **ACK**, completando o handshake.

No ataque identificado, o atacante enviou um grande número de pacotes SYN ao servidor, mas nunca completou o handshake com o ACK final. Cada SYN recebido, porém, obrigou o servidor a alocar recursos e manter a conexão "meio aberta" (_half-open_) na fila, aguardando uma resposta que nunca chegava. Repetido em alta velocidade e em grande volume, esse comportamento esgotou os recursos disponíveis do servidor (memória e slots de conexão), impedindo-o de processar novas requisições, inclusive as de usuários legítimos.

## Impacto negativo no site

**Indisponibilidade do serviço**: o site da agência de viagens ficou fora do ar, retornando erro de _connection timeout_ para quem tentava acessá-lo.
**Interrupção da operação interna**: os funcionários, que dependem do site para consultar pacotes de viagem para os clientes, ficaram impossibilitados de realizar essa tarefa durante o incidente.
**Perda potencial de negócio**: enquanto o site esteve fora do ar, promoções e vendas anunciadas na página ficaram inacessíveis a clientes em potencial, com possível impacto direto na receita e na reputação da empresa.
**Consumo de recursos de infraestrutura**: o servidor precisou ser retirado de operação temporariamente para se recuperar da sobrecarga, gerando indisponibilidade adicional mesmo após o fim do fluxo de ataque.

## Observação sobre a mitigação aplicada

O bloqueio do IP de origem no firewall foi uma medida de contenção válida, porém temporária: como o TCP SYN flood pode ser executado com **IP spoofing** (falsificação do endereço de origem), um atacante pode contornar bloqueios baseados em IP simplesmente alternando os endereços de origem falsificados. Por isso, essa mitigação deve ser tratada como paliativa, e não como solução definitiva — o que reforça a necessidade de medidas adicionais, como limitação de taxa (_rate limiting_), uso de **SYN cookies**, e possivelmente serviços de proteção contra DDoS/DoS.
