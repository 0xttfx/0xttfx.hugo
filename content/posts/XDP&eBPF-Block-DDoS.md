---
title: "XDP & eBPF Block DoS"
date: 2025-01-22T17:13:52-03:00
author: "Faioli a.k.a 0xttfx"
tags:
  - XDP
  - eBPF
  - Bash
  - Linux
  - nftables
categories:
  - Networking
collections:
  - Getting Started
draft: false
---

![Netfilter-packet-flow](/img/XDPeBPFDoS/eBPF.png)


Segue um teste para aplicar os conceitos de eBPF! É um esboço de uma solução que espera ser funcional e que combina um programa [XDP](https://prototype-kernel.readthedocs.io/en/latest/networking/XDP/introduction.html#what-is-xdp)/[eBPF](https://ebpf.io/) em C com um script em Bash(antes de usar Go) para monitorar o tráfego e atualizar dinamicamente um [map](https://docs.cilium.io/en/latest/reference-guides/bpf/architecture/#maps) [eBPF](https://ebpf.io/) com sources IPs que devem ser bloqueados quando idenficado um possível ataque [DoS](https://attack.mitre.org/techniques/T0814/). 

- Nesse cenário, o programa XDP/eBPF já está instalado na interface de rede e bloqueia pacotes provenientes de IPs listados no [map](https://docs.cilium.io/en/latest/reference-guides/bpf/architecture/#maps). 
- O script em Bash monitora  o arquivo de subsistema de rastreamento [conntrack](https://conntrack-tools.netfilter.org) do [Netfilter](https://www.netfilter.org/) [/proc/net/nf_conntrack](https://conntrack-tools.netfilter.org/manual.html#conntrack) para determinar se o número de conexões ativas ultrapassou o threshold configurado, e que ao ser atingido:
	- um log gerado por regras específicas [nftables](https://www.netfilter.org/projects/nftables/index.html) é analisado para identificar com base em uma "janela" de tempo, qual source IP ultrapassou o limite de conexões: indicando ser um possível ataque DoS:
		- que então é inserido no `map` eBPF, fazendo com que o programa XDP passe a descartar os pacotes desses endereços.



## 1. Pré-requisitos

- Kernel Linux com suporte a [eBPF e XDP](https://docs.cilium.io/en/latest/reference-guides/bpf/index.html#bpf-and-xdp-reference-guide)  
    Certifique-se de utilizar uma distribuição com kernel moderno (>= 4.18, de preferência 5.x ou superior).
- clang
- llvm
- gcc
- libbpf
- libbpf-devel 
- libxdp 
- libxdp-devel 
- xdp-tools 
- bpftool kernel-headers
- perf
- nftables
- iproute2


## Considrações e Limitações
- Rastreamento de Conexões:
	
  - **Real time**

  O `nf_conntrack` é atualizado dinamicamente pelo kernel, oferecendo um “instantâneo” do estado atual das conexões, possibilitando a detecção rápida de mudanças no tráfego.  
  - **Stateful**

  O `nf_conntrack` dá uma visão de conexões `stateful` ativas e seus estados (NEW, ESTABLISHED, RELATED e FIN_WAIT…). Cada linha em   `/proc/net/nf_conntrack` representa uma entrada que pode conter informações detalhadas, como endereços IP de origem/destino, portas, protocolos e o estado da conexão. Comprometendo a detecção de protocolos stateless... 
   - **Tabela conntrack**

  Tem tamanho limitado e, em condições normais, é dimensionada para o tráfego esperado. Mas em ataques volumétricos, o número de conexões satura a tabela, impedindo que novas conexões sejam registradas. Isso não só prejudica a detecção como pode impactar a disponibilidade do serviço, mesmo para tráfego legítimo.
  - **Falsos Positivos**

  Em ambientes com tráfego elevado – por exemplo, servidores com muitos clientes reais ou aplicações que abrem múltiplas conexões simultâneas – um pico pode ocorrer de forma legítima. Sem uma correlação com outros indicadores (como análise de logs ou métricas de latência), o sistema pode interpretar um aumento repentino de conexões como um ataque. 
  - **Complementaridade**

  Para uma detecção mais robusta, é recomendável combinar a análise de diversas fontes de dados. 
  - **Latência na Atualização**

  O método de contagem, que usa wc -l para contar linhas, pode não refletir exatamente a taxa de chegada de novas conexões, especialmente em sistemas de alta performance, onde as entradas podem ser criadas e removidas em milissegundos.


## 2. Programa XDP/eBPF

O programa utiliza um [map](https://docs.cilium.io/en/latest/reference-guides/bpf/architecture/#maps) eBPF do tipo HASH para armazenar os IPs que devem ser bloqueados. E para cada pacote recebido, é verificado se source IP consta no mapa. Se sim, o pacote é descartado ([XDP_DROP](https://prototype-kernel.readthedocs.io/en/latest/networking/XDP/implementation/xdp_actions.html)); caso contrário, ele é encaminhado normalmente ([XDP_PASS](https://prototype-kernel.readthedocs.io/en/latest/networking/XDP/implementation/xdp_actions.html)).

>[!INFO] **RTFM** e código atualizado: [repo Git](https://github.com/0xttfx/xdp-block-ddos)

---

```c
#include <linux/bpf.h>
#include <linux/if_ether.h>
#include <linux/ip.h>
#include <bpf/bpf_helpers.h>
#include <bpf/bpf_endian.h>

/*
 * Mapa do tipo HASH para armazenar os IPs bloqueados.
 * - Chave: __u32 (IP de origem, em formato de rede, em big-endian)
 * - Valor: __u32 (flag, por exemplo, 1 para indicar bloqueio)
 * - max_entries: define quantos IPs podem ser bloqueados simultaneamente
 */
struct {
    __uint(type, BPF_MAP_TYPE_HASH);
    __uint(max_entries, 1024);
    __type(key, __u32);
    __type(value, __u32);
} blocked_ips SEC(".maps");

/*
 * Programa XDP para bloquear pacotes provenientes de IPs listados no mapa.
 *
 * Fluxo do programa:
 *   1. Obtém os ponteiros para o início e fim dos dados do pacote.
 *   2. Valida a presença completa do cabeçalho Ethernet.
 *   3. Processa somente pacotes IPv4.
 *   4. Valida a presença do cabeçalho IP completo e a integridade do campo IHL.
 *   5. Extrai o IP de origem (localizado nos primeiros 20 bytes do cabeçalho IP).
 *   6. Consulta o mapa 'blocked_ips' e, se o IP estiver bloqueado, retorna XDP_DROP.
 *      Caso contrário, o pacote segue normalmente (XDP_PASS).
 *
 * Detalhes de performance:
 *   - A execução ocorre no caminho de dados (driver) com alta performance e baixa latência.
 *   - As verificações de limites são essenciais para satisfazer o verificador do eBPF.
 *   - A lógica foi mantida mínima, sem operações adicionais, para otimizar o tempo de execução.
 */
SEC("xdp")
int xdp_block_dos(struct xdp_md *ctx)
{
    // Obtém os ponteiros para os dados do pacote
    void *data = (void *)(long)ctx->data;
    void *data_end = (void *)(long)ctx->data_end;

    // Verifica se o cabeçalho Ethernet está completo
    struct ethhdr *eth = data;
    if ((void *)(eth + 1) > data_end)
        return XDP_PASS;

    // Processa somente pacotes IPv4
    if (bpf_ntohs(eth->h_proto) != ETH_P_IP)
        return XDP_PASS;

    // Calcula o início do cabeçalho IP (logo após o cabeçalho Ethernet)
    struct iphdr *ip = data + sizeof(*eth);
    
    // Verifica se pelo menos os 20 bytes mínimos do cabeçalho IP estão presentes.
    if ((void *)ip + sizeof(struct iphdr) > data_end)
        return XDP_PASS;

    // Valida o campo IHL: deve ser pelo menos 5 (5 * 4 = 20 bytes).
    if (ip->ihl < 5)
        return XDP_PASS;
    
    // Opcional: verificação completa do cabeçalho IP, considerando opções se presentes.
    if ((void *)ip + (ip->ihl * 4) > data_end)
        return XDP_PASS;

    // Extrai o IP de origem (localizado nos primeiros 20 bytes e sempre presente se ip->ihl >= 5)
    __u32 src_ip = ip->saddr;

    // Consulta no mapa: se o IP de origem estiver presente, descarta o pacote.
    __u32 *blocked = bpf_map_lookup_elem(&blocked_ips, &src_ip);
    if (blocked)
        return XDP_DROP;

    return XDP_PASS;
}

// Obrigatório: licença do programa eBPF, necessária para que o kernel aceite o carregamento.
char _license[] SEC("license") = "GPL";

```

### Detalhes Técnicos

1. **Verificações de Segurança e Integridade:**
    
    - **Cabeçalho Ethernet:**  
        O código garante que o cabeçalho Ethernet completo esteja presente, comparando o ponteiro para o fim do cabeçalho com `data_end`.
    - **Cabeçalho IP:**
        - É verificado se pelo menos 20 bytes (o tamanho mínimo do cabeçalho IP) estão disponíveis.
        - Verificação para o campo `ihl` (Internet Header Length), garantindo que seja maior ou igual a 5.
        - Se o IP possuir opções (quando `ihl > 5`), o código valida que o cabeçalho completo (tamanho `ip->ihl * 4`) está dentro dos limites do buffer.


2. **Extração do IP de Origem:**
    
    - O IP de origem (`ip->saddr`) é extraído sem necessidade de cálculos adicionais, pois ele sempre se encontra nos primeiros 20 bytes do cabeçalho IP.

3. **Consulta no Mapa eBPF:**
    
    - O mapa `blocked_ips` é consultado utilizando o helper `bpf_map_lookup_elem`.
    - Se o ponteiro retornado for não nulo, isso indica que o IP está bloqueado e, consequentemente, o pacote é descartado com `XDP_DROP`.
    - Essa operação é extremamente rápida, pois o mapa foi definido para ter um acesso em tempo O(1) (tabela hash).

4. **Performance e Considerações de Otimização:**
    
    - **Execução no XDP:**  
        O programa é executado no XDP, logo na entrada dos pacotes, o que garante baixa latência e alta performance.
    - **Minimalismo:**  
        A lógica foi mantida mínima para reduzir o número de instruções e evitar qualquer latência adicional.
    - **Verificador eBPF:**  
        As verificações de limites são necessárias para passar pelo verificador do kernel, garantindo que o programa seja seguro e não acesse memória fora dos limites.
    - **Possíveis Micro-Otimizações:**  
        Em cenários mais avançados, técnicas como _branch prediction hints_ (por meio de macros como `__builtin_expect`) podem ser consideradas, mas normalmente o compilador BPF já otimiza o código de forma eficaz.

5. **Licenciamento:**
    
    - A inclusão da string de licença `"GPL"` é obrigatória para o carregamento do programa no kernel.


### Compilando

Utilize o clang para compilar para o formato eBPF:

```bash
clang -O2 -g -target bpf -c -Wall -Werror src/xdp-block-dos-0.1.0.c -o xdp-block-dos.o
```


## 3. Carregando na Interface de Rede

Para carregar o programa na interface, vamos usar  o comando  [ip](https://man7.org/linux/man-pages/man8/ip.8.html) do [iproute2](https://wiki.linuxfoundation.org/networking/iproute2)

```bash
sudo ip link set dev <?> xdp obj xdp-block-dos.o sec xdp
```

Para verificar se o programa foi carregado corretamente, utilize:

- note a anotação `prog/xdp id 87 name xdp_block_dos ...`

```bash
sudo ip -details link show dev enp1s0

2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 xdp qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 52:54:00:9a:8d:fd brd ff:ff:ff:ff:ff:ff promiscuity 0 allmulti 0 minmtu 68 maxmtu 65535 numtxqueues 1 numrxqueues 1 gso_max_size 65536 gso_max_segs 65535 tso_max_size 65536 tso_max_segs 65535 gro_max_size 65536 gso_ipv4_max_size 65536 gro_ipv4_max_size 65536 parentbus virtio parentdev virtio1 
    prog/xdp id 87 name xdp_block_dos tag 20447cf6515f37e3 jited load_time 53777200546280 created_by_uid 0 btf_id 150 
```

Se necessário, para remover o programa:

```bash
sudo ip link set dev <?> xdp off
```



## 4. Script Bash para Monitoramento e Atualização do Mapa
Segue script em Bash que implementa uma abordagem híbrida para detectar um ataque DDoS: 

- Monitorando o número de conexões em [/proc/net/nf_conntrack](https://conntrack-tools.netfilter.org/manual.html#conntrack).

- E existindo um pico suspeito, analisa os logs do `nftables` para identificar IPs suspeitos devido a sua alta frequência.

- Para então atualizar o mapa eBPF (fixado em um caminho configurável) para bloquear os IPs suspeitos.

>[!INFO] **RTFM** e código atualizado: [repo Git](https://github.com/0xttfx/monitor_ddos)

---

```bash
#!/bin/bash

# Valores padrões
MAP_PIN_DEFAULT="/sys/fs/bpf/blocked_ips"
THRESHOLD_CONN_DEFAULT=2000
LOG_THRESHOLD_DEFAULT=5
INTERVAL_DEFAULT=10
LOG_FILE_DEFAULT="/var/log/nftables.log"

# Inicializa variáveis com os valores padrões
MAP_PIN="$MAP_PIN_DEFAULT"
THRESHOLD_CONN="$THRESHOLD_CONN_DEFAULT"
LOG_THRESHOLD="$LOG_THRESHOLD_DEFAULT"
INTERVAL="$INTERVAL_DEFAULT"
LOG_FILE="$LOG_FILE_DEFAULT"

# Função para exibir a mensagem de ajuda
usage() {
    cat << EOF
Uso: $(basename "$0") [opções]

Opções:
  -m MAP_PIN         Caminho do mapa eBPF pinado (padrão: $MAP_PIN_DEFAULT)
  -c THRESHOLD_CONN  Número de conexões para acionar análise (padrão: $THRESHOLD_CONN_DEFAULT)
  -l LOG_THRESHOLD   Número de ocorrências de um IP nos logs para bloqueá-lo (padrão: $LOG_THRESHOLD_DEFAULT)
  -i INTERVAL        Intervalo de tempo para verificação em segundos (padrão: $INTERVAL_DEFAULT)
  -f LOG_FILE        Caminho para o arquivo de log do nftables (padrão: $LOG_FILE_DEFAULT)
  -h                 Exibe esta mensagem de ajuda e sai

Exemplo:
  sudo $(basename "$0") -m /sys/fs/bpf/blocked_ips -c 500 -l 10 -i 15 -f /var/log/nftables.log

EOF
}

# Processa as opções de linha de comando
while getopts "m:c:l:i:f:h" opt; do
    case "$opt" in
        m) MAP_PIN="$OPTARG" ;;
        c) THRESHOLD_CONN="$OPTARG" ;;
        l) LOG_THRESHOLD="$OPTARG" ;;
        i) INTERVAL="$OPTARG" ;;
        f) LOG_FILE="$OPTARG" ;;
        h)
            usage
            exit 0
            ;;
        ?)
            usage
            exit 1
            ;;
    esac
done

echo "Configurações utilizadas:"
echo "  MAP_PIN:         $MAP_PIN"
echo "  THRESHOLD_CONN:  $THRESHOLD_CONN"
echo "  LOG_THRESHOLD:   $LOG_THRESHOLD"
echo "  INTERVAL:        $INTERVAL"
echo "  LOG_FILE:        $LOG_FILE"
echo

# Função: Converte um IP (formato x.x.x.x) para valor hexadecimal (big-endian)
ip_to_hex() {
    local ip="$1"
    IFS='.' read -r a b c d <<< "$ip"
    printf "0x%02X%02X%02X%02X" "$a" "$b" "$c" "$d"
}

# Função: Atualiza o mapa eBPF para bloquear um IP específico
block_ip() {
    local ip="$1"
    local key_hex
    key_hex=$(ip_to_hex "$ip")
    echo "Bloqueando IP suspeito: $ip (chave: $key_hex)"
    bpftool map update pinned "$MAP_PIN" key "$key_hex" value 0x1 2>/dev/null
}

# Função: Retorna a contagem de conexões em /proc/net/nf_conntrack
get_conn_count() {
    if [ -f /proc/net/nf_conntrack ]; then
        wc -l < /proc/net/nf_conntrack
    else
        echo "0"
    fi
}

# Loop principal de monitoramento
while true; do
    conn_count=$(get_conn_count)
    echo "Conexões atuais: $conn_count"

    if [ "$conn_count" -gt "$THRESHOLD_CONN" ]; then
        echo "Alerta: Pico de conexões detectado ($conn_count > $THRESHOLD_CONN)."
        echo "Analisando os logs do nftables para identificar IPs suspeitos..."

        # Captura uma janela de logs do nftables durante o intervalo definido.
        # 'timeout' limita a execução do tail, 'grep' filtra linhas com "SRC=",
        # 'sed' extrai o IP, e 'awk' seleciona IPs com ocorrências acima do limite.
        suspicious_ips=$(timeout "$INTERVAL" tail -n 1000 "$LOG_FILE" 2>/dev/null | \
            grep "SRC=" | sed -n 's/.*SRC=\([0-9.]\+\).*/\1/p' | sort | uniq -c | \
            awk -v thresh="$LOG_THRESHOLD" '$1 >= thresh {print $2}')

        for ip in $suspicious_ips; do
            block_ip "$ip"
        done
    else
        echo "Número de conexões dentro do esperado."
    fi

    sleep "$INTERVAL"
done
```

---

## Detalhes técnicos


### 1. Inicialização de Variáveis e Processamento de Opções

- **Definição de Valores Padrão:**  
  O script já define valores padrões para as variáveis:

  - `MAP_PIN_DEFAULT`           → `/sys/fs/bpf/blocked_ips`
  - `THRESHOLD_CONN_DEFAULT`    → 2000 (número de conexões para acionar a análise)
  - `LOG_THRESHOLD_DEFAULT`     → 5 (ocorrências para considerar um IP suspeito)
  - `INTERVAL_DEFAULT`          → 10 segundos (intervalo de verificação)
  - `LOG_FILE_DEFAULT`          → `/var/log/nftables.log`

- **Processamento com `getopts`:**  
  Utilizando `getopts`, o script permite sobrescrever os valores padrões:

  - `-m` para o caminho do mapa eBPF;
  - `-c` para o limite de conexões;
  - `-l` para o limite de ocorrências de um IP nos logs;
  - `-i` para o intervalo de tempo;
  - `-f` para o caminho do log do nftables;
  - `-h` exibi ajuda e sai ...o/

*Após processar os argumentos, os valores configurados são informados*

---

### 2. Função `auto_pin_map`
  
  Essa função garante que o mapa eBPF esteja “pinado” no caminho especificado.
  
- **Funcionamento:**  
  1. **Verificação da Existência:**  
     - Se o arquivo definido por `MAP_PIN` não existir, o script tenta localizar o mapa “blocked_ips” usando o comando:
       ```bash
       bpftool map show 2>/dev/null | grep -m1 "blocked_ips" | awk '{print $1}' | tr -d ':'
       ```
     - Esse comando busca na saída do `bpftool map show` a linha que contenha “blocked_ips” e extrai o ID do mapa.
  2. **Pinagem do Mapa:**  
     - Se o ID for encontrado, o script utiliza:
       ```bash
       bpftool map pin id "$map_id" "$MAP_PIN"
       ```
       para fixar o mapa no caminho desejado.
  3. **Verificação de Erro:**  
     - Caso a pinagem falhe ou o mapa não seja encontrado, o script exibe uma mensagem de erro e encerra a execução.

---

### 3. Função `ip_to_hex`
 
  Converte o IP no formato decimal (ex.: `192.168.0.1`) para uma representação hexadecimal em big-endian, que é o formato esperado pela chave no mapa eBPF.
  
- **Como Funciona:**  
  - O comando `IFS='.' read -r a b c d <<< "$ip"` divide o IP nos seus quatro octetos.
  - O `printf "0x%02X%02X%02X%02X"` formata cada octeto em dois dígitos hexadecimais e os concatena.

---

### 4. Função `block_ip`

  Inserir no mapa eBPF (pinado em `MAP_PIN`) a chave correspondente a um IP suspeito.

- **Como Funciona:**  
  1. Converte o IP para hexadecimal utilizando `ip_to_hex`.
  2. Exibe uma mensagem indicando que o IP está sendo bloqueado, junto com sua chave hexadecimal.
  3. Utiliza o comando:
     ```bash
     bpftool map update pinned "$MAP_PIN" key "$key_hex" value 0x1 2>/dev/null
     ```
     para atualizar o mapa eBPF, inserindo a chave e definindo o valor (neste caso, `0x1` indica o bloqueio).

---

### 5. Função `get_conn_count`
  
  Conta o número de linhas em `/proc/net/nf_conntrack`, que corresponde ao número de conexões rastreadas pelo sistema.

- **Como Funciona:**  
  - Se o arquivo existe, o comando `wc -l` é utilizado para contar as linhas.
  - Caso o arquivo não exista, a função retorna “0”.

---

### 6.  Função `main`

- **Fluxo do Loop:**
  1. **Leitura do Número de Conexões:**  
     - A função `get_conn_count` é chamada para obter o número atual de conexões.
     - É exibida uma mensagem com o número de conexões.
  
  1. **Verificação de Pico de Conexões:**  
     - Se o número de conexões ultrapassa o limite definido por `THRESHOLD_CONN`, o script:
       - Exibe um alerta indicando que houve um pico.
       - Inicia a análise dos logs do nftables para identificar IPs suspeitos.
  
  1. **Análise dos Logs do nftables:**  
     - Utiliza uma cadeia de comandos para extrair os IPs:
       ```bash
       suspicious_ips=$(timeout "$INTERVAL" tail -n 1000 "$LOG_FILE" 2>/dev/null | \
           grep "SRC=" | sed -n 's/.*SRC=\([0-9.]\+\).*/\1/p' | sort | uniq -c | \
           awk -v thresh="$LOG_THRESHOLD" '$1 >= thresh {print $2}')
       ```
       - **Passo a passo da cadeia:**
         - `tail -n 1000 "$LOG_FILE"`: Obtém as últimas 1000 linhas do log.
         - `timeout "$INTERVAL"`: Garante que a leitura não ultrapasse o intervalo definido.
         - `grep "SRC="`: Filtra as linhas que contêm a string “SRC=”, que indica a presença de um endereço IP de origem.
         - `sed -n 's/.*SRC=\([0-9.]\+\).*/\1/p'`: Extrai o endereço IP dessas linhas.
         - `sort | uniq -c`: Ordena os IPs e conta as ocorrências de cada um.
         - `awk -v thresh="$LOG_THRESHOLD" '$1 >= thresh {print $2}'`: Filtra apenas os IPs cuja contagem seja maior ou igual ao valor definido em `LOG_THRESHOLD`.
  
  1. **Bloqueio dos IPs Suspeitos:**  
     - Para cada IP identificado como suspeito na etapa anterior, a função `block_ip` é chamada para inserir o IP no mapa eBPF.
  
  1. **Intervalo de Espera:**  
     - Ao final de cada iteração, o script aguarda o número de segundos definido por `INTERVAL` antes de iniciar a próxima verificação.

- **Loop:**  
  O `while true; do ... done` mantém o monitoramento contínuo, permitindo uma resposta dinâmica a picos de tráfego.

---

### 8. Execução Condicional

- **Execução Direta:**  
  O trecho final:
  ```bash
  if [[ "${BASH_SOURCE[0]}" == "$0" ]]; then
      main
  fi
  ```
  Garante que o loop principal só seja iniciado se o script for executado diretamente 


## 5. Regras e o Log nftables
Geralmente, o nftables envia os logs para o kernel, e com o rsyslog podemos filtrar essas mensagens e gravá-las em um arquivo dedicado.

### Regras nftables

Segue um exemplo de regras específicas para a solução e que gerará um Log com prefixo `DDoS_ALERT: SRC=` definido:

```nft
table inet ddos_filter {
    chain input {
        type filter hook input priority 0; policy accept;
        
        ip protocol tcp ct state new limit rate 10/second burst 20 packets \
            log prefix "DDoS_ALERT: SRC=" flags all

        ip protocol udp ct state new limit rate 10/second burst 20 packets \
            log prefix "DDoS_ALERT: SRC=" flags all

        ip protocol icmp ct state new limit rate 5/second burst 10 packets \
            log prefix "DDoS_ALERT: SRC=" flags all
    }
}
```

- Com essa configuração, os eventos de log contendo `"DDoS_ALERT: SRC="` serão redirecionados para o arquivo `/var/log/nftables.log`, permitindo que o script extraia os IPs suspeitos...


### Configurar o Log

1. **Garanta que o serviço esteja rodando**  
    No Fedora, ambiente de teste usado, o `rsyslog` geralmente vem instalado, mas certifique-se de que ele está em execução:
    
    ```bash
    sudo systemctl status rsyslog
    ```
    
2. **Filtrando os Logs**  
    Crie um arquivo de configuração para o rsyslog que irá filtrar nosso log
     
   ```bash
   touch /etc/rsyslog.d/nftables.conf
   ```
	
	1. E adicione uma regra para filtrar as mensagens que contenham o prefixo  `"DDoS_ALERT: SRC="`.
		```rsyslog
	    # Filtra mensagens que contenham "DDoS_ALERT:" e as grava em /var/log/nftables.log
	    :msg, contains, "DDoS_ALERT:" -/var/log/nftables.log
	    & ~
	    ```
		- A primeira linha diz ao rsyslog: se a mensagem contiver o texto `"DDoS_ALERT:"`, envie-a para o arquivo `/var/log/nftables.log`. O hífen (`-`) antes do caminho indica gravação assíncrona, reduzindo o overhead.
		- A linha `& ~` impede que essas mensagens sejam processadas por outras regras, evitando duplicidade.
	
	1. **Reinicie o serviço**
	 ```bash
     sudo systemctl restart rsyslog
     ```
    
3. **(Opcional) Configure o systemd-journald para encaminhar os logs ao rsyslog:**  
    No Fedora, o `journald` já encaminha as mensagens para o `rsyslog` por padrão, mas se necessário, verifique o arquivo `/etc/systemd/journald.conf` e certifique-se de que a linha abaixo esteja configurada:
    
    ```bash
    ForwardToSyslog=yes
    ```
    
4. **Verifique se os logs estão sendo gravados:**  
    Após aplicar a configuração e reiniciar o rsyslog, você pode testar sua configuração gerando uma mensagem de log (por exemplo, criando um evento com nftables) e verificando o conteúdo do arquivo:
    
    ```bash
    sudo tail -f /var/log/nftables.log
    ```


## 6. Testes de Validação

... calma! To fazendo

