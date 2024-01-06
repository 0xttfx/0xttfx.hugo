---
title: "RTFM & AleatoriedadesParaSysAdmin - Swap"
date: 2024-01-06T09:14:00-03:00
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

Swapping foi introduzida como um backup em disco para páginas não mapeadas. E existem três tipos de páginas que devem ser tratadas pelo subsistema de swapping:

- Pages que pertencem a uma região de memória anônima de um processo (User Mode stack)
- Dirty pages  que pertencem a um mapeamento de memória privada de um processo
- Pages que pertencem a uma região de memória compartilhada IPC

Numa "Regular Paging" cada entrada na "Page Table" inclui um sinalizador, uma "Present flag" e o Kernel explora esse sinalizador para sinalizar que uma página pertencente a um espaço de endereço de um processo qualquer foi "swapped out"! E além desse sinalizador, o Linux faz uso dos bits restantes do "Page Table" pra armazenar um identificador de "swapped-out page" que informa a localização no disco. 

A Principais características do subsistema swapping  são:

- Configura area swap para armazenamento de Pages que não possuem "disk image".
- Gerencia espaço na área de swap alocando e liberando "page slots".
- Fornece a função de "swap out" pages da RAM para a área de swap  e "swap in" pages da área de swap para RAM.
- Faz uso da “swapped-out page identifiers” das entradas na "Page Table" que foram swapped para acompanhar as posições dos dados na área de swap.

Em suma, o swapping é o principal recurso de "page frame reclaiming"! E se queremos ter certeza que todos os "page frames" obtidos por um processo, não apenas os "pages" que contem "disk image", possam ser recuperados pelo PFRA, devemos fazer uso do swapping...

Com isso podemos deduzir que grandes áreas de swap dão poder ao Kernel para iniciar vários processos onde o total de solicitações de memória ultrapassa a quantidade de RAM física.

E como na TI, nem tudo são flores, precisamos nos atentar que simular RAM em disco, nos traz um desempenho em milissegundos se comparado aos nanosegundos da RAM física ;) 

>[!NOTE]
>*Referencia: Daniel P. Bovet, Marco Cesati - Understanding the Linux Kernel - Capítulo 2 - Memory Addressing e Capítulo 17 - Page Frame Reclaiming*


Agora que nivelamos a introdução sobre swapping, vamos criar um espaço de troca, do tipo arquivo, que não muda em nada quando partição em disco: a não ser o processo de particionamento de disco etc...


1. Verificando se há algum swap

```
sudo swapon --show
```

2. Criando arquivo de swap de 2GiB

```
sudo dd if=/dev/zero of=/swapfile count=2192 bs=1MiB
```

3. Alterando permissão do arquivo

```
sudo chmod 600 /swapfile
```

4. Marcando o arquivo como um espaço de swap

```
sudo mkswap /swapfile
```

5. Verificando se está tudo ok

```
sudo swapon --show
```

6. Adicionando a linha no '/etc/fstab' para montagem automática

```
echo '/swapfile    none    swap    sw    0 0' | sudo tee -a /etc/fstab
```

7. Ajustando a configuração de Swap

O parâmetro `/proc/sys/vmwappiness` configura a frequência com que o sistema transfere dados da RAM para o espaço de swap. Sendo um  valor entre 0 e 100 que representa uma porcentagem! Um valor baixo significa que o seu sistema Linux troca processos raramente enquanto um alto valor significa que os processos são gravados em disco imediatamente

* Com valores próximos de zero, o kernel não irá transferir dados para o  disco a menos que seja absolutamente necessário.
* Lembre-se, as  interações com o arquivo de swap são “dispendiosas”! Pois são mais lentas que as interações com a RAM.
* Valores que estão mais próximos de 100 irão tentar colocar mais dados  no swap em um esforço para manter mais espaço da RAM livre.
	* Dependendo  do perfil de memória de seus aplicativos ou do motivo pelo qual você  está usando o seu servidor, isso pode ser melhor em alguns casos.

|     |     |
| --- | --- |
| Swappiness: 60 | Swap a partir de 40% de uso de RAM. |
| Swappiness: 40 | Swap a partir de 60% de uso de RAM. |
| Swappiness: 20 | Swap a partir de 80% de uso de RAM. |
| Swappiness: 10. | Swap a partir de 90% de uso de RAM. |
| Swappiness: 1 | Swap a partir de 99% de uso de RAM. |

Para um desktop, um valor de swappiness de 60 não é um valor ruim e normalmente é o valor dfault em uma distro Linux. Mas para um servidor, podemos deixá-lo mais próximo de 0, para fazer uso somente quando realmente necessário

Como exemplo podemos setar para  '10' para que o swap seja utilizado após 90% de RAM ocupada
```
sudo sysctl vm.swappiness=10
```
Para garantir que esse valor irá se manter após um boot
```
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

Mas como sempre, não há uma receita de bolo! Em determinados cenários  trocar processos de tempo de execução da RAM para o disco, deve ser evitado, noutros há vantagem... Conhecer a aplicação irá lhe guiar na configuração mais apropriada!

8. Ajustando a confiuração do vfs\_cache\_pressure

Esta opção controla a tendência do kernel de recuperar a memória que é usada para cache de diretórios e objetos inode.

* No valor padrão de vfs\_cache\_pressure=100, o kernel tentará recuperar dentries e inodes a uma taxa "justa" em relação ao pagecache e à recuperação do swapcache.
* Diminuir vfs\_cache\_pressure faz com que o kernel prefira manter os caches dentry e inode.
* Quando vfs\_cache\_pressure=0, o kernel nunca recuperará dentries e inodes devido à pressão de memória e isso pode levar facilmente a condições de falta de memória.
* Aumentar vfs\_cache\_pressure além de 100 faz com que o kernel prefira recuperar dentries e inodes.

Podemos definir isso em um valor mais conservador como 50
```
sudo sysctl vm.vfs_cache_pressure=50
```

Vamos garantir que após um boot o valor permaneça
```
echo 'vm.vfs_cache_pressure=50' | sudo tee -a /etc/sysctl.conf
```


