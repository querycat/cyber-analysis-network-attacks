# Como ler um log TCP/HTTP do Wireshark

Nesta leitura, você aprenderá como ler um log TCP/HTTP do Wireshark para o tráfego de rede entre visitantes do site da empresa e o servidor web da companhia. A maioria das ferramentas de análise de protocolo/tráfego de rede usadas para capturar pacotes fornecerá essas mesmas informações.

## Número da entrada do log e tempo

| No. | Time     |
|------|----------|
| 47   | 3.144521 |
| 48   | 3.195755 |
| 49   | 3.246989 |

Esta seção do log TCP do Wireshark fornecida começa na entrada de log número (No.) 47, que é três segundos e 0,144521 milissegundos após a ferramenta de registro começar a gravar. Isso indica que aproximadamente 47 mensagens foram enviadas e recebidas pelo servidor web nos 3,1 segundos após o início do log. Essa velocidade rápida de tráfego é o motivo pelo qual a ferramenta rastreia o tempo em milissegundos.

## Endereços IP de origem e destino

| Origem        | Destino      |
|---------------|--------------|
| 198.51.100.23 | 192.0.2.1    |
| 192.0.2.1     | 198.51.100.23|
| 198.51.100.23 | 192.0.2.1    |

As colunas de origem e destino contêm o endereço IP da máquina que está enviando um pacote e o endereço IP de destino pretendido do pacote. Neste arquivo de log, o endereço IP 192.0.2.1 pertence ao servidor web da empresa. A faixa de endereços IP 198.51.100.0/24 pertence aos computadores dos funcionários.

## Tipo de protocolo e informações relacionadas

| Protocolo | Info                                   |
|-----------|---------------------------------------|
| TCP       | 42584->443 [SYN] Seq=0 Win-5792 Len=120... |
| TCP       | 443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| TCP       | 42584->443 [ACK] Seq=1 Win-5792 Len=120... |

A coluna Protocolo indica que os pacotes estão sendo enviados usando o protocolo TCP, que está na camada de transporte do modelo TCP/IP. No arquivo de log fornecido, você notará que o protocolo eventualmente muda para HTTP, na camada de aplicação, uma vez que a conexão com o servidor web é estabelecida com sucesso.

A coluna Info fornece informações sobre o pacote. Ela lista a porta de origem seguida por uma seta → apontando para a porta de destino. Neste caso, a porta 443 pertence ao servidor web. A porta 443 é normalmente usada para tráfego web criptografado.

O próximo elemento de dados na coluna Info faz parte do processo de handshake de três vias para estabelecer uma conexão entre duas máquinas. Neste caso, os funcionários estão tentando se conectar ao servidor web da empresa:

- O pacote [SYN] é a solicitação inicial de um visitante funcionário tentando se conectar a uma página web hospedada no servidor. SYN significa "synchronize" (sincronizar).
- O pacote [SYN, ACK] é a resposta do servidor web à solicitação do visitante, concordando com a conexão. O servidor reserva recursos do sistema para a etapa final do handshake. SYN, ACK significa "synchronize acknowledge" (sincronizar e reconhecer).
- O pacote [ACK] é a máquina do visitante reconhecendo a permissão para conectar. Esta é a etapa final necessária para estabelecer uma conexão TCP bem-sucedida. ACK significa "acknowledge" (reconhecer).

Os próximos itens na coluna Info fornecem mais detalhes sobre os pacotes. No entanto, esses dados não são necessários para completar esta atividade. Se desejar aprender mais sobre propriedades de pacotes, visite a Introdução da Microsoft à Análise de Rastreamento de Rede.

## Tráfego normal do site

Uma transação normal entre um visitante do site e o servidor web seria assim:

| No. | Time     | Source        | Destination   | Protocol | Info                              |
|------|----------|---------------|---------------|----------|----------------------------------|
| 47   | 3.144521 | 198.51.100.23 | 192.0.2.1     | TCP      | 42584->443 [SYN] Seq=0 Win=5792 Len=120... |
| 48   | 3.195755 | 192.0.2.1     | 198.51.100.23 | TCP      | 443->42584 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| 49   | 3.246989 | 198.51.100.23 | 192.0.2.1     | TCP      | 42584->443 [ACK] Seq=1 Win-5792 Len=120... |
| 50   | 3.298223 | 198.51.100.23 | 192.0.2.1     | HTTP     | GET /sales.html HTTP/1.1          |
| 51   | 3.349457 | 192.0.2.1     | 198.51.100.23 | HTTP     | HTTP/1.1 200 OK (text/html)       |

Note que o processo de handshake leva alguns milissegundos para ser concluído. Em seguida, você pode identificar o navegador do funcionário solicitando a página sales.html usando o protocolo HTTP na camada de aplicação do modelo TCP/IP, seguido pela resposta do servidor web à solicitação.

## O Ataque

Atores maliciosos podem tirar vantagem do protocolo TCP inundando um servidor com pacotes SYN para a primeira parte do handshake. No entanto, se o número de solicitações SYN for maior do que os recursos do servidor disponíveis para lidar com as solicitações, o servidor ficará sobrecarregado e incapaz de responder.

Este é um ataque de negação de serviço (DoS) em nível de rede, chamado ataque de inundação SYN (SYN flood), que visa a largura de banda da rede para desacelerar o tráfego. Um ataque de inundação SYN simula uma conexão TCP e inunda o servidor com pacotes SYN.

Um ataque DoS direto origina-se de uma única fonte. Um ataque distribuído de negação de serviço (DDoS) vem de múltiplas fontes, frequentemente em locais diferentes, tornando mais difícil identificar o(s) atacante(s).

# Guia de Análise do Log TCP Colorido no Wireshark

Existem duas abas na parte inferior do arquivo de log. Uma delas está rotulada como “Color coded TCP log” (Log TCP codificado por cores). Se você clicar nessa aba, encontrará as interações do servidor com o endereço IP do atacante (203.0.113.0) marcadas com destaque vermelho (e a palavra “red” na coluna A).

| Cor   | No. | Time     | Origem       | Destino      | Protocolo | Info                              |
|-------|------|----------|--------------|--------------|-----------|----------------------------------|
| red   | 52   | 3.390692 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red   | 53   | 3.441926 | 192.0.2.1    | 203.0.113.0  | TCP       | 443->54770 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| red   | 54   | 3.493160 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [ACK Seq=1 Win=5792 Len=0... |
| green | 55   | 3.544394 | 198.51.100.14| 192.0.2.1    | TCP       | 14785->443 [SYN] Seq=0 Win-5792 Len=120... |
| green | 56   | 3.599628 | 192.0.2.1    | 198.51.100.14| TCP       | 443->14785 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| red   | 57   | 3.664863 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 58   | 3.730097 | 198.51.100.14| 192.0.2.1    | TCP       | 14785->443 [ACK] Seq=1 Win-5792 Len=120... |
| red   | 59   | 3.795332 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win-5792 Len=120... |
| green | 60   | 3.860567 | 198.51.100.14| 192.0.2.1    | HTTP      | GET /sales.html HTTP/1.1          |
| red   | 61   | 3.939499 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win-5792 Len=120... |
| green | 62   | 4.018431 | 192.0.2.1    | 198.51.100.14| HTTP      | HTTP/1.1 200 OK (text/html)       |

Inicialmente, a solicitação SYN do atacante é respondida normalmente pelo servidor web (itens do log 52-54). No entanto, o atacante continua enviando mais solicitações SYN, o que é anormal. Neste ponto, o servidor web ainda consegue responder ao tráfego normal dos visitantes, que está destacado e rotulado como verde.

Um visitante funcionário com o endereço IP 198.51.100.14 completa com sucesso o handshake SYN/ACK com o servidor web (itens do log 55, 56, 58). Em seguida, o navegador do funcionário solicita a página sales.html com o comando GET e o servidor responde (itens do log 60 e 62).

| Cor   | No. | Time     | Origem       | Destino      | Protocolo | Info                              |
|-------|------|----------|--------------|--------------|-----------|----------------------------------|
| green | 63   | 4.097363 | 198.51.100.5 | 192.0.2.1    | TCP       | 33638->443 [SYN] Seq=0 Win-5792 Len=120... |
| red   | 64   | 4.176295 | 192.0.2.1    | 203.0.113.0  | TCP       | 443->54770 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| green | 65   | 4.255227 | 192.0.2.1    | 198.51.100.5 | TCP       | 443->33638 [SYN, ACK] Seq=0 Win-5792 Len=120... |
| red   | 66   | 4.256159 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 67   | 5.235091 | 198.51.100.5 | 192.0.2.1    | TCP       | 33638->443 [ACK] Seq=1 Win-5792 Len=120... |
| red   | 68   | 5.236023 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 69   | 5.236955 | 198.51.100.16| 192.0.2.1    | TCP       | 32641->443 [SYN] Seq=0 Win-5792 Len=120... |
| red   | 70   | 5.237887 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 71   | 6.228728 | 198.51.100.5 | 192.0.2.1    | HTTP      | GET /sales.html HTTP/1.1          |
| red   | 72   | 6.229638 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| yellow| 73   | 6.230548 | 192.0.2.1    | 198.51.100.16| TCP       | 443->32641 [RST, ACK] Seq=0 Win-5792 Len=120... |
| red   | 74   | 6.330539 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 75   | 6.330885 | 198.51.100.7 | 192.0.2.1    | TCP       | 42584->443 [SYN] Seq=0 Win=5792 Len=0... |
| red   | 76   | 6.331231 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| yellow| 77   | 7.330577 | 192.0.2.1    | 198.51.100.5 | TCP       | HTTP/1.1 504 Gateway Time-out (text/html) |
| red   | 78   | 7.331323 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| green | 79   | 7.340768 | 198.51.100.22| 192.0.2.1    | TCP       | 6345->443 [SYN] Seq=0 Win=5792 Len=0... |
| yellow| 80   | 7.340773 | 192.0.2.1    | 198.51.100.7 | TCP       | 443->42584 [RST, ACK] Seq=1 Win-5792 Len=120... |
| red   | 81   | 7.340778 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red   | 82   | 7.340783 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red   | 82   | 7.340783 | 192.0.2.1    | 203.0.113.0  | TCP       | 443->54770 [RST, ACK] Seq=1 Win=5792 Len=0... |

Nas próximas 20 linhas, o log começa a refletir a dificuldade que o servidor web está tendo para acompanhar o número anormal de solicitações SYN que chegam em ritmo acelerado. O atacante está enviando várias solicitações SYN por segundo. As linhas destacadas e rotuladas em amarelo representam comunicações falhas entre visitantes legítimos do site e o servidor web.

### Dois tipos de erros nos logs incluem:

- Uma mensagem de erro HTTP/1.1 504 Gateway Time-out (text/html). Esta mensagem é gerada por um servidor gateway que estava aguardando uma resposta do servidor web. Se o servidor web demorar muito para responder, o servidor gateway enviará uma mensagem de erro de tempo esgotado para o navegador solicitante.
- Um pacote [RST, ACK], que seria enviado ao visitante solicitante se o pacote [SYN, ACK] não for recebido pelo servidor web. RST significa reset, acknowledge (reiniciar, reconhecer). O visitante receberá uma mensagem de erro de tempo esgotado no navegador e a tentativa de conexão será descartada. O visitante pode atualizar o navegador para tentar enviar uma nova solicitação SYN.

### Exemplo de entradas do log:

| Cor    | No.  | Time      | Origem       | Destino      | Protocolo | Info                              |
|--------|-------|-----------|--------------|--------------|-----------|----------------------------------|
| red    | 119   | 19.198705 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 120   | 19.521718 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| yellow | 121   | 19.844731 | 192.0.2.1    | 198.51.100.9 | TCP       | 443->4631 [RST, ACK] Seq=1 Win=5792 Len=0... |
| red    | 122   | 20.167744 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 123   | 20.490757 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 124   | 20.813770 | 192.0.2.1    | 203.0.113.0  | TCP       | 443->54770 [RST, ACK] Seq=1 Win=5792 Len=0... |
| red    | 125   | 21.136783 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 126   | 21.459796 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 127   | 21.782809 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 128   | 22.105822 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 129   | 22.428835 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 130   | 22.751848 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 131   | 23.074861 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 132   | 23.397874 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 133   | 23.720887 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 134   | 24.043900 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 135   | 24.366913 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 136   | 24.689926 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 137   | 25.012939 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red    | 138   | 25.335952 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 129   | 22.428835 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 130   | 22.751848 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 131   | 23.074861 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 132   | 23.397874 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 133   | 23.720887 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 134   | 24.043900 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 135   | 24.366913 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 136   | 24.689926 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 137   | 25.012939 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 138   | 25.335952 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 139   | 25.658965 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 140   | 25.981978 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 141   | 26.304991 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 142   | 26.628004 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 143   | 26.951017 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 144   | 27.274030 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 145   | 27.597043 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 146   | 27.920056 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 147   | 28.243069 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 148   | 28.566082 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 149   | 28.889095 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 150   | 29.212108 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 151   | 29.535121 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |
| red  | 152   | 29.858134 | 203.0.113.0  | 192.0.2.1    | TCP       | 54770->443 [SYN] Seq=0 Win=5792 Len=0... |

À medida que você percorre o restante do log, notará que o servidor web para de responder ao tráfego legítimo dos visitantes funcionários. Os visitantes recebem mais mensagens de erro indicando que não conseguem estabelecer ou manter uma conexão com o servidor web. A partir do item de log número 125, o servidor web para de responder. Os únicos itens registrados nesse ponto são do ataque.

Como há apenas um endereço IP atacando o servidor web, você pode assumir que este é um ataque direto de negação de serviço (DoS) do tipo inundação SYN (SYN flood).
