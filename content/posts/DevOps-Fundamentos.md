---

title: "DevOps Fundamentos"
date: 2024-05-07T02:31:50-03:00
author: "Faioli a.k.a 0xttfx"
toc: true
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

# Princípios

![Untitled.png](/images/DevOps-Fundamentos/Untitled.png)

>[!important]
>O DevOps é mais do que apenas equipes de desenvolvimento e operações trabalhando juntas. É mais do que ferramentas e práticas. O DevOps é uma forma de pensar, uma mudança cultural, em que as equipes adotam novas formas de trabalhar.


Na cultura de DevOps, os desenvolvedores se aproximam dos usuários, obtendo uma compreensão melhor dos requisitos e necessidades deles.

As equipes de operações se envolvem no processo de desenvolvimento e adicionam requisitos de manutenção e necessidades do cliente.

Ela também envolve aderir aos princípios essenciais a seguir, que ajudam as equipes a oferecer aplicativos e serviços:

- em um ritmo mais rápido
- com maior qualidade de código
- e melhor qualidade de vida ;)



São 5 os princípios fundamentais do modelo DevOps

- **Colaboração**

    A principal premissa por trás do DevOps é a colaboração. As equipes de desenvolvimento e operações se unem em uma equipe funcional que se comunica, compartilha feedback e colabora durante todo o ciclo de desenvolvimento e implementação. Muitas vezes, quer dizer que as equipes de desenvolvimento e operações se fundem em uma única equipe que trabalha em todo o ciclo de vida do aplicativo.

    Os membros de uma equipe de DevOps são responsáveis por garantir entregas de qualidade em cada faceta do produto, levando a um desenvolvimento mais "full stack", em que as equipes têm as responsabilidades completas de back-end e front-end de um recurso ou produto. As equipes vão ser as donas de um recurso ou projeto durante todo o ciclo de vida, da ideia até a entrega. O nível aprimorado de investimento e atenção da equipe leva a uma produção de maior qualidade.

- **Automação**

    Uma prática essencial do DevOps é automatizar o máximo possível o ciclo de vida do desenvolvimento do software, dando aos desenvolvedores mais tempo para escrever código e desenvolver novos recursos. A automação é um elemento crucial de um pipeline de IC/CD e ajuda a reduzir erros humanos e aumentar a produtividade da equipe. Com processos automatizados, as equipes obtêm melhoria contínua com tempos de iteração curtos, o que permite responder com mais rapidez aos comentários dos clientes.

- **Implementação contínua**

    A melhoria contínua foi estabelecida como um elemento básico das práticas ágeis, bem como da fabricação lean e Kata de melhoria. É a prática de se concentrar na experimentação, minimizar o desperdício e otimizar a velocidade, o custo e a facilidade de entrega. A melhoria contínua também está ligada à entrega contínua, permitindo que as equipes de DevOps enviem atualizações constantes que melhoram a eficiência dos sistemas de software. O pipeline constante de novas versões significa que as equipes promovem com consistência mudanças de código que eliminam o desperdício, melhoram a eficiência do desenvolvimento e trazem mais valor ao cliente.

- **Ação voltada ao cliente**

    As equipes de DevOps usam ciclos curtos de feedback com clientes e usuários finais para desenvolver produtos e serviços centrados nas necessidades do usuário. As práticas de DevOps permitem coleta e resposta rápidas ao feedback do usuário por meio do uso de monitoramento em tempo real e implementação rápida. As equipes obtêm visibilidade imediata de como os usuários ativos interagem com um sistema de software e usam os dados para desenvolver melhorias adicionais.

- **Criar com o final em mente**

    Esse princípio envolve entender as necessidades dos clientes e criar produtos ou serviços que resolvam problemas reais. As equipes não devem "criar uma bolha" ou um software com base em suposições sobre como os consumidores vão usar o software. Como alternativa, as equipes de DevOps devem ter uma compreensão holística do produto, da criação à implementação.

# História

O movimento do DevOps começou por volta de 2007

- Foi num momento em que as operações de TI e comunidades de desenvolvimento de software levantaram preocupações sobre:

    - Um modelo de desenvolvimento de software além do tradicional!
    - O fato de que mesmo as metodologias ágeis terem sido adotadas para melhorar a colaboração: as melhorias ficaram restritas somente nas equipes de desenvolvimento.
    - E ao fato de que mesmo a infraestrutura sempre apoiando os desenvolvedores em fases necessárias as equipes Infra/Operação e Produto/Dev não tinha alinhamentos trabalhando assim de forma ineficientes

Dessa forma a solução dada foi o modelo **DevOps**

- Que na prática é a evolução dos métodos Ágeis, mas com a crucial evolução:
    - A conexão das equipes de Operação e Desenvolvimento
        - O DevOps reúne as habilidades, os processos e as ferramentas das equipes de desenvolvimento e operações para que trabalhem em conjunto!

# Benefícios

Podemos simplificar de forma bem clara 3 benefícios centrais do DevOps:

- benefícios técnicos,
    - Complexidade reduzida, entrega contínua e Resolução de problemas mais ágeis! Além de código de alta qualidade com mais rapidez com muito menos stress das equipes ;)
- benefícios culturais
    - Equipes mais produtivas e eficientes devido a melhor qualidade de vida e usuários/clientes mais satisfeitos como consequência natural
- benefícios de negócios.
    - Melhor colaboração e maior confiança entre os membros das equipes, resultando assim em entregas mais ágeis num ambiente operacional mais estável

# Cultura

O DevOps traz uma mudança cultural muito forte onde as equipes trabalham amparadas por métodos de:

- engenharia de software,
- fluxo de trabalho
- conjunto de ferramentas

O que de forma operacional coloca num único nível de importância: arquitetura, design e desenvolvimento!

As equipes passam a ter uma maior compreensão dos requisitos e necessidades dos usuários.

Os valores da transparência, comunicação e colaboração se elevam consideravelmente entre as equipes.

# Estrutura DevOps

## Framework CALMS

CALMS foi compartilhado pela primeira vez por Jez Humble no **The DevOps Handbook.** E o significado da sigla é:

- Cultura
- Automação
- Simplicidade
- Medição
- Compartilhamento

## **Topologias de equipe**

No livro, [“Team Topologies”](https://www.amazon.com/dp/B07VWYNGCQ?plink=vFQ5MJYpTDzGies3&ref=adblp13nvvxx_0_1_ti), Matthew Skelton e Manuel Pais definem quatro tipos fundamentais de equipes e o conceito de “mudança de fluxo”.

Entendendo o conceito do livro fica fácil entender porque não funciona colocar “DevOps” no nome da equipe ou no cargo para criar uma “Equipe de DevOps” ou um “Engenheiro de DevOps”.

Mas sim, em vez disso, as topologias de equipe ajudam a entender como suas práticas e ferramentas que se encaixam no quadro geral. Por isso primeiro passo para uma transformação de DevOps é identificar a estrutura organizacional em vigor e a partir dela idealizar a implementação do modelo DevOps.

## **Métricas DORA**

Os profissionais confiam em quatro métricas principais, desenvolvidas pela DORA, para medir a eficácia das práticas de DevOps.

- **Tempo de espera para alterações: qual é o tempo de espera** para alterações de código desde o momento em que o código é verificado até o ponto em que ele é liberado para produção?
- **Frequência de implementação:** com que frequência e com que rapidez você lança para produção?
- **Tempo de restauração do serviço:** quando um incidente é detectado, quanto tempo leva para corrigir e restaurar o serviço?
- **Taxa de falha de alteração:** com que frequência ocorrem falhas de implementação na produção que exigem correção ou rollback imediatos?

No projeto DORA, existe uma check rápido para medir o desempenho de entrega de software de equipe em menos de um minuto! E pode apoiar na implementação das métricas DORA ao comparar o resultado com de outras empresas no setor …

> [!info] DORA | DevOps Quick Check
> DevOps Research and Assessment (DORA) is a long running research program that seeks to understand the capabilities that drive software delivery and operations performance.
> [https://dora.dev/quickcheck/](https://dora.dev/quickcheck/)


A equipe de pesquisa DORA, também criou o Four Keys que permite coletar dados de seu ambiente de desenvolvimento (como GitHub ou GitLab) e compilá-los em um painel que exibe essas métricas principais

- Ele é desenhado para funcionar sem modificações em cloud GCP! Porém podemos fazer algumas modificações para fazer uso com outros fornecedores...

https://github.com/dora-team/fourkeys

> [!info] DORA 2022 Accelerate State of DevOps Report now out | Google Cloud Blog
> Security-enhancing DevOps practices are broadly adopted, this year’s DORA Accelerate State of DevOps Report found, but that’s not the whole story.
> [https://cloud.google.com/blog/products/devops-sre/dora-2022-accelerate-state-of-devops-report-now-out](https://cloud.google.com/blog/products/devops-sre/dora-2022-accelerate-state-of-devops-report-now-out)

> [!info] Use Four Keys metrics like change failure rate to measure your DevOps performance | Google Cloud Blog
> Learn how the Four Keys open source project lets you gauge your DevOps performance according to DORA metrics.
> [https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance](https://cloud.google.com/blog/products/devops-sre/using-the-four-keys-to-measure-your-devops-performance)

> [!info] Medição do DevOps: recursos de gerenciamento visual  |  Recursos de DevOps  |  Google Cloud
> [https://cloud.google.com/architecture/devops/devops-measurement-visual-management?hl=pt-br](https://cloud.google.com/architecture/devops/devops-measurement-visual-management?hl=pt-br)

# livros

- **O Projeto Fênix**
	- **POR KEVIN BEHR, GENE KIM E GEORGE SPAFFORD**

Criado por alguns dos nomes mais influentes do DevOps, este livro conta a história familiar de uma equipe de TI trabalhando com outras equipes da empresa para agregar valor.

[Saiba mais](https://www.amazon.com/dp/B00VATFAMI?plink=X05H5K1dDS3nLFtY&ref=adblp13nvvxx_0_1_ti)

- **O Projeto Unicórnio**
	- *POR GENE KIM**

A sequência de “The Phoenix Project”, este livro conta a história de fazer o trabalho da perspectiva de um desenvolvedor de software.

[Saiba mais](https://www.amazon.com/dp/B0812C82T9?plink=1fMbUAVvPrxjdCaS&ref=adblp13nvvxx_0_0_ti)

- **O Manual do DevOps**
	- **POR GENE KIM, PATRICK DEBOIS, JOHN WILLIS E JEZ HUMBLE**

Como uma continuação dos livros orientados por narrativas, “O Projeto Fênix” e “O Projeto Unicórnio”, este livro oferece conselhos mais práticos para alcançar agilidade, confiabilidade e segurança de qualidade internacional em empresas de tecnologia.

[Saiba mais](https://www.amazon.com/dp/B0767N1MM2?plink=3bLPfZu84xYkLKdf&ref=adblp13nvvxx_0_2_ti)

- **Accelerate**
	- **POR NICOLE FORSGREN, JEZ HUMBLE E GENE KIM**

Os autores apresentam os resultados da pesquisa com análises rigorosas sobre a construção e o dimensionamento de empresas de tecnologia de alto desempenho.

[Saiba mais](https://www.amazon.com/dp/B07BMBYHXL?plink=LG7z8TSNaJPkXuoS&ref=adblp13nvvxx_0_0_ti)

- **Effective DevOps**
	- **POR JENNIFER DAVIS E RYN DANIELS**

Muitas empresas se concentram em trazer mais ferramentas para impulsionar a transformação de DevOps, mas o “DevOps eficaz” vai além das ferramentas para lidar com a transformação cultural necessária para impulsionar a adoção.

[Saiba mais](https://www.amazon.com/Effective-DevOps-Building-Collaboration-Affinity/dp/1491926309/#ace-7448806443)

- **Project to Product**
	- **POR MIK KERSTEN**

Para sobreviver e prosperar no clima atual, as empresas devem se destacar na entrega de software em larga escala. O Dr. Mik Kersten apresenta o Flow Framework para ajudar as empresas a se tornarem inovadoras centradas em produtos.

[Saiba mais](https://www.amazon.com/Project-Product-Survive-Disruption-Framework-ebook/dp/B07F3DJMZ1/)

