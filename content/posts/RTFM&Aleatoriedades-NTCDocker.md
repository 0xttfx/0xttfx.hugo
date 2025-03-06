---

title: "RTFM & Aleatoriedades - Network Trobleshooting Container Docker"
date: 2024-01-06T14:55:51-03:00
tags: ["Docker"]
author: "Faioli"
#toc: true
hideSummary: false
comments: true
showComments: true
showDate: true
showTitle: true
showShare: true
disableShare: false
norss: false
nosearch: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

O intuito desse how to é apoiar na solução de problemas rede de containers Docker


## Disclaimer 
Se caiu aqui no momento em que está tentando apagar um incendio: você está no lugar errado! 
- Mas retorne depois do incendio .. ;)


## Requeriments
Para fazer um debug descente, precisamos estar familiarizados com questões básicas! 

### Como as coisas "funfam" dabaixo do capô da containerização

[Namespaces](https://man7.org/linux/man-pages/man7/namespaces.7.html) Linux fornecem as tecnologias fundamentais da implementação de containers. Fornecendo isolamento de recursos globais entre processos independentes

- [Namespaces](https://lwn.net/Articles/766124/) fornece isolamento e não restrição ao hardware adjacente! Isso é papel do [cgroups](https://docs.kernel.org/admin-guide/cgroup-v2.html)

São 8 namespaces até o momento

1. [Mount](https://man7.org/linux/man-pages/man7/mount_namespaces.7.html) - Mount points
	- cria uma hierarquia de diretórios isolado do sistema de arquivos do host, visivel apenas pelo processo em execução nessa árvore de diretórios do namespace.
1. [UTS](https://man7.org/linux/man-pages/man7/uts_namespaces.7.html) - Hostname and NIS domain  name
	- cria isolamento dos identificadores  hostname e o NIS domain name que são definidos
       usando [sethostname(2)](https://man7.org/linux/man-pages/man2/sethostname.2.html), [setdomainname(2)](https://man7.org/linux/man-pages/man2/setdomainname.2.html) e pode ser recuperado usando [uname(2)](https://man7.org/linux/man-pages/man2/uname.2.html) , [gethostname(2)](https://man7.org/linux/man-pages/man2/gethostname.2.html) e [getdomainname(2)](https://man7.org/linux/man-pages/man2/getdomainname.2.html) .
1. [IPC](https://man7.org/linux/man-pages/man7/ipc_namespaces.7.html) - System V IPC, POSIX message queues 
    - isolam  [sysvipc(7)](https://man7.org/linux/man-pages/man7/sysvipc.7.html) - System V Objetos IPC e [mq_overview(7)](https://man7.org/linux/man-pages/man7/mq_overview.7.html) - POSIX filas de mensagens. Dessa forma, o namespace tem seus identificadores IPC e filas de mensageria POSIX próprios, vistos apenas pelos processos  que executam nele.
1. [PID](https://man7.org/linux/man-pages/man7/pid_namespaces.7.html) - Process IDs
    - Cria espaço do número de ID do processo isolados do host. Permite que o processo containerizado faça uso do recurso bem legal, o [Freezing of tasks](https://www.kernel.org/doc/html/next/power/freezing-of-tasks.html) que permite  suspender um conjunto de processos de um contêiner e migralos para outro container, em outro host,  enquanto os processos internos mantêm os mesmos PIDs... 
         - agradeça ao Freezing tasks pelas mágicas no seu notebook ao hibernar...  
1. [Network](https://man7.org/linux/man-pages/man7/network_namespaces.7.html) - Network devices, stacks, ports,  etc.
    - fornecem isolamento dos dispositivos de rede, pilhas de protocolos IP v4 e v6, tabelas de roteamento IP, regras de firewall, os diretórios  _/proc/net_ (link para _/proc/_ pid _/net_ ), _/sys/class/net_ , arquivos em _/proc/sys/net_ , soquetes etc... Bem como também dos soquetes abstratos do domain [unix(7)](https://man7.org/linux/man-pages/man7/unix.7.html).
        - Tanto uma interface de rede física, quanto uma [veth(4)](https://man7.org/linux/man-pages/man4/veth.4.html) 
1. [User](https://man7.org/linux/man-pages/man7/user_namespaces.7.html) - User and group IDs
    - Faz isolamento dos recursos [credentials(7)](https://man7.org/linux/man-pages/man7/credentials.7.html) que são os identificadores relacionados à segurança e atributos, em particular, IDs de usuário e IDs de grupo, o diretório raiz, [keyrings(7)](https://man7.org/linux/man-pages/man7/keyrings.7.html)  e [capabilities(7)](https://man7.org/linux/man-pages/man7/capabilities.7.html).
2. [Cgroup](https://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html) - Cgroup root directory
    - Faz o isolamento virtualizando a visão do [cgroups(7)](https://man7.org/linux/man-pages/man7/cgroups.7.html) por um processo via /proc/pid/cgroup e /proc/pid/mountinfo. Dessa forma o namespace possui seu próprio conjunto de diretórios raiz cgroup, que são os caminhos relativos dos registros correspondentes no arquivo /proc/pid/cgroup, quando um processo cria um novo namespace usando [clone(2)](https://man7.org/linux/man-pages/man2/clone.2.html) ou [unshare(2)](https://man7.org/linux/man-pages/man2/unshare.2.html) com a flag **CLONE_NEWCGROUP**...
3. [Time](https://man7.org/linux/man-pages/man7/time_namespaces.7.html) - Boot and monotonic clocks
     - O time afeta afeta várias APIs como o [clock_gettime(2)](https://man7.org/linux/man-pages/man2/clock_gettime.2.html), [clock_nanosleep(2)](https://man7.org/linux/man-pages/man2/clock_nanosleep.2.html), [nanosleep(2)](https://man7.org/linux/man-pages/man2/nanosleep.2.html), [timer_settime(2)](https://man7.org/linux/man-pages/man2/timer_settime.2.html), [timerfd_settime(2)](https://man7.org/linux/man-pages/man2/timerfd_settime.2.html) e /proc/uptime, fazendo a virtualização isolada dos relógios de sistema:
         - **CLOCK_MONOTONIC** (e também **CLOCK_MONOTONIC_COARSE** e **CLOCK_MONOTONIC_RAW** ), um relógio não configurável que representa um  tempo monótono desde **então**(conforme descrito por POSIX - "algun ponto não especificado no passado").
         - **CLOCK_BOOTTIME** (e também **CLOCK_BOOTTIME_ALARM** ), um relógio não configurável que é idêntico a **CLOCK_MONOTONIC** , exceto que também inclui qualquer momento em que o sistema está suspenso.

Dessa forma já conseguimos tridimensionalizar mentalmente que:

- A coisa toda é meio que um processo containerizado por recursos que o fazem rodar em um diretório onde existirá um sistema de arquivos, fazendo com que o processo o veja como  o uma arvore filesystem, com seus pid/gid e pilha de rede, hostname e domainname / nisdomainname bem como as chamadas IPC e relógios de sistema isolados do host hospedeiro..
  - Aqui vale lembrar que esses namespaces não são de propriedade do processo! Eles operam de forma independente
     - qualquer outro processo pode ser containerizado nesses namespaces já existentes, dessa forma compatilhando-os... E de forma independente
         - pois você pode executar um processo dentro de um namespace Network, sem que ele tenha um Mount e UTS por exemplo
             - isso é massa porque é possível usar um comando  que existe no Host, porém a execução será dentro de um namespace!
                 - por isso, não é necessário instalar uma tool em um Mount de um processo containerizado para fazer teste ou debug...


O que nos interessa é o namespace Network! Por isso, apesar de eu explorar outros pontos, nesse nivelamento: network é o nosso foco


###### continua...

---
<script src="https://giscus.app/client.js"
        data-repo="0xttfx/0xttfx.github.io"
        data-repo-id="R_kgDOK3wAHw"
        data-category="BlogPostComments"
        data-category-id="DIC_kwDOK3wAH84Cnmtb"
        data-mapping="pathname"
        data-strict="1"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="top"
        data-theme="preferred_color_scheme"
        data-lang="en"
        data-loading="lazy"
        crossorigin="anonymous"
        async>
</script>

