---
title: "Cybersecurity DoS DDoS UDP Flood"
date: 2024-05-09T00:38:46-03:00
tags: ["linux","openbsd","freebsd","ipfw","pf","nftables","iptables","cybersecurity","ddos","dos","engineering","firewall"]
author: "0xttfx"
toc: true
hideSummary: false
comments: true
#showComments: true
showDate: true
showTitle: true
showShare: true
#disableShare: false
norss: false
nosearch: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
#UseHugoToc: true



---

Um ataque DDoS UDP flood é uma forma de ataque distribuído de negação de serviço que explora serviços vulneráveis na Internet que utilizam o protocolo UDP - User Datagram Protocol para inundar um alvo com uma grande quantidade de tráfego de pacotes UDP. E como cada novo pacote UDP recebido pelo servidor, consome recursos computacionais, rapidamente o servidor vai consumindo seus recursos devido a sobrecarga até exaurir o poder computacional o impedindo de processar e responder requisições normais dos usuários, ficando indisponível.

![UDPflood.drawio](/images/DDoSUDPflood/UDPflood.drawio.svg)

### Funcionamento

1. **Inundação UDP**: o atacante fazendo uso de uma botnet envia uma grande quantidade de pacotes UDP contra um alvo.

2. **IP Spoofing**: o endereço IP de origem dos pacotes UDP são falsificados para impedir a identificação da origem do ataque.

3. **Amplificação de tráfego**: também é aplicado a técnica de amplificação para aumentar o impacto do ataque. Dependendo do serviço vulnerável na Internet, usado pelo atacante, uma pequena requisição UDP irá gerar um resposta com grande quantidade de dados que será encaminhado para o alvo, aumentando ainda mais a sobrecarga na infraestrutura.

4. **Sobrecarga dos recursos**: devido a arquitetura do protocolo, a infraestrutura do alvo fica rapidamente sobrecarregada com a quantidade massiva de pacotes UDP recebidos, gerando assim indisponibilidade do serviço para usuários legítimos.

### Características

- **Poder do ataque**: o ataque é relativamente simples de ser realizado e requer poucos recursos do lado do atacante, já que é apenas uma questão de enviar uma grande quantidade de pacotes falsificados.

- **Dificuldade de Rastreamento**: devido ao IP spoofing, rastrear a origem do ataque e aplicar mitigação direcionada fica bem difícil.

- **Impacto na disponibilidade**: é o objetivo do ataque torná-lo inacessível para usuários legítimos.

### Mitigação

- **Filtragem de pacotes**: implementar filtros de pacotes para bloquear ou limitar o tráfego UDP suspeito.
- **Detecção de Amplificação**: identificar e corrigir vulnerabilidades em serviços que podem ser explorados para amplificar tráfego.
- **Implementação de limites de taxa**: configurar limites de taxa para o tráfego UDP.
- **Análise de tráfego**: monitorar o tráfego de rede para identificar padrões suspeitos de tráfego UDP para medidas de mitigação proativas.
- **Serviços de proteção DDoS**: considerar o uso de serviços de proteção DDoS de provedores CDN e segurança que podem detectar e mitigar ataques em tempo real.
- **Atualização e patches de segurança**: manter sistemas e serviços atualizados com as últimas correções de segurança para mitigar vulnerabilidades conhecidas que podem ser exploradas em ataques.

Os ataques DDoS UDP flood representam uma ameaça significativa para a disponibilidade de serviços online e podem ser difíceis de mitigar devido à sua natureza distribuída e à facilidade de execução. A implementação de uma combinação de técnicas de mitigação ainda é a melhor forma de reduzir o impacto desses ataques e manter a disponibilidade dos serviços.

#### Firewalls.
Algumas técnicas possíveis

##### nftables

###### Limitação de taxa de pacotes
Limitar a taxa de pacotes UDP por origem ou destino reduz a intensidade do ataque.

```bash
nft add table ip filter
nft add chain ip filter input { type filter hook input priority 0 \; }
nft add rule ip filter input udp limit rate 50/second burst 100 packets accept
nft add rule ip filter input udp limit rate over 50/second drop
```

- `add table ip filter`:  cria uma [tabela](https://wiki.nftables.org/wiki-nftables/index.php/Configuring_tables#Adding_tables) chamada "filter" de [família de endereço](https://wiki.nftables.org/wiki-nftables/index.php/Nftables_families) "ip".
- `add chain ip filter input { type filter hook input priority 0 \; }`: adiciona a [cadeia](https://wiki.nftables.org/wiki-nftables/index.php/Configuring_chains#Adding_base_chains) base "input" na tabela "filter" fazemos um filtro no [gancho input](https://wiki.nftables.org/wiki-nftables/index.php/Netfilter_hooks) com [prioridade](https://wiki.nftables.org/wiki-nftables/index.php/Configuring_chains#Base_chain_priority) 0.
- `add rule ip filter input ip protocol udp limit rate 50/second burst 100 packets accept`: serão aceitos pacotes com taxa abaixo de 50 por segundo com um burst(pico) de 100 pacotes.
- `add rule ip filter input ip protocol udp limit rate over 50/second burst 100 packets drop`: pacotes acima do limite de taxa serão descartados.

###### Filtragem de portas e pacotes
 Bloquear portas incomuns.

```bash
nft add rule ip filter input ip protocol udp dport 0-1023 drop
```

- `udp dport 0-1023 drop`: descarta pacotes destinados ao intervalo de portas (aqui como exemplo usamos as reservadas).

###### Rastreamento de conexão
O rastreamento de conexão ajuda a identificar e bloquear pacotes que não fazem parte de um padrão esperado. Apesar do UTP não ser orientado a conexão, o sistema "ct" trata a transação como uma única conexão rastreada baseada na origem+destino de endereço IP e portas enquanto ela passa pelas funções hook do ct. Por exemplo: o comando host dispara uma query para um servidor DNS criando o "ctinfo" "[IP_CT_NEW](https://github.com/torvalds/linux/blob/master/include/uapi/linux/netfilter/nf_conntrack_common.h#L17)"! E e quando o pacotes alcança a função hook "[help+config](https://github.com/torvalds/linux/blob/master/net/netfilter/nf_conntrack_core.c#L1151)" o status é "[IPS_CONFIRMED](https://github.com/torvalds/linux/blob/master/net/netfilter/nf_conntrack_core.c#L1214)" e o timeout de 30s e o rastreamento da conexão é adicionado na tabela ct... Quando o servidor DNS responde a query o sistema reconhece que as características desses pacotes coincidem com a conexão salva na tabela e que ainda está dentro do timeout configurado e então o "ctinfo" é configurado para "IP_CT_ESTABLISHED_REPLY" e o "status" para "[IPS_SEEN_REPLY](https://github.com/torvalds/linux/blob/master/include/uapi/linux/netfilter/nf_conntrack_common.h#L48)" e o timeout é configurado para 30s novamente e se não ocorrer mais nenhum pacote de ambos os hosts, o "status" é configurado para  "[IPS_DYING](https://github.com/torvalds/linux/blob/master/include/uapi/linux/netfilter/nf_conntrack_common.h#L85)"

```bash
nft add rule ip filter input ip protocol udp ct state new,established accept
nft add rule ip filter input ip protocol udp ct state invalid drop
```

- `udp ct state new,established accept`: aceita conexões novas e estabelecidas.
- `udp ct state invalid drop`: descarta pacotes considerados inválidos.

###### Filtragem de pacotes anômalos e fragmentados
Pacotes com cabeçalhos ou tamanho inconsistentes podem indicar um ataque.

```bash
nft add rule ip filter input ip protocol udp ip length 0 drop
nft add rule ip filter input ip protocol udp ip length 512-65535 drop
nft add rule ip filter input ip frag-off != 0 ip protocol udp drop
```

- `udp ip length 0 drop`: descarta pacotes com tamanho zero.
- `udp ip length 512-65535 drop`: descarta pacotes com tamanho acima de 512 bytes.
- `ip frag-off != 0 ip protocol udp drop`: descartar pacotes IP fragmentados para evitar a recomposição de pacotes maliciosos.

###### Filtragem de redes e IP suspeitos
Se você identificar fontes de ataques conhecidas, pode criar listas de bloqueio para impedir tráfego desses endereços.

```bash
nft add set ip filter block_ips {type ipv4_address \; timeout 6h \; comment \"descarta pacotes desses sources\" \; }
nft add rule ip filter input ip saddr @block_ips drop
nft add element ip filter block_ips { ?.?.?.?, ?.?.?.?/? }
```

- `add set ip filter block_ips {type ipv4_address \;timeout 6h \;}`: cria o [conjunto](https://wiki.nftables.org/wiki-nftables/index.php/Sets#Named_sets) "block_ips" do tipo "ipv4_address" na tabela "filter" com para armazenar endereços IP e que serão removidos após 6 horas.
- `add rule ip saddr @block_ips drop`: aplica a regra a pacotes cujo endereço de origem esteja no conjunto "blocked_ips" descartando-os.
- `add element ip filter block_ips { ?.?.?.?, ?.?.?.?/? }`: adicionando IP's e blocos CIDR ao conjunto "block_ips".

##### ipfw

###### Habilitar e limitar o  log
Limitar registros de logs de regras para minimizar ocupação de recursos.

```bash
sysrc firewall_logging="YES"
echo "net.inet.ip.fw.verbose_limit=6" >> /etc/sysctl.conf
```

-  `firewall_logging="YES"`: habilitando log
- `"net.inet.ip.fw.verbose_limit=6"`: limitando o número de vezes que a regra é logada por tentativa de conexão.

###### Limitação de taxa de pacotes
Limitar a taxa de pacotes UDP por origem ou destino reduz a intensidade do ataque.

```shell
ipfw add 090 check-state
ipfw add 100 deny limit src-addr 100/s log udp from any to any
```

- `check-state`: verifica o estado das conexões para permitir tráfego de conexões estabelecidas ou relacionadas.
- `deny limit src-addr 100/s log udp from any to any`: limita a 100 pacotes por segundo por endereço IP e rejeita pacotes excedentes.

###### Filtragem de portas
Bloquear portas incomuns.

```shell
ipfw add 200 deny log udp from any to any dst-port 0-1234
```

- `deny log udp from any to any dst-port 0-1234`: loga e bloqueia intervalo de portas de destino (aqui como exemplo usamos as reservadas).

###### Rastreamento de estado
O rastreamento de estado permite que o firewall diferencie conexões legítimas de atividades suspeitas.

```shell
ipfw add 300 allow udp from any to any keep-state
```

- `allow udp from any to any keep-state`: cria regra UDP dinâmica que corresponde ao tráfego bidirecional entre os endereços e portas de origem e destino para permitir tráfego UDP relacionado.

###### Filtragem de pacotes anômalos e fragmentados
Pacotes com cabeçalhos ou tamanho inconsistentes podem indicar um ataque.

```bash
ipfw add 400 deny udp from any to any frag
ipfw add 500 deny udp from any to any iplen 512-65535
```

- `add 400 deny udp from any to any frag`: nega todos os pacotes fragmentados para o protocolo UDP.
- `deny udp from any to any iplen 512-65535`: descartar pacotes com tamanho maior que 512 bytes, indicando amplificação ou fragmentação maliciosa.
###### Uso de Tabelas para Bloquear Endereços IP
Outra técnica é usar tabelas para bloquear endereços IP suspeitos.

```shell
ipfw table 1 create type addr
ipfw table 1 add ?.?.?.?/?
ipfw add 600 deny all from table(1) to any
```

- `ipfw table 1 create type addr`: cria tabela para armazenar endereços IP que podem ser usados para rastreio.
- `ipfw table 1 add ?.?.?.?/?`: adiciona IP ou bloco de endereços à tabela para bloqueio.
- `ipfw add 600 deny all from table(1) to any`: Bloqueia pacotes de endereços IP que estão na tabela "1".

##### PF

###### Limitação de taxa de pacotes
Limitar a taxa de pacotes UDP por origem ou destino reduz a intensidade do ataque.

```shell
table block_ip persist
pass in proto udp from any to any port any keep state (max 10, max-src-conn-rate 5/second, overload block_ip flush)
block in quick from block_ip
```

- `table block_ip persist`: cria uma tabela para armazenar endereços IP que não respeitaram o limite imposto pela regra.
- `pass in proto udp from any to any port any keep state (max 10, max-src-conn-rate 5/second, overload block_ip flush)`:  mantem o estado de até 10 conexões por endereço IP, limitando a taxa de conexões para 5 por segundo e quando o limite é excedido, o endereço é adicionado na tabela `block_ip` além de todas as conexões desse endereços serem removidas.
- `block in quick from block_ip`: Bloqueia rapidamente qualquer tráfego de endereços na tabela `<blocked_ips>`. A opção `quick` garante que a regra seja aplicada imediatamente, sem continuar verificando regras subsequentes.

###### Rastreamento de estado
O controle de estados permite que o firewall mantenha o rastreamento das conexões ativas, permitindo identificar tráfego anômalo. Você pode definir regras para permitir apenas tráfego com um estado válido.

```shell
pass in proto udp from any to any keep state
block in proto udp from any to any keep state (no-state)
```

- `pass out proto udp from any to any keep state`: permite tráfego que faz parte de conexões estabelecidas ou relacionadas.
- `block in proto udp from any to any keep state (no-state)`: bloqueia tráfego que não pertence a conexões válidas

###### Bloqueio de IP
Podemos criar uma nova tabela ou fazer uso da tabela já criada para bloquear IPs suspeitos.

```shell
# Adiciona um endereço IP à tabela
table block_ip add ?.?.?.?
```

- `table block_ip add ?.?.?.?`: adiciona um endereço IP na tabela block_ip para bloqueio.

###### Filtragem por Portas
Outra técnica é bloquear portas específicas que possam ser usadas para ataques ou analisar o conteúdo dos pacotes para detectar padrões suspeitos.

```shell
block in proto udp from any to any port 53
```

- `block in proto udp from any to any port 53`: bloqueia pacotes UDP destinados à portas ou range de portas específicas.

###### Filtragem de pacotes anômalos e fragmentados
Pacotes com características anormais ou malformados ou tamanho inconsistentes podem indicar um ataque.

```bash
block in quick proto udp from any to any port any tagged DOIDAO_UDP_SIZE iplen 512-65535
block in quick proto udp from any to any frag
```

- `tagged DOIDAO_UDP_SIZE iplen 512-65535`: descartar pacotes com tamanho maior que 512 bytes, que indicam também pacotes amplificados.
- `block in quick proto udp from any to any frag`: descarta pacotes fragmentados para prevenir amplificação por fragmentação.

##### iptables
###### Limitar a taxa de pacotes
Limitar a taxa de pacotes UDP por origem ou destino reduz a intensidade do ataque.

```bash
iptables -A INPUT -p udp -m limit --limit 100/sec --limit-burst 200 -j ACCEPT
```

- `-p udp`: especifica pacotes UDP.
- `-m limit`: módulo de limitação de taxa.
- `--limit 100/sec`: limita a 100 pacotes por segundo.
- `--limit-burst 200`: permite um pico (burst) de até 200 pacotes antes de aplicar a limitação.
- `-j ACCEPT`: Aceita pacotes que atendem a esse critério.

###### Bloquear de portas
Bloquear portas incomuns.

```bash
iptables -A INPUT -p udp --dport 0:1023 -j DROP
```

- `--dport 0:1023`: intervalo de portas de destino (aqui como exemplo usamos as reservadas).
- `-j DROP`: Descarte pacotes que atendem a esse critério.

###### Rastreamento de conexão
O rastreamento de conexão ajuda a identificar e bloquear pacotes que não fazem parte de um padrão esperado.

```bash
iptables -A INPUT -p udp -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p udp -m state --state INVALID -j DROP
```

- `-m state`: módulo de estado para rastreamento de conexão.
- `--state NEW,ESTABLISHED`: conexões novas e estabelecidas..
- `-j ACCEPT`: aceita pacotes que atendem a esse critério.
- `--state INVALID`: descartar pacotes considerados inválidos ou não solicitados

###### Filtragem de pacotes anômalos e fragmentados
Pacotes com cabeçalhos inconsistentes podem indicar um ataque.

```bash
iptables -A INPUT -p udp --length 0 -j DROP
iptables -A INPUT -p udp -m length --length 512-65535 -j DROP
iptables -A INPUT -p udp -f -j DROP
```

- `--length 0 -j DROP`: descarta pacotes com comprimento zero, indicando anomalia.
- `-m length --length 512-65535 -j DROP`: descarta pacotes com tamanho maior que 512 bytes, indicando pacotes amplificados ou malformados.
- `-f -j DROP`: filtra pacotes fragmentados para evitar recomposição.

###### Filtragem de redes e IP suspeitos
Se você identificar fontes de ataques conhecidas, pode criar listas de bloqueio para impedir tráfego desses endereços ou blocos de endereço.

```bash
iptables -A INPUT -s 0.0.0.0/0 -j DROP
```

- `0.0.0.0/0`: endereço ou blocos de endereço (CIDR)
- `-s`: Define a origem do pacote.
- `-j DROP`: descarte pacotes dessa origem.

##### Considerações

- Antes de aplicar essas regras em um ambiente de produção, teste cuidadosamente para garantir que não afetem tráfegos legítimos da infraestrutura.
- Monitore seu sistema após aplicar novas regras para detectar comportamentos inesperados ou interrupções.
- A combinação dessas técnicas ajuda a criar uma defesa robusta contra ataques DDoS UDP flood e manter a disponibilidade do serviço mesmo diante de tráfego malicioso

