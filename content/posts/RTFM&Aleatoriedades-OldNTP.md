---
title: "RTFM & Aleatoriedades - Old NTP Client"
date: 2024-01-09T12:08:11-03:00
layout: "simple"
# weight: 1
# aliases: ["/first"]
tags: ["NTP","Linux","RTFM"]
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

Sempre existe a possibilidade de um dia aparecer um servidor com uma versão de SO não atual precisando que o NTP client seja configurado ou revisado… :)


1. Instalando o serviço NTP
	1. Instale o pacote NTP com o utilitário yum
	```bash
	$ yum install ntp  -y
	```
	2. Em seguida liste os comandos de administração fornecidos pelo pacote npt
	```
	$ ntp<tab><tab>
	ntpd        ntpdate     ntpdc       ntp-keygen  ntpq        ntpstat     ntptime
	```
	3. Inicialize o serviço 
	```bash
	$ sudo service ntpd start
	ou
	$ sudo /etc/rc.d/init.d/ntpd start
	```
	4. Ao iniciar o serviço, analise em seguida os logs no /var/log/messages
    ```bash
    $ sudo tail /var/log/messages
    ou
    $ grep ntpd /var/log/messages
    Jul 23 13:11:51 labcentos01 yum[9343]: Installed: ntpdate-4.2.4p83.el6.centos.x86_64
     Jul 23 13:12:11 labcentos01 ntpd[9375]: ntpd  Fri Feb 22 11:23:27 UTC 2013 (1)
     Jul 23 13:12:11 labcentos01 ntpd[9376]: precision = 0.111 usec
     Jul 23 13:12:11 labcentos01 ntpd[9376]: Listening on interface #0 wildcard...
     Jul 23 13:12:11 labcentos01 ntpd[9376]: Listening on interface #1 wildca...
     Jul 23 13:12:11 labcentos01 ntpd[9376]: Listening on interface #2 lo, ::1...
     Jul 23 13:12:11 labcentos01 ntpd[9376]: Listening on interface #3 eth0, ....
     Jul 23 13:12:11 labcentos01 ntpd[9376]: Listening on interface #4 lo, 127.0....
     ```
     5. Automatize o serviço para que inicie no boot:
     ```bash 
     $ sudo chkconfig --level 35 ntpd on
     ```

2. Configuração do arquivo padrão do serviço “/etc/ntp.conf” 
	1. Segue o arquivo padrão fornecido pelo pacote de instalação:
     ```bash
     # be tightened as well, but to do so would effect some of 
		# the administrative functions. 
		restrict 127.0.0.1 
		restrict -6 ::1 
		
		# Hosts on local network are less restricted. 
		#restrict 192.168.1.0 mask 255.255.255.0 nomodify notrap 
		
		# Use public servers from the pool.ntp.org project. 
		# Please consider joining the pool (http://www.pool.ntp.org/join.html). 
		server 0.centos.pool.ntp.org iburst 
		server 1.centos.pool.ntp.org iburst 
		 
		#broadcast 192.168.1.255 autokey        # broadcast server 
		#broadcastclient                        # broadcast client 
		#broadcast 224.0.1.1 autokey            # multicast server 
		#multicastclient 224.0.1.1              # multicast client 
		#manycastserver 239.255.254.254         # manycast server 
		#manycastclient 239.255.254.254 autokey # manycast client 
		
		# Enable public key cryptography. 
		#crypto 
		includefile /etc/ntp/crypto/pw 
		
		# Key file containing the keys and key identifiers used when operating 
		# with symmetric key cryptography. 
		keys /etc/ntp/keys 
	
		# Specify the key identifiers which are trusted. 
		#trustedkey 4 8 42 
		
		# Specify the key identifier to use with the ntpdc utility. 
		#requestkey 8 
		
		# Specify the key identifier to use with the ntpq utility. 
		#controlkey 8 
		
		# Enable writing of statistics records. 
		#statistics clockstats cryptostats loopstats peerstat
     ```
     2. Modifique as linhas “server”, substituindo pelo servidor NTP local; sendo uma linha para cada servidor... :
        ```bash
        server 0.redhat.pool.ntp.org iburst 
        server 1.redhat.pool.ntp.org iburst 
        ```
	     - Modificando para o NTP Brasileiro ou na LAN:
		  ```bash
		  server 192.0.2.151 iburst
		  ou
		  server a.st1.ntp.br iburst
		  ou
		  server a.ntp.br iburst
		  ```
	      - a flag *iburst* faz com que uma saraivada de mensagens sejam trocadas para preparar os dados e ajustar o relógio por cerca de 10s.
      3. Reinicie o serviço NTP      
     ```bash 
     sudo /etc/init.d/ntpd restart
     ```
     4. Agora visualize como ficou o arquivo de configuração do serviço NTP:     
     ```bash
     $ sudo cat /etc/ntp.conf |grep -E -v "^$|^#" |nl 
     1  driftfile /var/lib/ntp/drift
     2  restrict default kod nomodify notrap nopeer noquery
     3  restrict -6 default kod nomodify notrap nopeer noquery
     4  restrict 127.0.0.1
     5  restrict -6 ::1
     6  server 192.0.2.151 iburst
     7  includefile /etc/ntp/crypto/pw
     8  keys /etc/ntp/keys
     ```
     5. Outra opção interessante para habilitar no “ntp.conf” para a configuração cliente é a diretiva de estatísticas:
     ```bash
     # estatisticas do ntp que exibe informaões de funcionamento.
     statsdir /var/log/ntpstats/
     statistics loopstats peerstats 
     filegen loopstats file loopstats type day enable
     ilegen peerstats file peerstats type day enable
     ```
3. Verificando se o daemon NTP está devidamente configurado e operando:
    1. Execute o comando “ntpq” com a opção “-c peers” que lista estatísticas dos servidores cadastrados no arquivo /etc/ntp.conf. 
     ```bash
     $ sudo ntpq -c pe** 
     ou
     $ sudo ntpq -c peers**
     ```
     - se a resposta for algo parecido com a descrita abaixo, o daemon não está ativo
     ```bash
     localhost.localdomain: timed out, nothing received
     *****Request timed out**
     ```
     - se a resposta for próxima da descrita abaixo, o NTP está operando:
     ```bash
     remote        refid           st t when poll reach delay offset jitter
	 ==========================================================================
	 *192.0.2.151  200.186.125.195  2 u 84   1024 377   0.750 0.138  0.279
	 ```
	 - Segue mais um resultado de um NTP com mais de 1 servidor de horas configurado
	 ```bash
	 remote         refid     st  t  when  poll reach  delay	offset	jitter 
	 ============================================================================= 
	 +a.st1.ntp.br  .ONBR.    1   u  8     64	  1      11.929 -0.405	1.353 
	 201.49.148.135  .STEP.   16  u  -     64   0      0.000  0.000 	0.000 
	 *d.st1.ntp.br  .ONBR.    1   u  4     64   1      19.496 -0.530	0.392 
	 -a.ntp.br  200.160.7.186 2   u  4     64   1      11.061 0.117	  1.151 
	 +b.ntp.br  200.20.186.76 2   u  -     64   1      14.459 -2.561	2.544
	 ```
	 - *stratum 16 indica servidor inoperante.*
	 - Compreendendo cada campo
	     - *offset* - mostra em milissegundos o quanto a hora no servidor precisa adiantar ou se atrasar para  ficar igual a do servidor NTP  ( esses valores devem estar em milissegundos! Acima de segundos não é um bom resultado.)
	     - *delay* mostra o tempo que o pacote demora para ir ao servidor e voltar para o host.
	     - *jitter* mostra o quanto em milissegundos existe de variação nas medidas de deslocamento dos pacotes e, o quanto mais baixo melhor.
	     - *reach* mostra o resultado em octal das últimas 8 tentativas de conexão ao servidor preferencial de horas. O valor 	esperado é  “377” que indica que as últimas tentativas obtiveram sucesso! Valores diferentes indicam falhas  ocorridas. 
	        - Para melhor compreensão, é  um registrador de 8 bits que vai girando para a esquerda representado na forma octal, que mostra o resultado das últimas 8 consultas à fonte de tempo: 377 = 11.111.111 significa que todas as consultas foram bem sucedidas; 
	            - outros número indicam falhas, por exemplo 375 = 11.111.101, indica que a penúltima consulta falhou.
	    - *refid* mostra a referência (par do sistema) ao qual, o servidor de tempo remoto está sincronizado.
	    - *st* mostra o strato da fonte de tempo.
	    - *when* quanto segundos se passaram desde a última consulta à essa fonte de tempo.
	    - *poll* mostra de quantos em quantos segundos essa fonte é consultada.

     2. Execute-o novamente com a opção “-c rv” para visualizar o estado da hora local
     ```bash
     $ sudo ntpq -c rv
     assID=0 status=46f4 leap_add_sec, sync_ntp, 15 events, event_peer/strat_chg,
     version="ntpd 4.2.4p8@1.1612-o Thu Jan 10 15:17:40 UTC 2013 (1)",
     processor="x86_64", system="Linux/2.6.32-279.19.1.el6.x86_64", leap=01,
     stratum=3, precision=-24, rootdelay=15.668, rootdispersion=59.025,
     peer=14178, refid=192.0.2.151,
     reftime=d59b9d48.ef695513  Thu, Jul 25 2013  9:49:12.935, poll=10,
     clock=d59ba39e.61ba5ac8  Thu, Jul 25 2013 10:16:14.381, state=4,
     offset=0.073, frequency=7.577, jitter=0.051, noise=0.232,
     stability=0.018, tai=0
     ```
     - compreendendo os campos
         -  *version* mostra a versão do NTP e sua data e hora de construção. 
         - *processor* mostra a versão e plataforma do hardware.
         - *stratum*  que pode ir de 1 até 15 mostra a distância que o servidor NTP usado está do o relógio de referência que transmite a hora UCT, que por sua vez está em stratum-0!
            - Lembrando que quanto mais próximo o servidor estiver do stratum-0, menos atraso ele terá em realação ao UCT; ou seja quanto menor a distância melhor... Variável “peer” ID associado ao servidor NTP preferencial.
         - *reftime*  mostra a data e hora que o servidor preferencial foi atualizado pela última vez pelo seu stratum superior. 
         - *clock* data e hora do dia! Aquele que foi setado/configurado no seu servidor host.
         - *offset*  deslocamento, quanto o relógio local tem de ser adiantado ou atrasado para chegar à hora certa (horaigual à do estrato 0).
         - *precision* precisão indicada com o expoente de um número base 2 Variável “rootdisperion” erro máximo da medida de offset em relação ao strato 0, em milissegundos.
         - *rootdelay* atraso ou tempo de ida e volta dos pacotes atéo strato 0, em milissegundos.
         - *refid* o par do sistema, ou principal referência.
         - *frequency* erro na freqüência do relógio local, em relação à freqüência do estrato 0, em partes por milhão (PPM).
     
     3. Obtendo informações separadamente de cada servidor NTP configurado! Essa forma é utilizada por exemplo para medir as informações de um servidor NTP secundário afim de tomar alguma decisão como a de substituir o servidor preferencial problemático por algum outro do pool...
     - Primeiro liste o ID dos servidores NTP configurados com o comando “ntpq -c as”:
     ```bash
     $ sudo ntpq -c as
     ou
     sudo ntpq -c associations
     ind 	assID	status  conf	reach	auth	condition	last_event	cnt
     =====================================================================
     1 	  14178	9614    yes	  yes  	none	sys.peer	reachable	   1
     ```
     4.  Depois de obtido o ID obtenha as informações do servisor NTP entrando na linha de comando da ferramenta ntpq e utilize a opção “rv” seguido do ID.
     ```bash
     $ sudo ntpq  [enter]
     $ sudo ntpq> rv 14178
     assID=14178 status=9614 reach, conf, sel_sys.peer, 1 event, event_reach,
     srcadr=192.0.2.151, srcport=123, dstadr=192.0.2.140, dstport=123,
     leap=00, stratum=2, precision=-23, rootdelay=16.342,rootdispersion=16.281, 
     refid=200.186.125.195, reach=377, unreach=0,
     hmode=3, pmode=4, hpoll=10, ppoll=10, flash=00 ok, keyid=0, ttl=0,
     offset=0.345, delay=1.032, dispersion=14.833, jitter=0.124,
     reftime=d59c0533.3051b8eb  Thu, Jul 25 2013 17:12:35.188,
     org=d59c0552.e99941c5  Thu, Jul 25 2013 17:13:06.912,
     rec=d59c0552.e9a47a2b  Thu, Jul 25 2013 17:13:06.912,
     xmt=d59c0552.e95ea055  Thu, Jul 25 2013 17:13:06.911,
     filtdelay=     1.03    1.12    1.22    1.10    0.64    1.30    1.57    1.09,
     filtoffset=    0.34    0.22    0.00   -0.01    0.16   -0.41   -0.41   -0.32,
     filtdisp=      0.00   15.38   30.75   46.11   61.50   76.85   92.22  107.58
     ```

4. Caso precise gerar alguma evidência para documentação, etc... É possível gerar gráficos utilizando os logs de estatísticas  do diretório “/var/log/ntpstats”, que são:
 - *loopstats*, que apresenta as informações do loop local, ou seja, as variáveis do sistema.
 - formato
```bash
54475 73467.286 -0.000057852 31.695 0.000015298 0.006470 4
54475 73548.286 -0.000084064 31.688 0.000017049 0.006471 4
54475 73682.286 -0.000077221 31.678 0.000016130 0.006988 4
54475 73698.286 -0.000077448 31.677 0.000015103 0.006550 4
54475 73761.286 -0.000083230 31.672 0.000014275 0.006376 4
54475 73889.286 -0.000059100 31.665 0.000015846 0.006487 4
54475 74004.285 -0.000045825 31.660 0.000015548 0.006324 4
```
    -  campos:
	     - coluna 1: day
	     - coluna 2: second
	     - coluna 3: offset
	     - coluna 4: drift compensation
	     - coluna 5: estimed error
	     - coluna 6: stability
	     - coluna 7: polling interval
 - *peerstats* que apresenta as informações de cada associação
 - formato
```bash
 54475 34931.294 200.20.186.75 9074 0.009958844 0.008390600 0.000390895 0.000132755
 54475 34931.301 200.192.232.43 f0f4 0.000348814 0.015550265 0.001120348 0.000023645
 54475 34932.303 200.189.40.28 f0f4 0.000810708 0.017701986 0.188995109 0.000043145
 54475 34934.286 200.160.0.28 f0d4 0.000332344 0.000271801 0.000620139 0.000037467
 54475 34935.286 200.160.7.165 9614 0.000003557 0.000216088 0.000826694 0.000022076
```
	 - campos
		 - coluna 1: day
		 - coluna 2: second
		 - coluna 3: address
		 - coluna 4: status
		 - coluna 5: offset
		 - coluna 6: delay
		 - coluna 7: dispersion
		 - coluna 8: skew/variance
- Além de poder ser usado para documentação, também fica mais fácil a leitura com o gráfico para ilustrá-lo. Excell pode ser usado... 
	- Mas porque não usar terminal e [gnuplot](http://www.gnuplot.info) :).
		- segue algumas referências: [IBM](https://developer.ibm.com/tutorials/ba-cleanse-process-visualize-data-set-3/?mhsrc=ibmsearch_a&mhq=gnuplot)
	 - Segue um exemplo de uso:
	     - Crie um arquivo texto com um nome pretendido, com o seguinte conteúdo:
```bash
cat <<Fin> /tmp/deslocamento.txt
set term gifset output 'Deslocamento.png'
set title "Deslocamento"
plot "/var/log/ntpstats/loopstats" using 2:3 t"deslocamento" with linespoints lt rgb "#d82886";
Fin
```
- Estamos referenciando o arquivo loopstats e, fazendo uso das colunas 2 e 3! A coluna 2 indica o tempo, no dia, em segundos; e a coluna 3 indica o deslocamento, em milissegundos. Também estamos utilizando cores RGB declarada em [hexadecimal](http://www.colorhexa.com).
	 -  Agora geramos o gráfico
	 ```bash
	 $ gnuplot /tmp/deslocamento.txt
	 ```

![deslocamento.png](/images/NTP/deslocamento.png)
 

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

