# Migrando para um serviço totalmente gerenciado: Amazon Elasticsearch Service

by [AWS Editorial Team](https://aws.amazon.com/pt/blogs/aws-brasil/author/admin/) | on 17 AUG 2020 | in [Amazon OpenSearch Service (Successor To Amazon Elasticsearch Service)](https://aws.amazon.com/pt/blogs/aws-brasil/category/analytics/amazon-elasticsearch-service/), [Analytics](https://aws.amazon.com/pt/blogs/aws-brasil/category/analytics/), [AWS Big Data](https://aws.amazon.com/pt/blogs/aws-brasil/category/big-data/) | [Permalink](https://aws.amazon.com/pt/blogs/aws-brasil/migrando-para-um-servico-totalmente-gerenciado-amazon-elasticsearch-service/) | [ Share](https://aws.amazon.com/pt/blogs/aws-brasil/migrando-para-um-servico-totalmente-gerenciado-amazon-elasticsearch-service/#)

*Por Kevin Fallis, Arquiteto Especialista em Pesquisa de Soluções na AWS* 

Antes de ingressar na AWS, liderei uma equipe de desenvolvimento que criou soluções de Mobile Advertising com o Elasticsearch. O Elasticsearch é um mecanismo popular de pesquisa e análise de código aberto para análises de log, monitoramento de aplicativos em tempo real, análise de fluxo de cliques e, é claro, pesquisa. A plataforma pela qual eu era responsável era essencial para impulsionar os negócios da minha empresa.

Minha equipe executou uma implantação autogerenciada com o Elasticsearch da AWS. Na época, não havia uma oferta gerenciada do Elasticsearch. Tivemos que criar scripts e ferramentas para implantar nossos clusters do Elasticsearch em três regiões geográficas, o que incluiu as seguintes tarefas, entre outras:

- Configuração de rede, roteamento e firewall para permitir que o cluster se comunique.
- Proteção de APIs de gerenciamento do Elasticsearch contra acesso não autorizado.
- Criação de load balancers para distribuição de solicitação entre nós de dados.
- Criação de grupos de Auto Scaling para substituir as instâncias, se houvesse problemas.
- Automatização da configuração.
- Gerenciamento de upgrades por questões de segurança.

Se eu tivesse que descrever a experiência em uma palavra, seria “dolorosa”. Para implantar e gerenciar corretamente seus próprios clusters do Elasticsearch em escala, é preciso muito tempo e conhecimento. Talvez o ponto principal, foi que isso os distanciou de tarefas como inovar para os clientes, pois precisavam gerenciar o cluster.

[O Amazon Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/) (Amazon ES) foi lançado em 1 de outubro de 2015, quase 2 anos depois que eu ingressei na AWS. Quase 5 anos depois, o Amazon ES está na melhor posição de todos os tempos para fornecer a você um conjunto atraente de recursos que atenda às suas necessidades relacionadas à pesquisa. Com o Amazon ES, você obtém um serviço totalmente gerenciado que facilita a implantação, a operação e o dimensionamento de clusters do Elasticsearch de maneira segura e econômica na Nuvem AWS. Ele oferece acesso direto às APIs do Elasticsearch, que fazem com que o código e os aplicativos existentes usando o Elasticsearch funcionem perfeitamente com o serviço.

O Amazon ES fornece todos os recursos para o cluster do Elasticsearch e o inicia em qualquer região de sua escolha em questão de minutos. Ele detecta e substitui automaticamente os nós do Elasticsearch com falha, o que reduz a sobrecarga associada às infraestruturas autogerenciadas. Você pode dimensionar seu cluster horizontal ou verticalmente, até 3 PB de dados, com tempo de inatividade zero por meio de uma única chamada de API ou alguns cliques no [Console de Gerenciamento da AWS](https://aws.amazon.com/pt/console/). Com essa flexibilidade, o Amazon ES pode suportar qualquer carga de trabalho, de clusters de desenvolvimento de nó único a clusters de vários nós em escala de produção.

O Amazon ES também oferece um conjunto robusto de plug-ins Kibana, livres de quaisquer taxas de licenciamento. Recursos como como [controle de acesso refinado](https://docs.aws.amazon.com/pt_br/elasticsearch-service/latest/developerguide/fgac.html), [alerta](https://docs.aws.amazon.com/pt_br/elasticsearch-service/latest/developerguide/alerting.html), [gerenciamento de estado do índice](https://docs.aws.amazon.com/pt_br/elasticsearch-service/latest/developerguide/ism.html) e [suporte ao SQL](https://docs.aws.amazon.com/pt_br/elasticsearch-service/latest/developerguide/sql-support.html) são apenas alguns dos muitos exemplos. O conjunto de recursos do Amazon ES se origina das necessidades de clientes como você e de iniciativas comunitárias de código aberto, como o [Open Distro for Elasticsearch](https://opendistro.github.io/for-elasticsearch-docs/) (ODFE).

Você precisa levar diversos fatores em consideração em sua decisão de migrar para um serviço gerenciado. Você claramente quer que suas equipes se concentrem em realizar um trabalho significativo que impulsione o crescimento da sua empresa. Decidir quais processos você transfere para um serviço gerenciado em comparação ao que é melhor ser autogerenciado pode ser um desafio. Com base na minha experiência no gerenciamento do Elasticsearch na empresa anterior e tendo trabalhado com milhares de clientes que migraram para a AWS, considero as seções a seguir tópicos importantes a serem analisados.

 

## **Cargas de trabalhos**

Antes de migrar para um serviço gerenciado, você pode ver o que os outros estão fazendo em sua “vertical”, seja em finanças, telecomunicações, jurídico, comércio eletrônico, manufatura ou em vários outros mercados. Você pode ficar tranquilo em saber que milhares de clientes nessas verticais implantam com êxito suas pesquisas, análises de log, SIEM e outras cargas de trabalho no Amazon ES.

Por padrão, o Elasticsearch é um mecanismo de pesquisa. [A Compass](https://www.youtube.com/watch?v=V8yaocQZpr0) usa o Amazon ES para dimensionar sua infraestrutura de pesquisa e criar uma solução completa, escalável e de pesquisa sobre imóveis. Ao usar ferramentas de análise e de pesquisa líderes do setor, ela permite que cada anúncio do catálogo da empresa possa ser descoberto pelos consumidores e ajuda os profissionais do setor imobiliário a encontrar, comercializar e vender casas mais rapidamente.

Com ferramentas poderosas, como agregações e alertas, o Elasticsearch é amplamente usado para cargas de trabalho de análise de log para obter insights sobre atividades operacionais. À medida que a [Intuit](https://www.youtube.com/watch?v=7FSQVQEfJ9k) migra para uma arquitetura de hospedagem em nuvem, a empresa está em uma jornada de “observabilidade” para transformar a maneira como monitora a saúde de seus aplicativos. A Intuit usou o Amazon ES para criar uma solução de observabilidade, que fornece visibilidade à sua integridade operacional em toda a plataforma, de contêineres a aplicativos sem servidor.

Quando se trata de segurança, a [Sophos](https://www.sophos.com/) é líder mundial em segurança cibernética de próxima geração e protege seus clientes das ameaças cibernéticas mais avançadas da atualidade. A Sophos desenvolveu um sistema de alerta e monitoramento de segurança em larga escala usando o Amazon ES e outros componentes da AWS, porque ela sabe que o Amazon ES é adequado para casos de uso de segurança em escala.

Seja para encontrar uma casa, detectar eventos de segurança ou ajudar os desenvolvedores a encontrar problemas nos aplicativos, o Amazon ES oferece suporte a uma ampla variedade de casos de uso e cargas de trabalho.

 

## **Custo**

Qualquer discussão sobre as melhores práticas operacionais deve levar em consideração o custo. Com o Amazon ES, você pode selecionar o tipo de instância ideal e a opção de armazenamento para sua carga de trabalho com apenas alguns cliques no console. Se você não tiver certeza de quais são seus requisitos de computação e armazenamento, o Amazon ES oferece preços sob demanda, sem custos iniciais ou compromissos de longo prazo. Ao conhecer seus requisitos de carga de trabalho, você pode obter uma economia significativa de custos com a definição de preço da [Instância reservada](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/aes-ri.html) do Amazon ES.

Os custos de computação e infraestrutura são apenas uma parte da equação. Na AWS, incentivamos os clientes a avaliar seu [Custo Total de Propriedade](https://aws.amazon.com/tco-calculator/) (TCO) ao comparar soluções. Como tomador de decisão organizacional, você deve considerar todos os benefícios de custo relacionados ao optar por substituir seu ambiente autogerenciado. Alguns dos fatores que incentivo meus clientes a considerar são:

- Quanto você está pagando para gerenciar a operação do seu cluster ininterruptamente?
- Quanto você gasta na criação de componentes operacionais, como processos de suporte, procedimentos de correção automatizados e manuais para clusters em seu ambiente?
- Quais são os custos de licença para recursos avançados?
- Quais custos você paga pela rede entre os clusters ou os serviços DNS para expor suas ofertas?
- Quanto você gasta em processos de backup e com que rapidez você pode se recuperar de alguma falha?

A beleza do Amazon ES é que você não precisa se concentrar nesses problemas. O Amazon ES fornece equipes operacionais para gerenciar seus clusters, backups automáticos de dados por hora durante 14 dias para seu cluster, correção automatizada de eventos com seu cluster e recursos incrementais sem licença como um dos inquilinos básicos do serviço.

Você também precisa prestar muita atenção ao gerenciamento do custo de armazenamento de dados no Elasticsearch. No passado, para evitar que os custos de armazenamento ficassem fora de controle, os usuários autogerenciados do Elasticsearch tinham que confiar em soluções que eram complicadas de gerenciar em camadas de armazenamento e, em alguns casos, não forneciam acesso rápido a esses dados. A AWS resolveu esse problema com o [UltraWarm](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/ultrawarm.html), um novo nível de armazenamento de baixo custo. O UltraWarm permite armazenar e analisar interativamente seus dados, com o suporte do [Amazon Simple Storage Service](https://aws.amazon.com/s3/) (Amazon S3) usando o Elasticsearch e o Kibana, enquanto reduz o custo por GB em quase 90% em relação às opções de armazenamento quente existentes.

**Segurança**

Nas minhas conversas com os clientes, a principal preocupação deles é a segurança. Uma violação de dados pode custar milhões e prejudicar para sempre a reputação de uma empresa. O fornecimento das ferramentas para proteger seus dados é um componente essencial de nosso serviço. Para seus dados no Amazon ES, você pode fazer o seguinte:

• Obtenha isolamento de rede com o [Amazon Virtual Private Cloud](https://aws.amazon.com/pt/vpc/), (Amazon VPC);
• Criptografe dados em repouso usando as chaves criadas e controladas pelo [AWS Key Management Service](https://aws.amazon.com/pt/kms/) (AWS KMS);
• Use TLS 1.2 para criptografia em trânsito;
• Gerencie autenticações e controle de acesso com as políticas do [Amazon Cognito](https://aws.amazon.com/pt/cognito/) e do [AWS Identity and Access Management](https://aws.amazon.com/pt/iam/) (IAM).

Muitos clientes desejam ter um ambiente de logon único ao se integrarem ao Kibana. O Amazon ES oferece [autenticação do Amazon Cognito para Kibana](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-cognito-auth.html). Você pode optar por integrar provedores de identidade, como [AWS Single Sign-On](https://aws.amazon.com/single-sign-on/), [PingFederate](https://www.pingidentity.com/en/software/pingfederate.html), [Okta](https://www.okta.com/) e outros. Para obter mais informações, consulte [Integrando provedores de identidade SAML de terceiros com grupos de usuários do Amazon Cognito](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-integrating-3rd-party-saml-providers.html).

Recentemente, o Amazon ES introduziu o [controle de acesso minucioso](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/fgac.html) (FGAC). O FGAC fornece controle granular de seus dados no Amazon ES. Por exemplo, dependendo de quem faz a solicitação, convém que uma pesquisa retorne resultados de apenas um índice. Você pode ocultar certos campos em seus documentos ou excluir completamente certos documentos. O FGAC permite controlar quem vê quais dados existem no seu domínio Amazon ES.

 

## **Conformidade**

Muitas organizações precisam aderir a vários padrões de conformidade. Aqueles que passaram por atividades de auditoria e certificação sabem que garantir a conformidade é um processo caro, complexo e longo. No entanto, ao usar o Amazon ES, você se beneficia do trabalho que a AWS fez para garantir a conformidade com vários padrões importantes. O Amazon ES é compatível com PCI DSS, SOC, ISO e FedRamp para ajudá-lo a atender aos requisitos regulamentares ou específicos do setor. Como o Amazon ES é um serviço qualificado para HIPAA, o processamento, o armazenamento e a transmissão de PHI podem ajudá-lo a acelerar o desenvolvimento dessas cargas de trabalho sensíveis.

O Amazon ES faz parte dos[ serviços no escopo](https://aws.amazon.com/pt/compliance/services-in-scope/) da avaliação mais recente. Você pode criar soluções com o Amazon ES tendo a certeza de que auditores independentes reconhecem que o serviço atende aos padrões desses importantes padrões do setor.

 

## **Disponibilidade e resiliência**

Ao criar uma implantação do Elasticsearch no local ou em ambientes em nuvem, você precisa pensar em como sua implementação pode sobreviver a falhas. Você também precisa descobrir como se recuperar de falhas quando elas ocorrerem. Na AWS, gostamos de planejar o fato de que as coisas quebram, como falhas de hardware e disco, para citar alguns.

Diferentemente de quase qualquer outro provedor de infraestrutura de tecnologia, cada região da AWS tem várias zonas de disponibilidade. Cada zona de disponibilidade consiste em um ou mais data centers, fisicamente separados um do outro, com energia e rede redundantes Para alta disponibilidade e desempenho de seus aplicativos, você pode implantar aplicativos em várias zonas de disponibilidade na mesma região para tolerância a falhas e baixa latência. As Zonas de disponibilidade interconectam-se com redes de fibra ótica privadas e rápidas, que permitem projetar aplicativos que executam failover automaticamente entre as zonas de disponibilidade sem interrupção. As zonas de disponibilidade são mais altamente disponíveis, tolerantes a falhas e escalonáveis do que as infraestruturas tradicionais de um ou vários data centers.

O Amazon ES oferece a [opção de implantar suas instâncias em uma, duas ou três AZs](https://docs.aws.amazon.com/pt_br/elasticsearch-service/latest/developerguide/es-createupdatedomains.html#es-managedomains-multiaz) . Se você estiver executando cargas de trabalho de desenvolvimento ou teste, escolha a opção AZ único. Quem executa cargas de trabalho de nível de produção deve usar duas ou três zonas de disponibilidade.

Para obter mais informações, consulte [Increase availability for Amazon Elasticsearch Service by deploying in three Availability Zones](https://aws.amazon.com/es/blogs/database/increase-availability-for-amazon-elasticsearch-service-by-deploying-in-three-availability-zones-2/). Além disso, a implantação em várias zonas de disponibilidade com nós principais dedicados significa que você obtém os benefícios do [Amazon ES SLA](https://aws.amazon.com/elasticsearch-service/sla/).

 

## **Operações**

Uma equipe operacional 24 horas com experiência no gerenciamento de milhares de clusters do Elasticsearch em todo o mundo monitora o Amazon ES. Se você precisar de suporte, poderá obter orientação e assistência especializada em tecnologias do AWS Support para alcançar seus objetivos mais rapidamente e com custos mais baixos. Quero enfatizar a importância de ter uma única fonte de suporte para sua infraestrutura de nuvem. O Amazon ES não é executado isoladamente, e o suporte a toda a infraestrutura de nuvem de uma única fonte simplifica bastante o processo de suporte. A AWS também oferece a opção de usar [planos de suporte de nível empresarial](https://aws.amazon.com/premiumsupport/plans/), onde você pode ter um gerente de contas técnico dedicado que se torna essencialmente um membro de sua equipe e está comprometido com seu sucesso com a AWS.

O uso de ferramentas no Amazon ES, como [alertas](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/alerting.html), que fornece um meio de ação sobre eventos em seus dados e o [gerenciamento de estados de índice](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/ism.html), que permite automatizar atividades como o expurgo de logs, fornece recursos operacionais adicionais que você não precisará construir.

Quando se trata de monitorar suas implantações, o Amazon ES fornece diversas métricas do [Amazon CloudWatch](http://aws.amazon.com/cloudwatch) com as quais você pode monitorar todas as implantações do Amazon ES dentro de um “painel de vidro único”. Para obter mais informações, consulte [Monitorando métricas de cluster com o Amazon CloudWatch](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-managedomains-cloudwatchmetrics.html).

Manter-se atualizado é outro tópico importante. Para habilitar o acesso a versões mais recentes do Elasticsearch e Kibana, o Amazon ES oferece [atualizações do Elasticsearch no local](https://docs.aws.amazon.com/elasticsearch-service/latest/developerguide/es-version-migration.html) para domínios que executam as versões 5.1 e posteriores. O Amazon ES fornece acesso às versões mais estáveis e atuais da comunidade de código aberto, desde que a distribuição seja aprovada em nossas rigorosas avaliações de segurança. Nosso serviço se orgulha do fato de oferecermos uma versão que passou em nossas próprias análises de segurança internas da AWS.

 

## **Integrações da AWS e outros benefícios**

A AWS oferece uma ampla gama de serviços que se integram perfeitamente ao Amazon ES. Como muitos clientes, você deve monitorar a saúde e o desempenho de seus serviços de nuvem nativos na AWS. A maioria dos eventos de log dos serviços da AWS está no [Amazon CloudWatch Logs](http://aws.amazon.com/cloudwatch). Você pode configurar um grupo de logs para transmitir dados que recebe para o seu domínio Amazon ES através de uma assinatura do CloudWatch Logs.

O volume de dados do log pode ser altamente variável, e você deve considerar as camadas de buffer ao operar em larga escala. O buffer permite projetar estabilidade em seus processos. Ao projetar para escala, essa é uma das maneiras mais fáceis que conheço para evitar sobrecarregar seu cluster com eventos pontuais de ingestão. [O Amazon Kinesis Data Firehose](https://aws.amazon.com/kinesis/data-firehose/) tem uma [integração direta com o Amazon ES](https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html) e oferece armazenamento em buffer e novas tentativas como parte do serviço. Você configura o Amazon ES como um destino por meio de algumas configurações simples, e os dados podem começar a ser transmitidos para o domínio do Amazon ES.

 

## **Maior velocidade e agilidade**

Ao criar novos produtos e ajustar as soluções existentes, você deve poder experimentar. Como parte deste experimento, não conseguir resultados positivos imediatos é um processo natural, que oferece à sua equipe a capacidade de experimentar novas abordagens para acelerar o ritmo da inovação. Parte desse processo envolve o uso de serviços que permitem criar ambientes rapidamente e, se o experimento falhar, recomeçar com uma nova abordagem ou usar recursos diferentes que, em última análise, permitem alcançar os resultados desejados.

Com o Amazon ES, você tem o benefício de poder provisionar um cluster inteiro do Elasticsearch, completo com o Kibana, em “questão de minutos” em um ambiente gerenciado e seguro. Se o seu teste não produzir os resultados desejados, você poderá alterar as dimensões do cluster horizontal ou verticalmente usando diferentes ofertas de instância no serviço por meio de uma única chamada de API ou de alguns cliques no console.

Quando se trata de implantar seu ambiente, ferramentas nativas, como o [Amazon CloudFormation](https://aws.amazon.com/cloudformation/) fornecem ferramentas de implantação que permitem criar ambientes inteiros por meio de [scripts de configuração via JSON ou YAML](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-elasticsearch-domain.html). A [Interface da Linha de Comando da AWS](http://aws.amazon.com/cli) (CLI da AWS) fornece ferramentas de linha de comando que também podem ativar domínios com um pequeno conjunto de comandos. Para quem deseja aproveitar a última onda de criação de scripts em seus ambientes, o [AWS CDK](https://aws.amazon.com/cdk/) tem um [módulo para o Amazon ES](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-elasticsearch-readme.html).

 

## **Conclusão**

É essencial direcionar o foco de suas equipes no trabalho importante e inovador de criação de produtos e serviços que diferenciam sua empresa. O Amazon ES é uma ferramenta fundamental para fornecer estabilidade operacional, segurança e desempenho de sua infraestrutura de pesquisa e análise. Quando você considera os seguintes benefícios que o Amazon ES oferece, a decisão de migrar é simples:

- Suporte para pesquisa, análise de log, SIEM e outras cargas de trabalho;
- Funcionalidade inovadora usando o UltraWarm para ajudá-lo a gerenciar seus custos;
- Ambientes altamente seguros que abordam cargas de trabalho PCI e HIPAA;
- Capacidade de descarregar processos operacionais para um provedor experiente que sabe operar o Elasticsearch em escala;
- Plug-ins sem custo adicional que fornecem acesso refinado, algoritmos de similaridade baseados em vetores ou alertas e monitoramento com a capacidade de automatizar a resposta a incidentes.

Você pode começar a usar o Amazon ES com o [Nível gratuito da AWS](https://aws.amazon.com/free). Esse nível oferece uso gratuito de até 750 horas por mês de uma instância t2.small.elasticsearch e 10 GB por mês de armazenamento opcional do EBS (Magnético ou Propósito geral).

Ao longo dos próximos meses, vou ajudar a escrever uma série de postagens que apresentam padrões de migração para ajudá-lo a migrar para o Amazon ES. Além disso, a AWS possui um ecossistema robusto de parceiros e uma equipe de serviços profissionais que fornece a você pessoal qualificado para ajudá-lo com sua migração.

 

*Este artigo foi traduzido do [Blog da AWS em Inglês](https://aws.amazon.com/blogs/big-data/moving-to-managed-the-case-for-amazon-elasticsearch-service/).*

 

------

## Sobre o autor

![img](https://d2908q01vomqb2.cloudfront.net/d435a6cdd786300dff204ee7c2ef942d3e9034e2/2020/08/17/kffallis-1.png)**Kevin Fallis** é um arquiteto de soluções de pesquisa especializado da AWS. Sua paixão na AWS é ajudar os clientes a aproveitar a combinação correta de serviços da AWS para alcançar o sucesso de seus objetivos de negócios. Suas atividades fora do trabalho incluem família, projetos de faça você mesmo, carpintaria, tocar bateria e tudo relacionado à música.

 