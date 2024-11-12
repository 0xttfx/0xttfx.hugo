---
title: "RTFM & Aleatoriedades para SysAdmin, SecOps, DevOps, {SN}RE - Firewall"
date: 2024-11-12T18:07:52-03:00
# weight: 1
# aliases: ["/first"]
tags: ["PF", "OpenBSD", "Firewall", "DoS", "DDoS", "SegInfo"]
author: "0xttfx"
# author: [0xttfx] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Requisitos mínimos para um firewall tecnicamente descente e profissional."
#canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---
# Firewall Policy

Em 2016 eu fiz participei do curso [FreeBSD S.S.A](https://www.freebsdbrasil.com.br/treinamento/treinamentos-ssa.html) na [FreeBSD brasil](https://www.freebsdbrasil.com.br) que sem sombra de dúvidas foi um dos **raros treinamentos** que de fato entregou conhecimento e me fez avançar de diversas formas no gerenciamento de redes, roteamento e segurança complexos de forma KISS.

>[!TIP]
>SSA - Server and System Systems Administration
>December 2016
>FreeBSD Brasil LTDA
>[Certificado Número: 011245](https://www.freebsdbrasil.com.br/treinamento/certificacao-validar-certificados.html)

Como reflexo reflexo do conhecimento adquirido. A atenção aos requisitos mínimos, necessários para conceber um firewall tecnicamente descente e profissional: tornaram-se mandatórios! E por isso, segue como sugestão, uma lista mandatória de requisitos mínimos que precisam ser atendidos:
 
- tratar tráfego loopback e pseudo interfaces,
- tratar anomalias IP e por protocolo,
- criar limites de fluxo IP,
- tratar transformações e validações de pacotes,
- criar políticas por protocolo, serviço, host e redes,
- aplicar técnicas para mitigar DoS e DDoS,
- considerar o esgotamento de recursos computacionais.
- estar alinhado às diretrizes do "NIST SP 800-41 Revision 1 - Guidelines on Firewalls and Firewall Policy".

>[!NOTE]
> Ambiente de teste:
>- Aplicações
>	- httpd daemon,
>	- OpenSSH,
>	- Wireguard,
>	- Prometheus Node Exporter,
>	- Tor relay obfs4 bridge.
>- servidor
>	- VPS [VULTR](https://www.vultr.com)
>		- Regular Cloud Compute.
>		- 1 vCPU, 1024 MB RAM, 25 GB SSD, 1.00 TB Transfer.
>		- OpenBSD 7.4 (GENERIC.MP) \#1396: Sun Oct 8 09:20:40 MDT 2023.


### Ruleset
Segue abaixo, uma  sugestão de conjunto de regras de firewall  [_Open_**BSD**](https://www.openbsd.org) PF - Packet Filtering.

- /etc/pf.conf
``` bash
# Definições de interfaces
ext_if = "vio0"            # Interface externa (WAN)
int_if = "vio1"            # Interface interna (LAN)
lo_if = "lo0"       # Interface loopback
wg_if = "wg0"             # Interface VPN WireGuard

# Endereços IP e redes
server_ip = "215.230.100.124"  # IP do servidor de aplicação
lan_net = "10.42.95.0/20"   # Rede interna (LAN)
vpn_net = "10.0.0.0/24"      # Rede da VPN WireGuard
srv_monit_ip = "0.0.0.0"# IP do servidor de monitoramento

# Portas Permitidas (HTTP, HTTPS, SSH, WireGuard, Prometheus)
allowed_ports_tcp = "{ 22, 80, 443, 9100, 51820, 9001, 9003 }"  # 51820 é a porta padrão do WireGuard
allowed_ports_udp = "{ domain }"

# ICMP
# https://man.openbsd.org/icmp
icmp_types = "{ echoreq, unreach, timex, trace }"

#
non_routable = "{ 127.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8, 169.254.0.0/16, 192.0.2.0/24, 0.0.0.0/8, 240.0.0.0/4 }"

# LOG
pflog_logfile="/var/log/pflog"

# Criando tabelas
# https://www.openbsd.org/faq/pf/tables.html
table <trusted_admins> persist file "/etc/trusted_admins"
table <blocked_hosts> persist
table <blocked_bforce> persist

# Política de Bloqueio Padrão e de Limites de Estado e Fragmentação
# Bloqueio de todo o tráfego por padrão, com permissões específicas para tráfego necessário.
# https://www.openbsd.org/faq/pf/options.html
# Política de segurança mínima - "deny all". Alinhado com o NIST SP 800-41 Revision 1, Seção 4 - Firewall Policy 
set block-policy drop        # Define o comportamento padrão para regras de filtro: drop - packet is silently dropped
set loginterface $ext_if     # Loga tráfego na interface externa para auditoria
set skip on $lo_if           # Ignora tráfego na interface loopback para desempenho

# Limita o número de estados de conexão e fragmentos permitidos para prevenir ataques DoS.
# NIST SP 800-41, Seção 4.4 - Policies Based on Network Activity
# https://www.openbsd.org/faq/pf/options.html
# Estabelendo limites na operação do Pf. Configurações atuais: `pfctl -s memory`.
set limit { states 10000, frags 5000 }  # frags - número máximo de entradas no pool de memória usado para remontagem de pacotes (scrub rules). O padrão é de 5000. states - número máximo de entradas no pool de memória usado para entradas de tabela de estado (regras de filtro que especificam o keep state). O padrão é 100000.
set optimization normal  # Configuração otimizada para tráfego em ambiente de rede: normal - adequado para quase todas as redes.

# Alinhado com a prática de "deny-all" na entrada e saída: exceto o permitido explicitamente.
# https://www.openbsd.org/faq/pf/tagging.html#policy
# https://www.openbsd.org/faq/pf/filter.html#defdeny
# Conforme NIST SP 800-41, Seção 4.1 (Política padrão de bloqueio)
block in log all  # bloqueará o tráfego de entrada em todas as interfaces em qualquer direção de qualquer lugar para qualquer lugar.
block out log all # bloqueará o tráfego de saida em todas as interfaces em qualquer direção de qualquer lugar para qualquer lugar.

# Prevenção de Anomalias IP
# Previne spoofing e anomalias de IP usando a diretiva `antispoof`.
# https://www.openbsd.org/faq/pf/filter.html#antispoof
# Conforme NIST SP 800-41, seção 4.1.1 - IP Addresses and Other IP Characteristics
antispoof quick for { $ext_if $int_if $wg_if }

# Bloqueio de endereços IP não roteáveis.
# Conforme NIST SP 800-41, Seção 4.1.1 - IP Addresses and Other IP Characteristics
block drop in quick on $ext_if from any to $non_routable    # Bloqueia loopback na interface externa
block drop in quick on $ext_if from any to 255.255.255.255  # Bloqueia broadcast global

# Loopback e Pseudo-Interfaces. Permitindo tráfego local na interface loopback para evitar tretas.
# Alinhamento com NIST SP 800-41, Seção 4.1 - Policies Based on IP Addresses and Protocols
pass quick on $lo_if

# Validação e Transformação de Pacotes (scrubbing)
# Reassembla pacotes fragmentados e randomiza IDs de pacotes para prevenir ataques que exploram fragmentação.
# https://home.nuug.no/~peter/pf/en/scrub.html
# https://man.openbsd.org/pf.conf.5#TRAFFIC_NORMALnoISATION
# https://man.openbsd.org/pf.conf#match
# Alinhamento com NIST SP 800-41, seção 2.1.1 - Packet Filtering 
match in all scrub (no-df max-mss 1440)
match out all scrub (random-id) # randomização de IDs para dificultar fingerprinting de pacotes

# Permissão explícita para serviços
# https://www.openbsd.org/faq/pf/filter.html#stateopts
# Alinhamento com NIST SP 800-41, seção 4.2 - Policies Based on Applications e 2.1.1 - Packet Filtering
# https://www.openbsd.org/faq/pf/filter.html#stateopts
# https://www.openbsd.org/faq/pf/filter.html#synproxy
# bloqueando sources suspeitos
# https://www.openbsd.org/faq/pf/tables.html
block in quick from <blocked_hosts> to any
block in quick from <blocked_bforce> to any

# Limitar o tráfego ICMP
# Alinhamento com NIST SP 800-41, seção 4.1.4 - ICMP e 4.1.2 IPv6 e 4.4 - Policies Based on Network Activity
# se os pacotes exceder 1000 por 10s, pacotes adicionais não serão processados bem como se houver mais de 1000 entradas de estados. E também, se existir mais de 100 IPs exclusivos, ou se um único IP tiver mais de 100 entradas...
# ICMP para IPv4
pass in log on $ext_if inet proto icmp all from any to $ext_if icmp-type $icmp_types max-pkt-rate 1000/10 keep state (max 1000, source-track rule, max-src-nodes 100, max-src-states 100)
# ICMPv6 para IPv6
#pass in log on $ext_if inet proto icmp all from any to $ext_if inet6 proto icmp6 max-pkt-rate 1000/10 keep state (max 1000, source-track rule, max-src-nodes 100, max-src-states 100)

# "synproxy" para proteger contra ataques SYN Flood ao validar pacotes antes de enviar ao servidor.
# Alinhamento com NIST SP 800-41, seção 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP
# https://www.openbsd.org/faq/pf/filter.html#stateopts
# https://www.openbsd.org/faq/pf/filter.html#synproxy
pass in log on $ext_if proto tcp from any to $ext_if port 80 flags S/SA synproxy state \
    (max-src-conn 10, max-src-conn-rate 10/5, \
     overload <blocked_hosts> flush)

# "synproxy" para proteger contra ataques SYN Flood ao validar pacotes antes de enviar ao servidor.
# Alinhamento com NIST SP 800-41, seção 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP
# https://www.openbsd.org/faq/pf/filter.html#stateopts
# https://www.openbsd.org/faq/pf/filter.html#synproxy
pass in log on $ext_if proto tcp from any to $ext_if port 443 flags S/SA synproxy state \
    (max-src-conn 10, max-src-conn-rate 10/5, \
    overload <blocked_hosts> flush)

# Limite de conexões para evitar ataques de brute force
# Alinhamento com NIST SP 800-41, seção 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP
# https://www.openbsd.org/faq/pf/filter.html#stateopts
pass in log on $ext_if proto tcp from any to $ext_if port 22 keep state \
    (max-src-conn 5, max-src-conn-rate 3/10, \
     overload <brute_force> flush global)

# Node Exporter porta 9100.
# Limite de conexões para evitar ataques de brute force
# Alinhamento com NIST SP 800-41, seção 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP
# https://www.openbsd.org/faq/pf/filter.html#stateopts
pass in log on $ext_if proto tcp from any to $ext_if port 9100 keep state \
    (max-src-conn 5, max-src-conn-rate 5/5, \
     overload <blocked_bforce> flush)

# Tor relay obfs4 bridge
# Permite o serviço Tor nas portas 9001 e 9003.
pass in on $ext_if proto tcp from any to $ext_if_ip port {9001, 9003} keep state

# Permissão de tráfego de saída apenas para serviços essenciais.
# Alinhado com o NIST SP 800-41, seção 4.1.1 - IP Addresses and Other IP Characteristics 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP
pass out quick on $ext_if proto tcp from $server_ip to any port $allowed_ports_tcp keep state
pass out quick on $ext_if proto udp from $server_ip to any port 51820 keep state  # Saída para WireGuard VPN
pass out quick on $ext_if proto { tcp udp } from ($ext_if) to any port 53 # dns resolver 
pass out quick on $ext_if inet proto icmp all icmp-type $icmp_types # icmp prove 

# Políticas para redes
# Define permissões e restrições para a rede interna e a rede externa (WAN).
# Alinhado com o NIST SP 800-41, 4.1 Policies Based on IP Addresses and Protocols
# Tráfego permitido entre a rede interna (LAN) e o servidor.
pass in quick on $int_if from $lan_net to any keep state

# Bloqueio de tráfego desnecessário da WAN.
block in on $ext_if from any to $lan_net
block out on $ext_if from $lan_net to any

# Monitoramento e Auditoria
# https://man.openbsd.org/pflog.4
# https://www.openbsd.org/faq/pf/logging.html
# https://man.openbsd.org/pf.conf#match
# Log de tráfego relevante para auditoria e detecção de incidentes, conforme recomendado pelo NIST SP 800-41, Seção 5.
match log (all) on $ext_if
```

### NIST Special Publications

Estou usando como referência principal, a coleção de recomendações do guia [Special Publication 800-41 Revision 1 - Guidelines on Firewalls and Firewall Policy](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-41r1.pdf) da subsérie de especificações técnicas [SP 800 - Computer security](https://csrc.nist.gov/publications/sp800) da  [**NIST Special Publications**](https://csrc.nist.gov/publications/sp) 

O conjunto de regras, tenta estar alinhado de acordo com as recomendações, parar garantir as melhores práticas para a segurança e controle de tráfego em um firewall PF em um servidor OpenBSD. E apesar de nem sempre existir uma seção do documento que faz referência direta ao método que está sendo aplicado em uma regra específica; ainda assim, eu tento sinalizar as seções que tratam de forma concreta ou abstrata... ;)

Por fim, conseguimos atender tecnicamente os seguintes pontos:

- Política de Bloqueio Padrão e Defesa em Profundidade
	- Todas as conexões de entrada e saída são bloqueadas por padrão para aplicar uma política de segurança mínima e alinhada ao conceito de "defesa em profundidade". O NIST recomenda permitir tráfego apenas quando explicitamente autorizado.
- Limites de Conexão e Fragmentação
	- Os limites de `states` e `frags` são configurados para evitar ataques DoS e DDoS, prevenindo o esgotamento de recursos no firewall e alinhando-se com o NIST sobre restrições de uso de estados. 
- Regras de Normalização e Scrubbing
	- A regra `scrub` assegura a reconstrução de pacotes fragmentados e previne anomalias de tráfego, alinhando-se com as recomendações do NIST sobre validação de pacotes.
- Prevenção de spoofing, syn flood e integridade de pacotes
	- O comando `antispoof` ajuda a bloquear pacotes forjados provenientes de IPs internos e `synproxy` evitando DoS syn flood  conforme o NIST recomenda a integridade de pacotes.
- Tráfego loopback
	- O tráfego loopback é permitido para garantir a comunicação interna do sistema, seguindo as diretrizes do NIST sobre a configuração de interfaces locais.
- Regras específicas por aplicação
	- Para SSH, HTTP/HTTPS, WireGuard, Prometheus e Tor, cada serviço possui regras de passagem individualizadas. O NIST recomenda políticas por serviço para controlar o tráfego conforme as necessidades do aplicativo. Cada regra possui limitação de taxa de conexões para prevenir ataques de força bruta e DoS.
- Limitação de conexões ICMP
	- Limita o número de requisições ICMP permitidas por host, ajudando a mitigar possíveis abusos e ataques de DoS via ICMP. A configuração separa IPv4 e IPv6 para garantir a compatibilidade.
- Controle e auditoria de tráfego com log
	- O `match log (all)` permite auditoria completa do tráfego de entrada e saída, alinhado ao NIST, que recomenda manter registros de incidentes para análise e conformidade.



