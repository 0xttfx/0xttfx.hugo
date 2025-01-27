---
title: "RTFM & Aleatoriedades - Firewall"
date: 2024-11-12T18:07:52-03:00
# weight: 1
# aliases: ["/first"]
tags: ["PF", "OpenBSD", "Firewall", "DoS", "DDoS", "SegInfo"]
toc: true
author: "Faioli a.k.a 0xttfx"
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Requisitos mínimos para um firewall profissional."
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

## Firewall

Em 2016 eu participei do curso [FreeBSD S.S.A](https://www.freebsdbrasil.com.br/treinamento/treinamentos-ssa.html) na [FreeBSD brasil](https://www.freebsdbrasil.com.br) que sem sombra de dúvidas foi um dos **raros treinamentos** que de fato entregou conhecimento e me fez avançar de diversas formas no gerenciamento de redes, roteamento e segurança complexos de forma KISS.

>[!TIP]
>SSA - Server and System Systems Administration.
>December 2016.
>FreeBSD Brasil LTDA.
>[Certificado Número: 011245](https://www.freebsdbrasil.com.br/treinamento/certificacao-validar-certificados.html).

Como reflexo do conhecimento adquirido. A atenção aos requisitos mínimos, necessários para conceber um firewall tecnicamente descente, profissional e em conformidade com os requisitos do projeto: tornaram-se mandatórios!

E para esse fim, como apoio e referência

- [NIST SP 800-41 Rev 1 - Guidelines on Firewalls and Firewall Policy](https://csrc.nist.gov/pubs/sp/800/41/r1/final).
- [NIST Cybersecurity Framework - CSF](https://www.nist.gov/cyberframework)
- [NIST SP 800-53 Rev 5 - Diretrizes sobre firewalls e política de firewall](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final)

E para melhor embasamento, e expansão do próprio conhecimento sobre as possibilidades e o mais importante: no processo cognitivo para a solução de problemas. Indico fortemente conhecer padrões de conformidade em segurança que fazem uso do NIST:

- [HIPAA](https://www.hhs.gov/hipaa/index.html)[NIST](https://csrc.nist.gov/glossary/term/payment_card_industry_data_security_standard) é uma Lei de Portabilidade e Responsabilidade de Seguros de Saúde dos EUA (HIPAA) que regula como as informações de saúde são tratadas e protegidas para garantir a proteção das informações de saúde através de controles e práticas de segurança para informações eletrônicas de saúde. E que regulamenta organizações: `covered entities` - provedores de saúde, planos de saúde e câmaras de compensação de saúde; `business associates` - empresas de cobrança, fornecedores de registro eletrônico de saúde (RES), consultores ou provedores de TI.

- [Personally Identifiable Information - PII](https://csrc.nist.gov/glossary/term/PII) que são quaisquer dados que possam ser usados para identificar alguém e que direta ou indiretamente se vinculam a uma pessoa. Qual empresa não coleta, armazena e processa PII nos dias de hoje? É claro que é uma definição americana! A GPDR UE tem outra definição: dados pessoais. Mas o importante é o arcabouço cognitivo de estar atendo ao métodos de proteger esses dados pessoais etc.

- [Payment Card Industry - PCI](https://www.pcisecuritystandards.org/document_library/) [NIST](https://csrc.nist.gov/glossary/term/pci_dss) é um conjunto de políticas de segurança para os dados do portador do cartão. Organizações que processam transações financeiras com cartões de crédito, débito e pré-pago estão sujeitas aos requisitos de conformidade com PCI.

- [Federal Information Security Modernization Act - FISMA](https://www.cisa.gov/topics/cyber-threats-and-advisories/federal-information-security-modernization-act) [NIST](https://csrc.nist.gov/topics/laws-and-regulations/laws/FISMA) é uma legislação dos Estados Unidos que define uma estrutura de diretrizes e padrões de segurança para proteger as operações de tecnologia da informação do governo contra ameaças cibernéticas. Essa estrutura de gerenciamento de risco foi assinada como lei como parte do Electronic Government Act de 2002, posteriormente atualizadas e alteradas em 2014.
  Ela exige de agências federais o desenvolvimento, documentação e implementação de programas de infosec para proteger informações sensíveis e confidenciais. O ato também descreve as responsabilidades do National Institute of Standards and Technology - NIST e do Office of Management and Budget (OMB).

- [ISO/IEC 27001:2022](https://www.iso.org/standard/27001) é um norma de segurança de informações reconhecida internacionalmente, desenvolvida pelo órgão de certificação [Organização Internacional de Padronização - ISO](https://www.iso.org/) e a [Comissão Eletrotécnica Internacional - CEI](https://iec.ch/homepage). A sua versão mais recente é: ISO/IEC 27001:2022. [equivalência: SP 800-53].

- [ISO/IEC 27002:2022]() guia de melhores práticas para a implementação de controles de segurança da informação, aplicado aos controles da norma ISSO 27001:2022. [equivalência: Cybersecurity Framework - CSF e SP 800-53]

## NIST Special Publications

Estou usando como referência principal, a coleção de recomendações da subsérie de especificações técnicas [SP 800 - Computer security](https://csrc.nist.gov/publications/sp800) da  [**NIST Special Publications**](https://csrc.nist.gov/publications/sp).

O [NIST SP 800-53 Rev 5](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-53r5.pdf) que também faz referência ao [SP 800-41 Rev 1](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-41r1.pdf) como um documento para atingir conformidade em um firewall. Tem várias referências a firewall no terceiro capítulo e seguintes seções:

- Terceiro capítulo, **The Controls** - SECURITY AND PRIVACY CONTROLS AND CONTROL ENHANCEMENTS.
  - **3.1** Access Control:
    - **AC-3** ACCESS ENFORCEMENT
    - **AC-4** INFORMATION FLOW ENFORCEMENT
    - **AC-6** LEAST PRIVILEGE
  - **3.5** CONFIGURATION MANAGEMENT
    -  **CM-7** LEAST FUNCTIONALITY
  - **3.8** INCIDENT RESPONSE
    - **IR-4** INCIDENT HANDLING
  - **3.16** RISK ASSESSMENT
    - RA-10 THREAT HUNTING
  - **3.17** SYSTEM AND SERVICES ACQUISITION
    - **SA-9** EXTERNAL SYSTEM SERVICES
  - **3.18** SYSTEM AND COMMUNICATIONS PROTECTION
    - **SC-7** BOUNDARY PROTECTION
    - **SC-28** PROTECTION OF INFORMATION AT REST
  - **3.19** SYSTEM AND INFORMATION INTEGRITY
    - **SI-3** MALICIOUS CODE PROTECTION
    - **SI-4** SYSTEM MONITORING
    - **SI-7** SOFTWARE, FIRMWARE, AND INFORMATION INTEGRITY
    - **SI-8** SPAM PROTECTION

O guia [SP 800-41 Revision 1 - Guidelines on Firewalls and Firewall Policy](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-41r1.pdf) é a minha principal referência de recomendações para garantir as melhores práticas na segurança e controle de tráfego em um firewall.

**Seção 2 - Overview of Firewall Technologies**
Nessa seção temos a definição de diferentes tipos de Firewalls e arquitetura nas quais são melhor empregados para o controle do fluxo de tráfego de rede entre redes ou hosts que empregam diferentes posturas de segurança.

Inicialmente, podemos tomar como ponto de atenção. O seguinte resumo de recomendações:

- O uso de NAT deve ser considerado uma forma de roteamento, não um tipo de firewall.
  - cuidado! Seu entendimento sobre DMZ pode ser um problema!
- As organizações devem permitir apenas tráfego de saída que use os endereços IP de origem em uso pela organização.
  - esteja atento aos requisitos de segurança da arquitetura de redes.
- A verificação de conformidade só é útil em um firewall quando pode bloquear a comunicação que pode ser prejudicial aos sistemas protegidos.
  - o firewall deve se basear na segurança do layout de redes sem perder flexibilidade: permitindo assim a evolução da infraestrutura.
- Ao escolher o tipo de firewall a ser implantado, é importante decidir se o firewall precisa atuar como um proxy de aplicativo.
  - esteja atento a qual tecnologia de firewall usar observando suas capacidades de lidar com as camadas TCP/IP.
- O gerenciamento de firewalls pessoais deve ser centralizado para ajudar a criar, distribuir e aplicar políticas de forma eficiente para todos os usuários e grupos.
  - a sobreposição de firewalls sempre gerará carga operacional e complexidade.

**Seção 3 - Firewalls and Network Architectures**
Esta seção se concentra em firewalls de rede porque os outros tipos geralmente não estão relacionados a problemas de topologia de rede.

- Em geral, um firewall deve se encaixar no layout de uma rede atual. No entanto, uma organização pode alterar sua arquitetura de rede ao mesmo tempo em que implanta um firewall como parte de uma atualização geral de segurança.

- Diferentes arquiteturas de rede comuns levam a escolhas muito diferentes sobre onde colocar um firewall, então uma organização deve avaliar qual arquitetura funciona melhor para seus objetivos de segurança.

- Se um firewall de borda tiver uma DMZ, considere quais serviços voltados para fora devem ser executados a partir da DMZ e quais devem permanecer na rede interna.

- Não confie em NATs para fornecer os benefícios dos firewalls.

- Em alguns ambientes, colocar um firewall atrás de outro pode levar a um objetivo de segurança desejado, mas, em geral, essas múltiplas camadas de firewalls podem ser problemáticas.

**Seção 4 - Firewall Policy**

- 4.1 - Policies based on IP Address and Protocol

- 4.2 - Policies Based on Applications

- 4.3 - Policies Based on User Identity

- 4.4 - Policies Based on Network Activity

- Seção 5 - Firewall Planning and Implementation

  - 5.2.1 - Hardware and Software Installation
  - 5.2.3 - Logging and Alerts Configuration
  - 5.5 - Manage

## Ambiente
Ambiente de teste:

- Aplicações
  - httpd daemon,
  - OpenSSH,
  - Wireguard,
  - Prometheus Node Exporter,
  - Tor relay obfs4 bridge.
- servidor
  - VPS [VULTR](https://www.vultr.com)
  - Regular Cloud Compute.
  - 1 vCPU, 1024 MB RAM, 25 GB SSD, 1.00 TB Transfer.
  - OpenBSD 7.4 (GENERIC.MP) \#1396: Sun Oct 8 09:20:40 MDT 2023.

**Requisitos**

- Políticas de segurança baseadas em endereço IP e protocolo
- tratar tráfego loopback e pseudo interfaces,
- tratar anomalias IP e por protocolo,
- criar limites de fluxo IP,
- tratar transformações e validações de pacotes,
- criar políticas por protocolo, serviço, host e redes,
- aplicar técnicas para mitigar DoS e DDoS,
- considerar o esgotamento de recursos computacionais.

## Ruleset

Segue abaixo, uma sugestão de conjunto de regras de firewall [*Open***BSD**](https://www.openbsd.org) PF - Packet Filtering.

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
allowed_ports_tcp = "{ 22, 80, 443, 9100, 51820, 9001, 9003 }"
allowed_ports_udp = "{ domain }"

# ICMP
# https://man.openbsd.org/icmp
icmp_types = "{  echoreq, echorep, unreach, timex, trace }"

#
non_routable = "{ 127.0.0.0/8, 192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8 , fd00::/8, fe80::/10 }"

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

# WireGuard VPN # Permite UDP na porta WireGuard. 
pass in on $ext_if inet proto udp from any to $ext_if_ip port 51820 keep state

# Políticas para redes
# Define permissões e restrições para a rede interna e a rede externa (WAN).
# Alinhado com o NIST SP 800-41, 4.1 Policies Based on IP Addresses and Protocols

# Tráfego permitido entre a rede interna (LAN) e o servidor.
pass in quick on $lan_if from $lan_net to self keep state

# Permitindo tráfego entre rede Wireguard e o servidor
pass in on $wg_if from $wg_net to self keep state

# Permissão de tráfego de saída apenas para serviços essenciais.
# Alinhado com o NIST SP 800-41, seção 4.1.1 - IP Addresses and Other IP Characteristics 4.2 - Policies Based on Applications e 4.4 Policies Based on Network Activity e 4.1.3 TCP and UDP

# TCP ports list
pass out quick on $wan_if proto tcp from $ext_if_ip to any port $allow_ptcp keep state  

# Tor bridge
pass out quick on $wan_if proto tcp from $ext_if_ip to any port $allow_pbtor keep state

#wireguard
pass out on egress inet from ($wg_if:network) nat-to ($ext_if:0) keep state

# ICMP prove
#pass out quick inet proto icmp all icmp-type $icmp_types
pass out quick on $ext_if inet proto icmp all icmp-type $icmp_types 

# DNS resolver 
pass out quick on $ext_if proto { tcp udp } from ($wan_if) to any port 53 keep state

# Tráfego de saída da LAN para WAN.
pass out on $ext_if from $int_net to any

# Monitoramento e Auditoria
# https://man.openbsd.org/pflog.4
# https://www.openbsd.org/faq/pf/logging.html
# https://man.openbsd.org/pf.conf#match
# Log de tráfego relevante para auditoria e detecção de incidentes, conforme recomendado pelo NIST SP 800-41, Seção 5.
match log (all) on $ext_if
```

Nesse documento onde aplico um conjunto de regras em um SO OpenBSD e firewall PS, apesar de nem sempre existir uma seção do documento que faz referência direta ao método que está sendo aplicado em uma regra específica; ainda assim, eu tento sinalizar as seções que tratam de forma concreta ou abstrata... ;)

Por fim, para o que foi proposto, conseguimos atender tecnicamente os seguintes pontos:

- Política de Bloqueio Padrão e Defesa em Profundidade
  - Todas as conexões de entrada e saída são bloqueadas por padrão para aplicar uma política de segurança mínima e alinhada ao conceito de "defesa em profundidade". O NIST recomenda permitir tráfego apenas quando explicitamente autorizado.
- Limites de Conexão e Fragmentação
  - Os limites de `states` e `frags` são configurados para evitar ataques DoS e DDoS, prevenindo o esgotamento de recursos no firewall e alinhando-se com o NIST sobre restrições de uso de estados.
- Regras de Normalização e Scrubbing
  - A regra `scrub` assegura a reconstrução de pacotes fragmentados e previne anomalias de tráfego, alinhando-se com as recomendações do NIST sobre validação de pacotes.
- Prevenção de spoofing, syn flood e integridade de pacotes
  - O comando `antispoof` ajuda a bloquear pacotes forjados provenientes de IPs internos e `synproxy` evitando DoS syn flood conforme o NIST recomenda a integridade de pacotes.
- Tráfego loopback
  - O tráfego loopback é permitido para garantir a comunicação interna do sistema, seguindo as diretrizes do NIST sobre a configuração de interfaces locais.
- Regras específicas por aplicação
  - Para SSH, HTTP/HTTPS, WireGuard, Prometheus e Tor, cada serviço possui regras de passagem individualizadas. O NIST recomenda políticas por serviço para controlar o tráfego conforme as necessidades do aplicativo. Cada regra possui limitação de taxa de conexões para prevenir ataques de força bruta e DoS.
- Limitação de conexões ICMP
  - Limita o número de requisições ICMP permitidas por host, ajudando a mitigar possíveis abusos e ataques de DoS via ICMP. A configuração separa IPv4 e IPv6 para garantir a compatibilidade.
- Controle e auditoria de tráfego com log
  - O `match log (all)` permite auditoria completa do tráfego de entrada e saída, alinhado ao NIST, que recomenda manter registros de incidentes para análise e conformidade.






