# Investigação de Incidente de Segurança: Ataque de Negação de Serviço (DoS)

## Sobre o projeto

Este repositório documenta a investigação de um incidente de segurança em uma agência de viagens fictícia, na qual o servidor web da empresa ficou indisponível devido a um grande volume de requisições TCP SYN provenientes de um endereço IP suspeito.

Como analista de segurança responsável pelo caso, o objetivo foi identificar a causa raiz da interrupção do serviço, analisar o tráfego de rede capturado, determinar o tipo de ataque envolvido e comunicar as descobertas e os próximos passos à gestão.

## Cenário

Uma agência de viagens divulga promoções de pacotes de viagem em seu site, usado diariamente pelos funcionários. Um alerta automático do sistema de monitoramento indicou falha no servidor web, e o acesso ao site passou a retornar erro de tempo limite de conexão (connection timeout). A análise com um sniffer de pacotes revelou um volume anormal de requisições TCP SYN vindas de um único IP desconhecido, sobrecarregando o servidor e impedindo respostas legítimas.

Como medidas imediatas, o servidor foi colocado temporariamente offline para recuperação, e o IP de origem foi bloqueado no firewall — uma solução reconhecidamente paliativa, já que o atacante pode falsificar (spoof) outros endereços IP para contornar o bloqueio.

## Estrutura do repositório e ordem de leitura sugerida

Os arquivos foram organizados para reproduzir o raciocínio investigativo: primeiro adquirir a capacidade de ler a evidência, depois examinar a evidência bruta, em seguida interpretá-la para identificar a causa e o impacto do ataque e, por fim, consolidar tudo em um relatório final voltado à gestão.

| Ordem | Arquivo | Propósito |
|---|---|---|
| 1     | [`how-to-read-a-wireshark-TCP-HTTP-log.md`](docs/how-to-read-a-wireshark-TCP-HTTP-log.md) | Guia de referência para interpretar os campos e padrões de um log do Wireshark capturando tráfego TCP/HTTP — necessário antes de examinar o log real. |
| 2 | [`wireshark-TCP-HTTP-log.xlsx`](docs/wireshark-TCP-HTTP-log.xlsx) | A evidência bruta: o log de pacotes capturado durante o incidente, mostrando o volume anômalo de requisições SYN. |
| 3 | [`analysis-network-attacs.md`](result/analysis-network-attacs.md) | Análise e solução do cenário: a partir da evidência do log, identifica o tipo de ataque de rede envolvido (TCP SYN flood) e explica como ele ocorreu e qual foi seu impacto negativo no site.
| 4 | [`cybersecurity-incident-report.md`](result/cybersecurity-incident-report.md) | Relatório final do incidente, consolidando a análise, a causa identificada, o impacto no negócio e as recomendações para a gestão.

## Por que essa ordem?

- O guia de leitura vem primeiro porque é um pré-requisito de metodologia: ensina a interpretar o log antes de ele ser apresentado.
- O log do Wireshark é a evidência primária do incidente e vem em seguida, como matéria-prima da investigação.
analysis-network-attacks.md é a análise/solução propriamente dita: usa a evidência do log para identificar o tipo de ataque, explicar seu funcionamento e seu impacto — por isso vem depois do log, não antes.
- O relatório de incidente é o entregável final, que referencia e sintetiza os três arquivos anteriores — por isso fica por último.

## Habilidades demonstradas

- Análise de tráfego de rede e captura de pacotes (Wireshark)
- Identificação de padrões de ataque (TCP SYN flood / DoS)
- Investigação e resposta a incidentes de segurança
- Comunicação técnica de achados para stakeholders não técnicos

## Aviso

Este projeto é baseado em um cenário educacional (adaptado do Google Cybersecurity Certificate) e tem finalidade exclusivamente de portfólio/estudo.