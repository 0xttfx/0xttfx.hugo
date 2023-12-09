---
title: "WhoRAM"
date: 2023-11-15T20:25:31-03:00
# weight: 1
# aliases: ["/first"]
tags: ["first"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
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
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

 RAM é um hardware valioso e caro bem como a sua latência é ainda mais importante que a latência do disco. E por isso, o kernel Linux tenta ao máximo otimizar os uso da memória, fazendo uso de técnicas como compartilhando de páginas entre processos e Page Cache para melhorar a velocidade de I/O de armazenamento, armazenando um subconjunto de dados do disco na memória.
 O Page Cache realiza compartilhamento implícito de memória e de forma assíncrona com o armazenamento em segundo plano! Isso por sí só, traz ainda mais complexidade à estimativa de uso de memória por parte dos administradores!


 ## Mas o que é Page Cache

 o Page Cache faz parte do [Virtual File System - VFS](https://en.wikipedia.org/wiki/Virtual_file_system) que é uma camada de software do núcleo que trata de todas as chamadas de sistema relacionadas a um sistema de arquivos Unix. 
 Sua principal vantagem é prover uma interface genérica para diversos tipos de sistemas de arquivos. Ou seja, o VFS permite que chamadas de sistemas genéricas, tais como open( ) e read( ),possam ser executadas independentemente do sistema de arquivos usados ou do meio físico! O que implica diretamente na latência de I/O das operações de leitura e gravação.

 - Quando um sistema grava dados no cache, em algum momento ele também deve gravar esses dados no armazenamento. O tempo dessa gravação é controlado pelo que é conhecido como *write policy*, e existem duas abordagens básicas de escrita: 

   - *Write-through*: a gravação é feita de forma síncrona tanto no cache quanto no armazenamento de apoio. 
   - *Write-back*: inicialmente, a escrita é feita apenas no cache. A gravação no armazenamento de apoio é adiada até que o conteúdo modificado esteja prestes a ser substituído por outro bloco de cache.
     - Nesse [link você pode ter mais informações sobre o algorítimo](https://en.wikipedia.org/wiki/Cache_%28computing%29#Writing_policies)

 Page é a unidade de memória que o Kernel trabalha com Page Cache, e geralmente possui 4k de comprimento mínimo de armazenamento no Page Cache. Dessa forma todo I/O de arquivo está alinhado a uma quantidade específica de páginas...

 Até aqui, podemos entender que o Page Cache, é o principal cache de disco usado pelo kernel do Linux. Sendo utilizado ao ler ou gravar no disco; quando novas páginas são adicionadas ao cache de páginas para satisfazer as solicitações de leitura dos processos do Modo de Usuário. 
 - Se a página ainda não estiver no cache, uma nova entrada será adicionada ao cache e preenchida com os dados lidos do disco. Se houver memória livre suficiente, a página é mantida no cache por um período indefinido e pode então ser reutilizada por outros processos sem acessar o disco.

 - Da mesma forma, antes de gravar uma página de dados em um dispositivo de bloco, o kernel verifica se a página correspondente já está incluída no cache; caso contrário, uma nova entrada é adicionada ao cache e preenchida com os dados a serem gravados no disco. 
   - A transferência de dados de I/O não começa imediatamente: a atualização do disco é atrasada por alguns segundos, dando assim aos processos a chance de modificar ainda mais os dados a serem gravados (em outras palavras, o kernel implementa operações de deferred write).

 As páginas incluídas no Page Cache podem ser dos seguintes tipos:

 - Pages contendo dados de arquivos regulares; no Capítulo 16, descrevemos como o kernel lida com operações de leitura, gravação e mapeamento de memória neles.
 - Pages contendo diretórios; o Linux lida com os diretórios de forma muito semelhante aos arquivos normais.
 - Pages contendo dados lidos diretamente de arquivos de dispositivos de bloco (ignorando a camada do sistema de arquivos); o kernel os trata usando o mesmo
conjunto de funções como para pages contendo dados de arquivos regulares.
 - Pages contendo dados de processos do Modo Usuário que foram trocados em disco; o kernel pode ser forçado a manter-se armazenado em page cache algumas pages cujo conteúdo já foi escrito em uma área de swap.
 - Pages pertencentes a arquivos de sistemas de arquivos especiais, como o sistema de arquivos especial *shm* usado para região de memória compartilhada de comunicação entre processos (IPC).

 Como podemos ver, cada page incluída no Page Cache contém dados pertencentes a algum arquivo. Este arquivo – ou mais precisamente o inode do arquivo – é chamado de page’s owner.

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
   - Em vez disso, uma página no Page Cache é identificada por um proprietário e por um índice nos dados do proprietário – geralmente, um inode e um deslocamento dentro do arquivo correspondente.
 

 A estrutura de dados principal do cache de página é o objeto address_space, uma estrutura de dados incorporada no objeto inode que possui a página.* Muitas páginas no cache podem referir-se ao mesmo proprietário, portanto, podem estar vinculadas ao mesmo objeto address_space. Este objeto também estabelece uma ligação entre as páginas do proprietário e um conjunto de métodos que operam nessas páginas. Cada descritor de página inclui dois campos chamados mapeamento e índice, que vinculam a página ao cache de páginas (consulte a seção “Descritores de páginas” no Capítulo 8). O primeiro campo aponta para o objeto address_space do inode que possui a página. O segundo campo especifica o deslocamento em unidades de tamanho de página dentro do “espaço de endereço” do proprietário, ou seja, a posição dos dados da página dentro da imagem de disco do proprietário. Esses dois campos são usados ao procurar uma página no cache de páginas. Surpreendentemente, o cache de páginas pode conter múltiplas cópias dos mesmos dados do disco. Por exemplo, o mesmo bloco de dados de 4 KB de um arquivo normal pode ser acessado das seguintes maneiras: * Ocorre uma exceção para páginas que foram trocadas. Como veremos no Capítulo 17, essas páginas possuem um objeto address_space comum não incluído em nenhum inode. Este é o título do livro, edição eMatter Copyright © 2007 O’Reilly & Associates, Inc. Todos os direitos reservados. 602








 Dessa forma vamos tentar algumas abordagens para determinar valores mais próximos do real para o consumo de memória RAM.


# RSS e VSZ

 Vamos começar tendo como referencia o **VSZ** que é o tamanho da memória virtual que o Linux concedeu a um processo. Mas não necessariamente o processo está usando o valor informado. Um dos motivos são programas com funções para realizar determinadas tarefas, mas só as carregam na RAM, quando necessário. Bem como também a paginação por demanda do Linux, que só carrega páginas na memória quando o aplicativo tenta usá-las...  Por isso a leitura que devemos fazer do valor é: *memória se carregadas todas as suas funções e bibliotecas na memória física.*

 Já **RSS**, é o tamanho do conjunto residente. É a quantidade de RAM que o processo no momento para carregar as suas páginas. Ainda assim não podemos tomar essa informação como concreta, devido as bibliotecas compartilhadas torna-se um  valor impreciso por superestimativa...


 1. Primeiro criamos um processo em um novo cgroup 
    - Rodamos o comanod *mtr* utilizando o `systemd-run` para isolá-lo num cgroup(por que sim...rs)
      ```
      systemd-run --user -P -t -G --wait mtr 8.8.8.8
      ```
 
 2. Em seguida coletamos o seu PID e verificamos os valores de RSS e VSZ
    - Com o comando `ps` coletamos seu PID e os dados de RSS e VSZ
      - O PID do processo
        ```
        $ ps -aux |grep -E systemd-run.*mtr | grep -v grep |awk '{print $2}'
        236341
        ```
      - E os dados de RSS e VSZ  
        ```
        $  ps -o rss,vsz,cmd -p $(ps -aux |grep -E systemd-run.*mtr | grep -v grep |awk '{print $2}')
          RSS    VSZ CMD
         7168  14940 systemd-run --user -P -t -G --wait mtr 8.8.8.8
        ``` 
        - Só a título de cuiriosidade, segue algumas formas de coverter o valor para megabytes:
          ```
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
      ```
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


# PMAP

O kernel Linux 2.6.32 deu ao Linux um recurso chamado “Kernel SamePage Merging”. Isso significa que o Linux pode detectar regiões idênticas de dados em diferentes espaços de endereço. Imagine uma série de máquinas virtuais rodando em um único computador, e todas as máquinas virtuais rodando o mesmo sistema operacional. Usando um modelo de memória compartilhada e cópia na gravação, a sobrecarga no computador host pode ser drasticamente reduzida.

Tudo isso torna sofisticado o manuseio de memória no Linux. Mas é difícil observar um processo e saber qual é realmente o seu uso de memória.

O kernel expõe muito do que está fazendo com a RAM por meio de dois pseudoarquivos no pseudosistema de arquivos de informações do sistema “/proc”. Existem dois arquivos por processo, nomeados de acordo com o ID do processo ou PID de cada processo:* “/proc/$PID/maps” e “/proc/$PID/smaps”.

O valor [ anon ] é o mapeamento de memória anônimo, que faz parte da memória preenchida com dados não retirados do sistema de arquivos, mas alocados quando necessário.


:vulcan_salute:

* *Address:* The beginning memory address allocation
* *Kbytes:* Memory allocation in kilobytes
* *RSS:* Resident set size of the process in memory
* *Dirty:* The status of the memory pages
* *Mode:* Access mode and privileges
* *Mapping:* The user-facing name of the application or library
* *Offset:*	offset into the file
* *Device:*	device name (major:minor)

* *r:* The mapped memory can be read by the process.
* *w:* The mapped memory can be written by the process.
* *x:* The process can execute any instructions contained in the mapped memory.
* *s:* The mapped memory is shared, and changes made to the shared memory are visible to all of the processes sharing the memory.
* *R:* There is no reservation for swap space for this mapped memory. 

* *Address:* The start address of this mapping. This uses virtual memory addressing.
* *Perm:* The permissions of the memory.
* *Offset:* If the memory is file-based, the offset of this mapping inside the file.
* *Device:* The Linux device number, given in major and minor numbers. You can see the device numbers on your computer by running the lsblk command.
* *Inode:* The inode of the file the mapping is associated with. For example, in our example, this could be the inode that holds information about the pm program.
* *Size:* The size of the memory-mapped region.
* *KernelPageSize:* The page size used by the kernel.
* *MMUPageSize:* The page size used by the memory management unit.
* *Rss:* This is the resident set size. That is, the amount of memory that is currently in RAM, and not swapped out.
* *Pss:* This is the proportional share size. This is the private shared size added to the (shared size divided by the number of shares.)
* *Shared_Clean:* The amount of memory shared with other processes that has not been altered since the mapping was created. Note that even if memory is shareable, if it hasn't actually been shared it is still considered private memory.
* *Shared_Dirty:* The amount of memory shared with other processes that has been altered since the mapping was created.
* *Private_Clean:* The amount of private memory---not shared with other processes---that has not been altered since the mapping was created.
* *Private_Dirty:* The amount of private memory that has been altered since the mapping was created.
* *Referenced:* The amount of memory currently marked as referenced or accessed.
* *Anonymous:* Memory that does not have a device to swap out to. That is, it isn't file-backed.
* *LazyFree:* Pages that have been flagged as MADV_FREE. These pages have been marked as available to be freed and reclaimed, even though they may have unwritten changes in them. However, if subsequent changes occur after the MADV_FREE has been set on the memory mapping, the MADV_FREE flag is removed and the pages will not be reclaimed until the changes are written.
* *AnonHugePages:* These are non-file backed "huge" memory pages (larger than 4 KB).
* *ShmemPmdMapped:* Shared memory associated with huge pages. They may also be used by filesystems that reside entirely in memory.
* *FilePmdMapped:* The Page Middle Directory is one of the paging schemes available to the kernel. This is the number of file-backed pages pointed to by PMD entries.
* *Shared_Hugetlb:* Translation Lookaside Tables, or TLBs, are memory caches used to optimize the time taken to access userspace memory locations. This figure is the amount of RAM used in TLBs that are associated with shared huge memory pages.
* *Private_Hugetlb:* This figure is the amount of RAM used in TLBs that are associated with private huge memory pages.
* *Swap:* The amount of swap being used.
* *SwapPss:* The swap proportional share size. This is the amount of swap made up of swapped private memory pages added to the (shared size divided by the number of shares.)
* *Locked:* Memory mappings can be locked to prevent the operating system from paging out heap or off-heap memory.
* *THPeligible:* This is a flag indicating whether the mapping is eligible for allocating transparent huge pages. 1 means true, 0 means false. Transparent huge pages is a memory management system that reduces the overhead of TLB page lookups on computers with a large amount of RAM.
* *VmFlags:* See the list of flags below.
* *Mapping:* The name of the source of the mapping. This can be a process name, library name, or system names such as stack or heap.

The VmFlags - will be a subset of the following list.

* *rd:* Readable.
* *wr:* Writeable.
* *ex:* Executable.
* *sh:* Shared.
* *mr:* May read.
* *mw:* May write.
* *me:* May execute.
* *ms:* May share.
* *gd:* Stack segment grows down.
* *pf:* Pure page frame number range. Page frame numbers are a list of the physical memory pages.
* *dw:* Disabled write to the mapped file.
* *lo:* Pages are locked in memory.
* *io:* Memory-mapped I/O area.
* *sr:* Sequential read advise provided (by the madvise() function.)
* *rr:* Random read advise provided.
* *dc:* Do not copy this memory region if the process is forked.
* *de:* Do not expand this memory region on remapping.
* *ac:* Area is accountable.
* *nr:* Swap space is not reserved for the area.
* *ht:* Area uses huge TLB pages.
* *sf:* Synchronous page fault.
* *ar:* Architecture-specific flag.
* *wf:* Wipe this memory region if the process is forked.
* *dd:* Do not include this memory region in core dumps.
* *sd:* Soft dirty flag.
* *mm:* Mixed map area.
* *hg:* Huge page advise flag.
* *nh:* No huge page advise flag.
* *mg:* Mergeable advise flag.
* *bt:* ARM64 bias temperature instability guarded page.
* *mt:* ARM64 Memory tagging extension tags are enabled.
* *um:* Userfaultfd missing tracking.
* *uw:* Userfaultfd wr-protect tracking.



