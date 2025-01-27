---
title: "RTFM  & Aleatoriedades - Minerando Monero com OpenBSD e XMRig"
date: 2025-01-11T20:49:50-03:00
# weight: 1
# aliases: ["/first"]
tags: ["Monero", "XMR", "XMRig", "OpenBSD", "Criptomoeda"  ]
author: "0xttfx"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Minerando Monero com OpenBSD e XMRig"
canonicalURL: false
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

--- 
- STATUS: EM REVISÃO!
---



## **1. Introdução**
Este guia é uma tentativa de orientá-lo na criação de uma carteira Monero, na instalação do software de mineração XMRig no sistema operacional OpenBSD e na otimização de hardware para obter o melhor desempenho. Este documento busca dar uma melhor compreensão dos conceitos técnicos por trás de cada passo e indicando as documentaçãoes para aprofundamento...

### **Por que Monero?**
[Monero](https://academy.bit2me.com/pt/o-que-%C3%A9-criptomoeda-monero-xmr/) é uma das criptomoedas mais avançadas do mercado quando se trata de privacidade e segurança. Ele utiliza o algoritmo de consenso [Proof-of-Work (PoW)](https://www.deltecbank.com/news-and-insights/a-proof-of-work-explanation/) e o protocolo [RandomX](https://academy.bit2me.com/pt/que-algoritmo-mineria-randomx-monero/), que foi especificamente projetado para ser resistente a [ASICs - Application Specific Integrated Circuit](https://www.elprocus.com/application-specific-integrated-circuits). Isso significa que Monero pode ser [eficientemente minerado em CPUs](https://xmrig.com/benchmark), tornando-o mais acessível e democrático para mineradores individuais.

Além disso, o [Monero oferece transações anônimas](https://coinbureau.com/analysis/is-monero-anonymous/), protegendo remetentes, destinatários e valores transacionados. Essas características fazem do Monero uma escolha popular para entusiastas de criptomoedas preocupados com privacidade e descentralização.

### **Por que OpenBSD?**
[OpenBSD](https://www.openbsd.org/faq/index.html) é conhecido por sua robustez em segurança e simplicidade. Ele fornece um ambiente estável e seguro para aplicações sensíveis, como a mineração de criptomoedas. A escolha do OpenBSD neste guia reflete seu foco em fornecer configurações confiáveis.

- [**Segurança embutida**](https://www.openbsd.org/security.html): Ferramentas como o firewall [PF](https://www.openbsd.org/faq/pf/index.html), segurança baseada em privilégios mínimos e criptografia integrada tornam o OpenBSD uma escolha ideal para proteger carteiras e operações de mineração.
- **Transparência**: Todo o código do OpenBSD é revisado com foco em qualidade e segurança, reduzindo os riscos de vulnerabilidades no sistema.

### **O que você aprenderá?**
Este guia cobre os seguintes aspectos:
**Configuração da Carteira Monero**: Instruções passo a passo para criar e gerenciar sua carteira. 
**Instalação e Configuração do Minerador XMRig**: Detalhes técnicos sobre a instalação e otimização do minerador para sua CPU e memória RAM.
**Otimização de Hardware**: Configuração de Huge Pages, cálculo de threads de mineração e gerenciamento eficiente de logs, para maximizar a eficiência e a segurança.

Ao final, espera-se você tenha uma configuração robusta, segura e otimizada para mineração de Monero no OpenBSD.

---

## **2. Ambiente do Hardware**
O hardware disponível é o principal fator que determina o desempenho da mineração de Monero. Nesta seção, exploraremos como os recursos do sistema influenciam a eficiência, como calcular o uso ideal e como o algoritmo RandomX se comporta.

### **2.1. Especificações do Servidor**
Este guia assume o uso de um servidor virtual com as seguintes configurações:

- **CPU**: 2 vCPUs
- **Memória RAM**: 6 GB
- **Armazenamento em Disco**: 50 GB SSD

Essas especificações são adequadas para uma operação inicial de mineração com otimização de recursos.

---

### **2.2. Como os recursos afetam a mineração?**
1. **Processador (CPU)**:
    
    - O algoritmo RandomX utiliza intensivamente a CPU, com foco em instruções avançadas como [AES-NI](https://www.intel.com/content/www/us/en/developer/articles/technical/advanced-encryption-standard-instructions-aes-ni.html) e [FMA3](https://en.wikipedia.org/wiki/FMA_instruction_set). Cada [thread da CPU](http://www.inf.ufes.br/~zegonc/material/Sistemas_Operacionais/Threads.pdf) processa cálculos matemáticos complexos para encontrar blocos na blockchain do Monero.
    - **Exemplo**: Um sistema com 2 [vCPUs](https://www.hivelocity.net/blog/what-is-cpu-cores-multithreading-vcpu/) pode suportar até 2 threads de mineração. Se cada thread processar 600 hashes por segundo (H/s), o sistemta terá uma taxa total de 1.200 H/s.
    - **Nota importante**: CPUs mais modernas e com suporte a instruções otimizadas terão um desempenho superior. Para servidores mais antigos ou virtuais, a eficiência será limitada.

3. **Memória RAM**:
    
    - O [algoritmo RandomX](https://github.com/tevador/RandomX/blob/1.0.0/doc/specs.md) exige cerca de [**2 GB de RAM por thread**](https://www.kryptex.com/en/articles/randomx-memory-en). Isso porque cada thread de mineração do algoritmo RandomX utiliza aproximadamente **2 GB de memória RAM** e exige cerca de **2 MB de cache L3** para operar de forma eficiente. Isso significa que o número ideal de threads depende tanto da quantidade de RAM disponível quanto do tamanho do cache L3 do processador.

    - **Impacto da RAM insuficiente**: Se a memória for insuficiente, o sistema poderá recorrer a [SWAP](https://0xttfx.tcpip.net.br/posts/rtfmaleatoriedadesparasysadmin-swap/) (uso do disco como memória), o que reduz drasticamente o desempenho da mineração.
  
    - **Exemplo**: em um sistema com **6 GB de RAM** e **2 vCPUs**, configuramos 2 threads, reservando 2 GB para o sistema operacional. Além disso, é necessário verificar o tamanho do cache L3. Se o processador tiver, por exemplo, 4 MB de cache L3, o número máximo de threads eficientes será **2 threads** (4 MB / 2 MB por thread).
	
4. **Armazenamento em Disco**:
    
    - O daemon Monero precisa armazenar a blockchain localmente para validar transações. No momento, a blockchain completa ocupa aproximadamente **120 GB**.
    - Como o servidor tem apenas 50 GB de armazenamento, será necessário configurar o modo de sincronização leve (pruned node), que reduz o tamanho da blockchain armazenada para cerca de **10 GB**.
    - **Exemplo prático**: Configurar o daemon Monero em modo pruned economiza espaço, mas ainda permite mineração sem depender de terceiros.

5. **Rede**:
    
    - A mineração requer uma conexão constante com a blockchain. Uma largura de banda mínima de 5 Mbps é recomendada para evitar atrasos na comunicação com os pools de mineração e a rede Monero.

---

### **2.3. Planejamento de Recursos**
Com base nas especificações acima, aqui está uma configuração recomendada para maximizar o desempenho no ambiente descrito:

|Recurso|Configuração Ideal|Justificativa|
|---|---|---|
|**CPU Threads**|2|Utiliza todas as vCPUs disponíveis sem sobrecarga.|
|**RAM**|6 GB|Suficiente para 2 threads, cada um consumindo 2 GB de RAM.|
|**Blockchain**|Modo Pruned (10 GB)|Adapta-se ao limite de armazenamento do servidor virtual.|
|**Pool de Mineração**|Pool com suporte a TLS (SSL)|Garante conexões seguras e estabilidade.|

---

### **2.4. Por que o algoritmo RandomX depende tanto de hardware?**
O algoritmo RandomX foi projetado para ser resistente a ASICs (circuitos integrados para aplicação específica) e otimizado para CPUs. Isso significa que ele explora características específicas do hardware para maximizar a segurança e dificultar a centralização da mineração.

- **Utilização de RAM**: RandomX realiza operações pesadas em memória para evitar otimizações de hardware (ASICs).
- **Instruções avançadas da CPU**: AES-NI e FMA3 são essenciais para acelerar cálculos criptográficos, e CPUs que suportam essas tecnologias terão um desempenho significativamente melhor.
- **Exemplo comparativo**: Um processador moderno, como um Intel Core i7, pode atingir até 7.000 H/s, enquanto um processador antigo, sem suporte a AES-NI, pode não ultrapassar 500 H/s.

---

### **2.5. Considerações para ambientes virtuais**
No caso de servidores virtuais, o desempenho pode variar devido a limitações impostas pelo hypervisor:

- **Oversubscription**: Se o provedor de hospedagem alocar mais vCPUs do que o hardware físico suporta, o desempenho do minerador pode ser inconsistente.
- **SWAP**: Em ambientes virtuais, evite ao máximo o uso de memória SWAP, pois ele depende do disco, que é muito mais lento que a RAM.
- **Latência de rede**: Uma conexão de baixa latência com o pool de mineração melhora a estabilidade e reduz rejeições de hashes.

---

## **3. Configuração da Carteira Monero**
Uma carteira é essencial para receber recompensas da mineração. Vamos configurar o Monero CLI Wallet, que oferece maior controle e segurança.

### **3.1. Por que usar a CLI Wallet?**
A CLI Wallet é leve, segura e permite integração direta com o daemon Monero. Diferente de outras carteiras que dependem de servidores externos para sincronizar com a blockchain, a CLI Wallet garante total privacidade ao conectar-se diretamente à rede.

Além disso, ela é ideal para ambientes como OpenBSD, onde a simplicidade e o controle são cruciais. A CLI Wallet oferece:

- **Autonomia**: Você não depende de terceiros para acessar sua carteira.
- **Privacidade total**: Nenhum dado é compartilhado com provedores externos.
- **Integração com mineradores**: Configurações diretas permitem uma conexão rápida entre o minerador e a carteira.

### **3.2. Instalação**

#### **Passo 1: Download e Verificação de Integridade**
Baixe os binários oficiais da [página de downloads do Monero](https://www.getmonero.org/downloads/). Antes de instalar, é essencial verificar a autenticidade dos arquivos para garantir que eles não foram adulterados. Essa verificação utiliza assinaturas PGP (Pretty Good Privacy), que asseguram que o arquivo foi assinado por uma entidade confiável e não sofreu alterações. O PGP funciona por meio de criptografia assimétrica, onde uma chave privada é usada para assinar o arquivo e uma chave pública, distribuída ao público, é usada para validar essa assinatura. Para mais detalhes sobre PGP e seu funcionamento, consulte a [documentação oficial do PGP](https://www.openpgp.org/).

1. **Baixe as chaves e os arquivos de assinatura**:
    
    ```bash
    wget https://raw.githubusercontent.com/monero-project/monero/master/utils/gpg_keys/monero-project-key.asc
    wget https://www.getmonero.org/downloads/hashes.txt
    ```
    
2. **Importe a chave PGP oficial**:
    
    ```bash
    gpg --import monero-project-key.asc
    ```
    
3. **Verifique a assinatura**:
    
    ```bash
    gpg --verify hashes.txt
    ```
    
4. **Compare os hashes**:
    
    Use a ferramenta `diff` para verificar se o hash do arquivo baixado corresponde ao publicado. Primeiro, gere o hash do arquivo baixado utilizando o comando:
    
    ```bash
    sha256sum <nome-do-arquivo>
    ```
    
    Em seguida, compare o resultado com o hash publicado utilizando `diff`:
    
    ```bash
    echo "<hash-publicado>" > hash-referencial.txt
    sha256sum <nome-do-arquivo> | awk '{print $1}' > hash-baixado.txt
    diff hash-referencial.txt hash-baixado.txt
    ```
    
    Se não houver saída, os hashes coincidem, garantindo que o arquivo não foi adulterado. Isso evita ataques de supply chain.

##### Passo 2: Extração e Instalação
1. Extraia os arquivos:
    
    ```bash
    tar -xvjf monero-cli-linux.tar.bz2
    ```
    
2. Mova os binários para um local seguro:
    
    ```bash
    mv monero-cli-* /usr/local/opt/monero-cli
    chmod +x /usr/local/opt/monero-cli/*
    ```
    
3. Crie um diretório para os dados:
    
    ```bash
    mkdir -p /usr/local/var/lib/monero
    chown -R _monero /usr/local/var/lib/monero
    ```

### **3.3. Sincronização com a Blockchain**
A sincronização com a blockchain é essencial para validar transações e conectar sua carteira à rede Monero. A blockchain funciona como um registro global descentralizado, onde cada transação é registrada e validada por nós distribuídos na rede. Isso garante que todas as transações sejam verificadas de forma independente, protegendo a integridade e segurança da rede. Sem a sincronização, sua carteira não teria informações atualizadas sobre o estado da blockchain, impossibilitando o envio e recebimento de transações.

Para sistemas com armazenamento limitado, é recomendável usar o modo pruned, que elimina os dados antigos de transações já consolidadas. No modo pruned, apenas os dados essenciais para a validação de novas transações são mantidos, reduzindo o tamanho da blockchain armazenada localmente de cerca de 120 GB para aproximadamente 10 GB. Apesar de ser mais leve, esse modo ainda permite que sua carteira valide transações e participe da mineração, pois ela continua sincronizada com os blocos mais recentes da rede, garantindo que o minerador tenha acesso às informações necessárias para criar novos blocos.

#### **Daemon Monero?**
O daemon Monero («monerod») é o componente que interage diretamente com a rede blockchain, agindo como um intermediário entre a sua carteira e a rede descentralizada do Monero, baixando, validando e armazenando blocos da blockchain. Essa função garante que sua carteira tenha acesso às informações mais recentes, permitindo que transações sejam verificadas e sincronizadas com a rede de forma segura e eficiente.

#### **Passos para Configurar o Daemon**
1. **Inicie o daemon em modo pruned**:
    
    ```bash
    /usr/local/opt/monero-cli/monerod --data-dir /usr/local/var/lib/monero --prune-blockchain --detach
    ```
    
    - O modo pruned reduz o tamanho da blockchain para cerca de 10 GB, em vez de armazenar os 120 GB completos.
2. **Verifique o progresso da sincronização**:
    
    ```bash
    tail -f /usr/local/var/lib/monero/monero.log
    ```
    
3. **Configure a inicialização automática** (opcional):
    
    - Adicione ao arquivo `/etc/rc.local` para que o daemon seja iniciado automaticamente:
        
        ```bash
        /usr/local/opt/monero-cli/monerod --data-dir /usr/local/var/lib/monero --prune-blockchain --detach
        ```

### **3.4. Criação da Carteira**
1. **Execute o comando para criar uma nova carteira**:
    
    ```bash
    /usr/local/opt/monero-cli/monero-wallet-cli
    ```
    
    - Escolha um nome e senha para a carteira.
2. **Anote as palavras-semente**:
    
    - As palavras-semente (seed phrase) são a única forma de recuperar sua carteira em caso de perda ou roubo. Elas consistem em uma sequência de palavras geradas aleatoriamente, que representam as chaves privadas da sua carteira. 
	    - Sem elas, é impossível acessar seus fundos caso você perca o acesso à carteira original. 
	    - Recomenda-se armazená-las em um local seguro, como um cofre, evitando qualquer meio digital que possa ser comprometido, como fotos ou arquivos salvos na nuvem. 
	    - Para mais informações sobre segurança de palavras-semente, RTFM [Monero backup seed](https://www.getmonero.org/resources/user-guides/backup_seed.html).
	    
3. **Conecte a carteira ao daemon**:
    
    - Use o comando abaixo para sincronizar a carteira com o daemon:
        
        ```bash
        start_mining
        ```
        **RTFM**: [Monero CLI Guide](https://www.getmonero.org/resources/user-guides/cli_wallet_guide.html).

---

## **4. Instalação do Minerador XMRig**
XMRig é um dos mineradores mais eficientes e flexíveis disponíveis para Monero. Ele permite ajustes avançados de configuração, garantindo que seu hardware seja utilizado de forma ideal.

### **4.1. Dependências Necessárias**
1. Instale as dependências:
    
    ```
    pkg_add cmake gcc g++ libuv hwloc
    ```
    
    - **CMake**: Ferramenta para configurar e gerenciar o processo de compilação.
        
    - **GCC/G++**: Compiladores para transformar o código-fonte do XMRig em um binário executável.
        
    - **Libuv**: Biblioteca usada para operações assíncronas e I/O de alta performance.
        
    - **HWLOC**: Biblioteca que identifica a topologia do hardware, otimizando a alocação de threads na CPU.

### **4.2. Compilação do XMRig**
1. Clone o repositório oficial do XMRig:
    
    ```bash
    git clone https://github.com/xmrig/xmrig.git
    cd xmrig
    ```
    
2. Crie um diretório de compilação:
    
    ```bash
    mkdir build
    cd build
    ```
    
3. Configure a compilação com suporte a OpenSSL e HWLOC:
    
    ```bash
    cmake .. -DWITH_OPENSSL=ON -DWITH_HWLOC=ON
    ```
    
4. Compile o código:
    
    ```bash
    make -j$(sysctl -n hw.ncpu)
    ```
    
    - O parâmetro `-j` utiliza múltiplos núcleos da CPU para acelerar a compilação, permitindo que várias tarefas sejam executadas simultaneamente. Para mais informações RTFM [GNU Make](https://www.gnu.org/software/make/manual/make.html#Parallel).

### **4.3. Configuração do Minerador**
1. Edite o arquivo de configuração JSON:
    
    ```bash
    vim /usr/local/etc/xmrig-config.json
    ```
    
    - Exemplo de configuração básica:
        
        ```json
        {
            "api": {
                "worker-id": "minera1",
                "port": 8080
            },
            "cpu": {
                "enabled": true,
                "huge-pages": true,
                "asm": "auto",
                "max-threads-hint": 100
            },
            "pools": [
                {
                    "url": "pool.supportxmr.com:3333",
                    "user": "ENDEREÇO_DA_CARTEIRA",
                    "pass": "",
                    "tls": true
                }
            ]
        }
        ```
        
    - **Parâmetros**:
        
        - `worker-id`: Identifica este minerador em um pool de mineração, útil para monitorar desempenho.
            
        - `huge-pages`: Ativa o uso de Huge Pages para melhorar a eficiência de memória.
            
        - `asm`: Configura o uso de instruções otimizadas da CPU, como AES-NI.
            
        - `max-threads-hint`: Define a porcentagem de threads disponíveis que serão usadas.
            
        - `tls`: Habilita conexões seguras com o pool.


2. Escolha do pool de mineração:
    
    - Recomendamos o uso de pools confiáveis, como o [SupportXMR](https://supportxmr.com/). 
	    - Pools de mineração são servidores que conectam diversos mineradores para trabalharem juntos na solução de blocos, dividindo as recompensas de acordo com a contribuição de cada participante. 
	    - Pools grandes, como o SupportXMR, oferecem maior estabilidade devido ao número constante de mineradores conectados, reduzindo as variações nos pagamentos. 
	    - Por outro lado, pools menores podem aumentar as recompensas individuais a longo prazo, pois há menos divisão de lucros, embora isso possa acarretar períodos maiores sem recompensas dependendo da frequência com que novos blocos são encontrados. 
	    - Para mais informações sobre como funcionam os pools de mineração: RTFM [XMRig](https://xmrig.com/docs/mining).

### **4.4. Teste e Execução do Minerador**
1. Teste a configuração:
    
    ```
    ./xmrig --config /usr/local/etc/xmrig-config.json --dry-run
    ```
    
    - Isso verifica se o minerador está configurado corretamente sem iniciar a mineração.
    - **atente-se a**:
	    - **Pool acessível**: Certifique-se de que o minerador consegue se conectar ao pool configurado.
	    - **Detecção de hardware**: Verifique se o minerador detecta corretamente as threads.
        
2. Execute o minerador:
    
    ```
    ./xmrig --config /usr/local/etc/xmrig-config.json
    ```
    
3. Monitore o desempenho:
	- Use a API local configurada (porta 8080 por padrão) para acessar estatísticas detalhadas do minerador e monitorar o desempenho em tempo real. A API fornece informações como:
        
		- Taxa de hashes (H/s) para cada thread configurada.
    
		- Uso de CPU e memória.
    
		- Status da conexão com o pool (tempo de resposta, estabilidade).
    
		- Relatórios de aceitação e rejeição de trabalhos enviados ao pool.

Para acessar a API, utilize um comando como:

```bash
http://127.0.0.1:8080
```

A API retorna os dados em formato JSON, que podem ser analisados manualmente ou integrados a sistemas de monitoramento.

 ```bash
 curl http://127.0.0.1:8080
 ```

- Alternativamente, monitore diretamente pelo terminal. O XMRig exibirá informações como taxa de hashes (H/s), uso de CPU e estatísticas de conexão com o pool.
    
**Mais detalhes sobre a API**:  RTFM [XMRig GitHub](https://github.com/xmrig/xmrig)

## **5. Otimização de Hardware**
A otimização do hardware é crucial para garantir que o minerador opere com máxima eficiência e estabilidade, utilizando plenamente os recursos disponíveis sem sobrecarregar o sistema.

### **5.1. Huge Pages**

#### **O que são Huge Pages e por que são importantes?**
Huge Pages são blocos de memória de tamanho maior que as páginas padrão (normalmente 2 MB em vez de 4 KB). Elas foram introduzidas para melhorar o desempenho em aplicações que exigem acesso frequente a grandes quantidades de memória, como o algoritmo RandomX utilizado pelo Monero.
- Ao reduzir a fragmentação e o overhead do gerenciamento de memória, Huge Pages permitem que grandes blocos de dados sejam acessados com menor latência e menor consumo de CPU. 
	- Isso é especialmente vantajoso para operações de mineração por: 
		- evitar constantes trocas de páginas e falhas de cache. 
		- reduzir do número de falhas de tradução de endereços [TLB misses](https://www.geeksforgeeks.org/translation-lookaside-buffer-tlb-in-paging)
		- Melhor aproveitamento da memória para operações intensivas de criptografia.

	- Para mais informações técnicas sobre Huge Pages: RTFM [Kernel Linux Huge Pages](https://www.kernel.org/doc/Documentation/vm/hugetlbpage.txt).

**Impacto no desempenho**: O uso de Huge Pages permite que o minerador acesse a memória de forma mais rápida e eficiente, reduzindo falhas de cache e o consumo de CPU causado pela troca de páginas menores.

#### **Como calcular Huge Pages necessárias?**
1. **Cálculo base**:
    - Cada thread de mineração do RandomX utiliza cerca de 2 GB de RAM.
    - Para um sistema com 2 threads e 6 GB de RAM

```bash
ram_alocada_mb=4096
tamanho_pagina_mb=2
num_huge_pages=$((ram_alocada_mb / tamanho_pagina_mb))
```

- Isso resultará no valor  de Huge Pages necessárias para o sistema, com base na RAM alocada para a mineração. `Número de Huge Pages = (RAM alocada para mineração em MB) / 2 MB Exemplo: 4096 MB / 2 MB = 2048 Huge Pages`

```bash
$ echo "Valor Huge Pages: $num_huge_pages"
Valor Huge Pages: 2048
```

2. **Configuração no OpenBSD**:
    - Ajuste o número de Huge Pages no sistema:
        
        ```bash
        sysctl vm.hugetlb_pglist=2048 && echo "vm.hugetlb_pglist=2048" >> /etc/sysctl.conf
        ```
        - para verificação
        ```bash
        vmstat -m | grep hugetlb
        ```
    - Para garantir que o minerador tenha permissões para acessar Huge Pages, é necessário ajustar as permissões no sistema. 
	    - No OpenBSD, você pode fazer isso criando as regras apropriadas para o usuário que executará o minerador. Use o seguinte comando para verificar as permissões e, se necessário, ajustá-las:
			```bash
			chmod 0660 /dev/hugepages
			chown _xmrig:_xmrig /dev/hugepages
			```

Este exemplo configura o dispositivo Huge Pages para ser acessível pelo usuário `_xmrig` e pelo grupo correspondente. Certifique-se de que o serviço do minerador está sendo executado sob o mesmo usuário configurado. 
- Para mais informações: RTFM [ sobre dispositivos de memória compartilhada](https://man.openbsd.org/intro).

---

### **5.2. Configuração de Threads**

#### **Por que ajustar os threads?**
A configuração de threads acima da capacidade do hardware pode levar à sobrecarga porque cada thread de mineração depende de recursos físicos (vCPU e RAM) disponíveis no sistema. Se mais threads forem configuradas do que o hardware pode suportar, os seguintes problemas podem ocorrer:

1. Competição por recursos da CPU:  cada thread tenta usar uma vCPU. Configurar mais threads do que o número de vCPUs força o sistema operacional a compartilhar ciclos de CPU entre threads adicionais, o que reduz a eficiência de processamento.
    
2. Excesso de uso de RAM:  se mais threads forem configuradas do que a RAM disponível, o sistema utilizará SWAP (memória em disco), o que é significativamente mais lento e prejudica o desempenho da mineração.
    
3. Diminuição do hash rate:  a sobrecarga reduz a taxa de hashes por segundo (H/s), pois as threads competem por recursos limitados, criando gargalos no processamento.

- Cada thread de mineração RandomX
	- utiliza **2 GB de RAM** e aproximadamente **2 MB de cache L3**. 

Por isso é preciso ajustar o número de threads para corresponder aos recursos disponíveis, levando em conta não apenas o minerador, mas também as necessidades do sistema operacional e outros serviços em execução.

#### **Como calcular o número ideal de threads?**
1. **Fatores principais**:
    
    - **vCPUs disponíveis**: O número de threads não deve exceder o total de vCPUs do servidor (2 neste caso).
    - **RAM disponível**: Certifique-se de que haja memória suficiente para cada thread (2 GB por thread).
    
2. **Exemplo para este servidor 
	- 6 GB de RAM, 2 vCPU
	    - Reserve 2 GB para o sistema operacional.
	    - Aloque 4 GB para a mineração (2 threads).
			1. Calcule o número máximo de threads baseado na RAM disponível:
				```bash
				ram_disponivel_mb=4096  # Subtraia a RAM reservada para o sistema operacional
				ram_por_thread_mb=2048
				threads_ram=$((ram_disponivel_mb / ram_por_thread_mb))
				echo "Threads baseadas na RAM: $threads_ram"
				Threads baseadas na RAM: 2
				```
				- **ram_disponivel_mb**: Subtraia a memória reservada para o sistema operacional.
				- **ram_por_thread_mb**: Cada thread consome 2 GB de RAM.
				- **threads_ram**: Número de threads possíveis baseado na RAM
			1. Calcule o número máximo de threads baseado no cache L3:
				```bash
				cache_l3_kb=4096  # Tamanho do cache L3 em KB
				cache_por_thread_kb=2048
				threads_cache=$((cache_l3_kb / cache_por_thread_kb))
				echo "Threads baseadas no cache L3: $threads_cache"
				Threads baseadas no cache L3: 2
				```
				- **cache_l3_kb**: O cache total disponível.
				- **cache_por_thread_kb**: Cada thread utiliza 2 MB de cache L3.
			1. Combine os resultados para determinar o limite ideal:
				```bash
				threads_totais=$((threads_ram < threads_cache ? threads_ram : threads_cache))
				echo "Número ideal de threads: $threads_totais"
				Número ideal de threads: 2
			   ```
			    - O menor valor entre `threads_ram` e `threads_cache` determina o número ideal de threads.

3. **Configuração no arquivo JSON**:
    
    ```json
    "cpu": {
        "max-threads-hint": 75
    }
    ```
    - O valor `75` utiliza 75% das threads disponíveis, respeitando os limites calculados.
    - Como o servidor possui apenas 2 threads físicas (vCPUs) e o sistema operacional também precisa de recursos para operar, é necessário ajustar a configuração para evitar concorrência excessiva. Para garantir que o minerador não consuma toda a capacidade do sistema, considere reduzir o valor de `max-threads-hint` para algo como `75`, permitindo que 25% dos recursos das vCPUs sejam reservados para o sistema operacional e outros serviços em execução. Isso mantém o sistema estável enquanto maximiza o desempenho da mineração.
4. Para definir com precisão a quantidade de threads que podem ser utilizadas pelo minerador sem comprometer o desempenho do sistema operacional e outros serviços, é importante monitorar o consumo de recursos do sistema em tempo real. O OpenBSD oferece ferramentas nativas para essa tarefa:
	1. **Verificando o uso de CPU e RAM com** `top`
		1. Execute o comando `top` para monitorar os processos em execução:
		```
		$ top
	    ```
	    - **CPU Usage**: Observe a porcentagem de CPU usada pelo kernel, processos do sistema e outros serviços essenciais.
	    - **Memory Usage**: Verifique a quantidade de memória RAM em uso pelo sistema e serviços. Os campos relevantes incluem:
		    - `Active`: Memória utilizada pelos processos ativos.
		    - `Free`: Memória disponível para uso.
    2. **Analisando com** `vmstat`
	    1. Utilize o `vmstat` para obter uma visão detalhada do uso do sistema:
	    ```
	    $ vmstat 1
	    ```
	    - **CPU Metrics**:
		    - `us`: Tempo gasto em processos do usuário.
		    - `sy`: Tempo gasto em processos do sistema.
		    - `id`: Tempo de inatividade da CPU.
        - **Memory Metrics**:
	        - `free`: Quantidade de memória livre.
	        - `swap`: Quantidade de memória utilizada no swap.
        2. Execute o comando durante alguns minutos para capturar um padrão de uso médio.
    3. **Calculando a porcentagem de CPU para threads**
	    1. Após determinar o consumo médio do sistema operacional. Calcule a CPU restante disponível para o minerador:
	    ```
	    $ cpu_disponivel=$((100 - <percentual_de_uso_medio_da_cpu>))
	    $ max_threads=$((cpu_disponivel * <número_total_de_threads> / 100))
	    $ echo "Threads disponíveis para o minerador: $max_threads"
	    ```
	    - Substitua `<percentual_de_uso_medio_da_cpu>` pelo valor observado em `vmstat` ou `top`.
	    - Ajuste `<número_total_de_threads>` conforme o número total de vCPUs.

---

### **5.3. Logs e Monitoramento**
O gerenciamento de logs e o monitoramento são fundamentais para garantir que o minerador opere de forma eficiente e detectar problemas de desempenho ou falhas no sistema.

#### **1. Configuração de logs no XMRig**
1. Edite o arquivo de configuração do XMRig:
    
    ```bash
    nano /usr/local/etc/xmrig-config.json
    ```
    
    - Adicione ou ajuste os seguintes parâmetros:
        
        ```json
        "log-file": "/var/log/xmrig/xmrig.log",
        "log-level": 2
        ```
        
    - **Explicação dos parâmetros**:
        - `log-file`: Especifica o caminho onde os logs do minerador serão salvos.
        - `log-level`: Define o nível de detalhamento dos logs:
            - `0`: Apenas mensagens críticas.
            - `1`: Mensagens de erro.
            - `2`: Informações gerais (recomendado para monitoramento padrão).
            - `3`: Debug (uso avançado, detalhamento completo).

#### **2. Automação de rotação de logs**
1. Adicione uma entrada ao arquivo `/etc/newsyslog.conf` para configurar a rotação automática de logs:
    
    ```bash
    /var/log/xmrig/xmrig.log 640 7 * 24 Z
    ```
    
    - **Explicação dos campos**:
        - `640`: Permissões do arquivo de log rotacionado.
        - `7`: Número de arquivos de log antigos a serem mantidos.
        - `*`: Frequência de rotação (diária, semanal, etc.).
        - `24`: Hora da rotação.
        - `Z`: Compacta os logs antigos.
2. Aplique as configurações imediatamente:
    
    ```bash
    newsyslog

#### **3. Monitoramento do desempenho**
1. **API Local**:
    
    - Use a API do XMRig configurada na porta 8080 para acessar estatísticas em tempo real:

	  ```bash
		curl http://127.0.0.1:8080
		```
        
        - A API retorna informações em JSON, incluindo:
            - Taxa de hashes (H/s).
            - Uso de threads.
            - Latência de conexão com o pool.
            - Taxa de aceitação/rejeição de shares.
2. **Análise de logs no terminal**:
    
    - Visualize os eventos mais recentes:
        
        ```bash
        tail -f /var/log/xmrig/xmrig.log
        ```
        
        - Procure por mensagens de erro ou desempenho, como:
            - Erros de conexão com o pool.
            - Alerta de falta de memória ou falhas de threads.
3. **Ferramentas de monitoramento avançado**:
    
    - Integre o XMRig com ferramentas como Prometheus e Grafana para um monitoramento gráfico contínuo.
    - Configure exportadores JSON para enviar os dados da API local para sistemas de monitoramento:
        - [Guia de integração com Prometheus](https://prometheus.io/docs/).
        - [Configuração no Grafana](https://grafana.com/docs/).

#### **4. Boas práticas para logs em ambientes de produção**
1. **Gerenciamento de espaço em disco**:
    
    - Certifique-se de que a rotação de logs está configurada para evitar uso excessivo de espaço.
    - Monitore regularmente o diretório de logs com:
        
        ```bash
        du -h /var/log/xmrig
        ```
        
2. **Proteção de logs sensíveis**:
    
    - Configure permissões restritas para o diretório e os arquivos de log:
        
        ```bash
        chmod 640 /var/log/xmrig/xmrig.log
        chown _xmrig:_xmrig /var/log/xmrig/xmrig.log
        ```
        
3. **Auditoria periódica**:
    
    - Revise os logs semanalmente para identificar possíveis problemas antes que afetem o desempenho.

---

