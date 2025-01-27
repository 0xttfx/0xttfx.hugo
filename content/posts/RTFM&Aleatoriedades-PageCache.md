---
title: "RTFM & Aleatoriedades  - Who Page Cache"
date: 2023-11-15T20:25:31-03:00
#layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["first"]
author: "Faioli a.k.a 0xttfx"
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

RAM é um hardware valioso e caro bem como a sua latência é ainda mais importante que a latência do disco. E por isso, o kernel Linux tenta ao máximo otimizar os uso da memória, fazendo uso de técnicas como compartilhando de páginas entre processos e Page Cache para melhorar a velocidade de I/O de armazenamento, armazenando um subconjunto de dados do disco na memória.
 
O Page Cache realiza compartilhamento implícito de memória e de forma assíncrona com o armazenamento em segundo plano! Isso por sí só, traz ainda mais complexidade à estimativa de uso de memória por parte dos administradores!


 ## Mas o que é Page Cache

o Page Cache faz parte do [Virtual File System - VFS](https://en.wikipedia.org/wiki/Virtual_file_system) que é uma camada de software do núcleo que trata de todas as chamadas de sistema relacionadas a um sistema de arquivos Unix. 

Sua principal vantagem é prover uma interface genérica para diversos tipos de sistemas de arquivos. Ou seja, o VFS permite que chamadas de sistemas genéricas, tais como open( ) e read( ),possam ser executadas independentemente do sistema de arquivos usados ou do meio físico! O que implica diretamente na latência de I/O das operações de leitura e gravação.

- Quando um sistema grava dados no cache, em algum momento também deve gravar esses dados no armazenamento. O tempo dessa gravação é controlado pelo que é conhecido como *write policy*, e existem duas abordagens básicas de escrita: 
	- *Write-through*: a gravação é feita de forma síncrona tanto no cache quanto no armazenamento de apoio. 
	- *Write-back*: inicialmente, a escrita é feita apenas no cache. A gravação no armazenamento de apoio é adiada até que o conteúdo modificado esteja prestes a ser substituído por outro bloco de cache.
    
		- Nesse [link você pode ter mais informações sobre o algorítimo](https://en.wikipedia.org/wiki/Cache_%28computing%29#Writing_policies)

Page é a unidade de memória que o Kernel trabalha com Page Cache, e geralmente possui 4k de comprimento mínimo de armazenamento no Page Cache. 
- Dessa forma todo I/O de arquivo está alinhado a uma quantidade específica de páginas...

Até aqui, podemos entender que o Page Cache, é o principal cache de disco usado pelo kernel do Linux e é utilizado ao ler ou gravar no disco quando novas páginas são adicionadas ao Page Cache para satisfazer as solicitações de leitura dos processos do *User Mode stack*. 
- Se a página ainda não estiver no cache, uma nova entrada será adicionada ao cache e preenchida com os dados lidos do disco. 
    - Se houver memória livre suficiente, a página é mantida no cache por um período indefinido e pode então ser reutilizada por outros processos sem acessar o disco.

Da mesma forma, antes de gravar page de dados em um dispositivo de bloco, o kernel verifica se a page correspondente já está incluída no cache; caso contrário, uma nova entrada é adicionada ao cache e preenchida com os dados a serem gravados no disco. 
   - A transferência de dados de I/O não começa imediatamente: 
    - a atualização do disco é atrasada por alguns segundos, dando assim aos processos a chance de modificar ainda mais os dados a serem gravados 
        - em outras palavras, o kernel implementa operações de *deferred write*

As páginas incluídas no Page Cache podem ser os seguintes tipos:

 - Pages contendo dados de arquivos regulares; no Capítulo 16, descrevemos como o kernel lida com operações de leitura, gravação e mapeamento de memória neles.
 - Pages contendo diretórios; o Linux lida com os diretórios de forma muito semelhante aos arquivos normais.
 - Pages contendo dados lidos diretamente de arquivos de dispositivos de bloco (ignorando a camada do sistema de arquivos); o kernel os trata usando o mesmo conjunto de funções como para pages contendo dados de arquivos regulares.
 - Pages contendo dados de processos do *User Mode stack* que foram trocados em disco; o kernel pode forçar a manter-se armazenado em page cache algumas pages cujo conteúdo já foi escrito em uma área de swap.
 - Pages pertencentes a arquivos de sistemas de arquivos especiais, como o sistema de arquivos especial *shm* usado para região de memória compartilhada de comunicação entre processos (IPC).

Como podemos ver, cada page incluída no Page Cache contém dados pertencentes a algum arquivo. Este arquivo – ou mais precisamente o inode do arquivo – é chamado de *page’s owner*.

Praticamente todas as operações read() e write() de arquivos dependem do Page Cache. 
 - a única exceção ocorre quando um processo abre um arquivo com a flag O_DIRECT definido: neste caso, o cache da página é ignorado e as transferências de dados de E/S fazem uso de buffers no espaço de endereço do modo de usuário do processo.
   - Várias aplicações database fazem uso da flag O_DIRECT pra que assim possam faze uso do próprio algorítimo de caching...

Os projetistas do kernel implementaram o Page Cache para atender a dois requisitos principais: 

 - Localizar rapidamente um page específica contendo dados relativos a um determinado proprietário. 
   Para aproveitar ao máximo o cache da página, pesquisá-lo deve ser uma operação muito rápida. 
 - Acompanhar como cada page do cache deve ser tratada ao ler ou escrever seu conteúdo. 
   - Por exemplo, a leitura de uma page de um arquivo regular, ou de um arquivo de dispositivo de bloco ou de uma área de swap, deve ser realizada de diferentes maneiras, portanto o kernel deve selecionar a operação adequada dependendo do proprietário da página. 

A unidade de informação mantida no page cache é, obviamente, uma página inteira de dados.
 - uma página não contém necessariamente blocos de disco fisicamente adjacentes, portanto ela não pode ser identificada por um número de dispositivo e um número de bloco. 
   - Em vez disso, uma página no Page Cache é identificada por um proprietário e por um índice nos dados do proprietário – geralmente, um inode e um offset dentro do arquivo correspondente.
 

A estrutura de dados principal do Page Cache é o objeto *address_space*, uma estrutura de dados incorporada no objeto inode proprietário da page.
 -  Muitas pages no cache podem referir-se ao mesmo proprietário, portanto, podem estar vinculadas ao mesmo objeto *address_space* que  também estabelece uma ligação entre a pages do proprietários e um conjunto de métodos que operam nessas pages. 
     - Aqui uma uma exceção ocorre para páginas que foram trocadas. Essas páginas possuem um objeto address_space comum não incluído em nenhum inode.
 
Cada descritor de página inclui dois campos chamados mapping e index, que vinculam a page ao Page Cache
 
 - O primeiro campo aponta para o objeto *address_space* do inode proprietário da page. 
 - O segundo campo especifica o *offset*  em unidades de *page-size* dentro do *addres_space*! 
	 - Ou seja: a posição dos dados da page dentro do *disk image*  proprietário.
	
Esses dois campos são usados ao procurar uma página no cache de páginas. 

Magitécnicamente, o cache de páginas pode conter múltiplas cópias dos mesmos dados do disco. Por exemplo, o mesmo bloco de dados de 4 KB de um arquivo normal pode ser acessado das seguintes maneiras: 
- Lendo o arquivo; os dados são incluídos em uma page pertencente ao inode do arquivo normal.
- Lendo o bloco do arquivo do dispositivo (partição do disco) que hospeda o arquivo;
	- os dados são incluídos em uma page pertencente ao *master inode* do arquivo do dispositivo de bloco.

Por isso, os dados de um disco aparecem em duas pages diferentes cada uma referenciada por um objeto address_space diferente...

 

Abaixo temos a tabela com os campos de um objeto *adress_space*

| Type | Field | Description | 
| --- | --- | --- |
| struct inode * | host | Pointer to the inode hosting this object, if any |
| struct radix_tree_root | page_tree | Root of radix tree identifying the owner’s pages |
| spinlock_t | tree_lock | Spin lock protecting the radix tree |
| unsigned int | i_mmap_writable | Number of shared memory mappings in the address space |
| struct prio_tree_root | i_mmap | Root of the radix priority search tree  | 
| struct list_head | i_mmap_nonlinear | List of non-linear memory regions in the address space |
| spinlock_t | i_mmap_lock | Spin lock protecting the radix priority search tree |
| unsigned int | truncate_count | Sequence counter used when truncating the file |
| unsigned long | nrpages | Total number of owner’s pages |
| unsigned long | writeback_index | Page index of the last write-back operation on the owner’s pages |
| struct address_space_ operations * | a_ops | Methods that operate on the owner’s pages |
| unsigned long | flags | Error bits and memory allocator flag |
| struct backing_dev_info * | backing_dev_info | Pointer to the backing_dev_info of the block device holding the data of this owner |
| spinlock_t | private_lock | Usually, spin lock used when managing the private_list list |
| struct list head | private_list | Usually, a list of dirty buffers of indirect blocks associated with the inode |
| struct address_space * | assoc_mapping | Usually, pointer to the address_space object of the block device including the indirect blocks |

Se o owner de uma page  no page cache for um arquivo, o objeto *address_space* será inserido no campo i_data de um objeto VFS inode. 
- O campo *i_mapping* do inode sempre aponta para o objeto *address_space* do proprietário das pages que contêm os dados do inode. 
    - O campo *host* do objeto *address_space* aponta para o objeto inode no qual o descriptor está embutido...

Por isso, se uma page pertence a um arquivo armazenado em um sistema de arquivos Ext3,
- o proprietário da page é o inode do arquivo e o objeto *address_space* correspondente é armazenado no campo *i_data* do objeto VFS inode. 
    - O campo *i_mapping* do inode aponta para o campo *i_data* do mesmo inode, e o campo *host* do objeto *address_space* aponta para o mesmo inode...

Mas como sempre: A "treta" está sempre presente na TI... :)

Se uma page contém dados, lidos de um arquivo de dispositivo de bloco(onde está o dado bruto(RAW)),  o objeto *address_space* é incorporado no *master inode* do arquivo no sistema de arquivos especial *bdev* associado ao dispositivo de bloco.
- Por isso, o campo *i_mapping* de um inode de um arquivo de dispositivo de bloco, aponta para o objeto *address_space* embutido no *master inode* 
    - da mesma forma, o campo *host* do objeto *address_space* aponta também para o *master idone*
        - dessa forma, todas as pages contendo dados lidos de um dispositivo de bloco possuem o mesmo objeto *address_space*, 
            - mesmo que tenham sido acessadas de arquivos de dispositivos de bloco diferentes


Os campos *i_mmap*, *i_mmap_writable*, *i_mmap_nonlinear* e *i_mmap_lock* referem-se ao mapeamento de memória e ao mapeamento reverso.

O campo *backing_dev_info* aponta o descritor *backing_dev_info* associado ao dispositivo de bloco que armazena os dados do proprietário.
- a estrutura *backing_dev_info* geralmente é incorporada no descritor da fila de solicitações do dispositivo de bloco.

O campo *private_list* é o cabeçalho de uma lista genérica que pode ser usada livremente pelo sistema de arquivos para seus propósitos específicos. 
- O Ext2 faz uso desse campo coletar os buffers sujos de blocos “indiretos” associados ao inode.
     - "buffers sujos" são dados ainda não escritos em disco.
     - Quando uma operação força o inode a ser gravado em disco, o kernel também libera todos os buffers nesta lista.

Um campo crucial do objeto *address_space* é *a_ops*.
- ele aponta para uma tabela do tipo *address_space_operations* contendo os métodos que definem como as pages dos proprietários são tratadas.
    - Os métodos mais importantes são:
        - readpage
        - writepage 
        - prepare_write 
        - commit_write

Os métodos vinculam os proprietários do objetos inode  aos drivers de baixo nível que acessam os dispositivos físicos. 
- Por exemplo:
   - a função que implementa o método *readpage* para um inode de um arquivo regular, sabe localizar as posições no dispositivo de disco físico dos blocos correspondentes a cada page do arquivo...

Abaixo podemos ver a tabela de métodos do *address_space*

| Method | Description |
| --- | --- |
| writepage | Write operation (from the page to the owner’s disk image) |
| readpage | Read operation (from the owner’s disk image to the page) |
| sync_page | Start the I/O data transfer of already scheduled operations on owner’s pages |
| writepages | Write back to disk a given number of dirty owner’s pages |
| set_page_dirty | Set an owner’s page as dirty |
| readpages | Read a list of owner’s pages from disk |
| prepare_write | Prepare a write operation (used by disk-based filesystems) |
| commit_write | Complete a write operation (used by disk-based filesystems) |
| bmap | Get a logical block number from a file block index | 
| invalidatepage | Invalidate owner’s pages (used when truncating the file) |
| releasepage | Used by journaling filesystems to prepare the release of a page |
| direct_IO | Direct I/O transfer of the owner’s pages (bypassing the page cache) |





>[!NOTE]
>Referencias: Daniel P. Bovet, Marco Cesati - Understanding the Linux Kernel, Third Edition-O'Reilly Media | Page Descriptors - Cap 8, Block Device Drivers - Cap 14 , The Page Cache - Cap 15, Accessing Files - Cap 16 , Page Frame Reclaiming - Cap 17 , The Ext2 and Ext3 Filesystems - Cap 18.




Até aqui já conseguimos entender que o mecanismo de cachear arquivos em memória de forma transparente, independente de se estar mapeando algo em memória, lendo ou escrevendo em disco, deixam as coisas um pouco mais complicadas para um administrador inexperiente ao tentar obter uma aferição de consumo de memória...  

Por isso, vamos tentar algumas abordagens para determinar valores mais próximos do real para o consumo de memória RAM.


# RSS e VSZ

 Vamos começar tendo como referencia o **VSZ** que é o tamanho da memória virtual que o Linux concedeu a um processo. Mas não necessariamente o processo está usando o valor informado. Um dos motivos são programas com funções para realizar determinadas tarefas, mas só as carregam na RAM, quando necessário. Bem como também a paginação por demanda do Linux, que só carrega páginas na memória quando o aplicativo tenta usá-las...  Por isso a leitura que devemos fazer do valor é: *memória se carregadas todas as suas funções e bibliotecas na memória física.*

 Já **RSS**, é o tamanho do conjunto residente. É a quantidade de RAM que o processo no momento para carregar as suas páginas. Ainda assim não podemos tomar essa informação como concreta, devido as bibliotecas compartilhadas torna-se um  valor impreciso por superestimativa...


 1. Primeiro criamos um processo em um novo cgroup 
    - Rodamos o comanod *mtr* utilizando o `systemd-run` para isolá-lo num cgroup(por que sim...rs)
      ```bash
      systemd-run --user -P -t -G --wait mtr 8.8.8.8
      ```
 
 2. Em seguida coletamos o seu PID e verificamos os valores de RSS e VSZ
    - Com o comando `ps` coletamos seu PID e os dados de RSS e VSZ
      - O PID do processo
        ```bash
        $ ps -aux |grep -E systemd-run.*mtr | grep -v grep |awk '{print $2}'
        236341
        ```
      - E os dados de RSS e VSZ  
        ```bash
        $  ps -o rss,vsz,cmd -p $(ps -aux |grep -E systemd-run.*mtr | grep -v grep |awk '{print $2}')
          RSS    VSZ CMD
         7168  14940 systemd-run --user -P -t -G --wait mtr 8.8.8.8
        ``` 
        - Só a título de cuiriosidade, segue algumas formas de coverter o valor para megabytes:
          ```bash
          $ echo $((7168/1024))
          7

          ou

          $ echo $((7168/1024)) |xargs -i printf "%'.1f MB" {}
          7.0 MB

          ou

          $ numfmt --from=si --to=iec 7168K
          6.9M

          ou

          $ numfmt --from=si --to-unit=1Mi --grouping 7168K
          7
          ```
    
    Podemos ver que o processo está consumindo **7168 Kilobytes**.

 3. Em seguida usamos o **procfs** para ter mais detalhes desse uso de RAM que o RSS está apontando.
    - Vamos ver o arquivo `smaps_rollup` que é uma soma das áreas de memória do `smaps` do nosso PID
      ```bash
      $ cat /proc/236341/smaps_rollup 
      559517c38000-7ffe5c398000 ---p 00000000 00:00 0                          [rollup]
      Rss:                7368 kB
      Pss:                1171 kB
      Pss_Dirty:           868 kB
      Pss_Anon:            868 kB
      Pss_File:            303 kB
      Pss_Shmem:             0 kB
      Shared_Clean:       6444 kB
      Shared_Dirty:          0 kB
      Private_Clean:        56 kB
      Private_Dirty:       868 kB
      Referenced:         7368 kB
      Anonymous:           868 kB
      LazyFree:              0 kB
      AnonHugePages:         0 kB
      ShmemPmdMapped:        0 kB
      FilePmdMapped:         0 kB
      Shared_Hugetlb:        0 kB
      Private_Hugetlb:       0 kB
      Swap:                  0 kB
      SwapPss:               0 kB
      Locked:                0 kB
      ```
      Para as métricas listadas
      - **RSS** que já é nossa conhecida.
      - **PSS** Proportional Set Size é o compartilhamento proporcional de memória do processo. É a contagem de páginas que ele possui na memória, onde cada página é dividida pelo número de processos que a compartilham. Portanto, se um processo tiver 1.000 páginas só para ele e 1.000 compartilhadas com outro processo, seu PSS será 1.500.
      - **Shared_Clean** aqui vemos que nosso processo usa cache de página. E representa o maior uso da memória. No arquivo *smaps*, conseguimos ver todas as bibliotecas compartilhadas que foram abertas com mmap() e residem no Page Cache.
      - **Shared_Dirty** quando o processo grava em arquivos com mmap(), esta linha mostra a quantidade de memória suja do cache de página ainda não salva.
      - **Referenced** é a quantidade de memória referenciada ou acessada até o momento. O valor é sempre igual ou próximo RSS.
      - **Anonymous** mostra a quantidade de memória que não pertence a nenhum arquivo.
      
   Até qui podemos ver que, embora o comando PS e Top nos mostre um RSS de 7MiB, a maior parte de seu RSS está oculto no cache de página e que quando ficarem inativas por um tempo, essas páginas, serão removidas da RAM pelo kernel.
    
   [Nesse artigo](https://lwn.net/Articles/642202/) do LWN.net temos mais informações direto da fonte ;). 


##### Continua em breve...













