# Trabalho realizado nas Semanas #2 e #3

## Identificação

- CVE-id : CVE-2022-22947
- Esta é uma vulnerabilidade de injeção de código do Spring Cloud Gateway entre as versões 3.0.0 e 3.0.6, e 3.1.0 e 3.1.1.
- Surge devido a uma falha na validação das expressões SPEL (linguagem que faz parte do spring framework) em cabeçalhos HTTP. Desta forma, poderiam ser enviados cabeçalhos HTTP com expressões SPEL maliciosas, e estas eram executadas diretamente no servidor, sem o devido controlo.
- Como consequência, atacantes que soubessem estruturar estas expressões, podiam injetar comandos que eram diretamente interpretados e executados no servidor, levando a uma execução remota de código.
- Esta falha dá aos invasores controlo sobre o servidor, permitindo práticas como roubo de dados, alteração de configurações ou instalação de malware.

## Catalogação

- A vulnerabilidade foi descoberta de forma independente por Wyatt Dahlenburg, tendo sido posteriormente relatada em março de 2022 pela equipa da VMware.
- Está catalogada como uma vulnerabilidade de code injection e de expression language injection, tendo uma pontuação base CVSS de 10.0 (crítico).
- Destaca-se:
  - Impacto de confidencialidade: Alto - o atacante pode aceder ou modificar dados sensíveis.
  - Impacto na integridade e disponibilidade: Alto - a vulnerabilidade permite alterações não autorizadas no sistema.
  - Complexidade de acesso: Baixa - requer apenas que o endpoint Actuator esteja ativo, não sendo necessária autenticação nem conhecimentos avançados.
  - Vetor de ataque: Rede

## Exploit

- Foram encontrados dois exploits que usão **CVE-2022-22947**, estes afetam o **Spring Cloud Gateway 3.0.7 a 3.1.1**  ,  possibilitado a injeção de comandos no servidor, se os endpoints do **Actuator API** estiverem expostos e desprotegidos.
- O primeiro exploit encontrado foi criado por Carlos E. Vieira dia 03/03/2022 que afeta **Spring Cloud Gateway**.
- O segundo expoit afeta **Spring Expression Language (SpEL)** que e usado pelo **Spring Cloud Gateway**.

### O procedimento dos exploits e o seguinte:

- O exploit começa por desligar os avisos de verificação SSL, a inclusão das bibliotecas necessárias para efetuar o exploit como, por exemplo, para efetuar pedidos HTTP que simule uma request feita pelo navegador e por fim cria um id aleatório que será usado como um identificador para o URL onde o comando será posto.
- A seguir, e criado um payload com o comando que se deseja injetar no servidor, este payload e depois enviado para o alvo. O payload pede ou servidor para adicionar um novo header com o comando malicioso.
- Depois são efetuados três passos para executar o código que foi adicionado no server:
  1. E feito uma request para o URL `/actuator/gateway/routes/{id}` sendo o id o que foi criado anteriormente.
  2. Depois e pedida uma atualização do Gateway a partir do URL `/actuator/gateway/refresh` para que a adição da nova rota faça efeito.
  3. Por fim e feita uma request para o URL `/actuator/gateway/routes/{id}` para que o comando faça efeito.
- Para terminar depois do comando ter sido efetuado e o resultado adquirido e apagado a rota adicionada para não ser detetável.

### Como impedir os exploits

- Para mitigar esta vulnerabilidade, é crucial atualizar o **Spring Cloud Gateway** para uma versão corrigida que não seja afetada pelo **CVE-2022-22947**. Além disso, é altamente recomendável garantir que o acesso aos endpoints do actuator esteja devidamente seguro, limitando o acesso apenas a utilizadores autorizados e implementando autenticação robusta.

## Ataques

- A Oracle é a principal fundação que utiliza a Spring Cloud Gateway como tecnologia para facilitar a comunicação e a gerência de dados nas suas plataformas. Quando esta vulnerabilidade afetou a framework, os produtos desta empresa também foram expostos a ataques maliciosos.
- Os produtos da Oracle que usufruíam da framework desempenhavam sobretudo um papel central em comércio digital e infraestruturas de telecomunicações 5G em softwares populares utilizados diariamente. Desta forma, esta vulnerabilidade impactou diretamente:
  - Serviços de e-commerce: prejudicou a experiência de compras online, causando a interrupção de serviços de busca de produtos e, ainda mais preocupante, expondo dados pessoais ou de transações.
  - Redes 5G: afetou o desempenho de redes móveis, o que resultou em interrupções em chamadas, acesso à internet e conectividade de dispositivos conectados à IoT (Internet of Things), que são utilizados em setores como a saúde. 
- Qualquer falha ou interrupção nesses sistemas pode ter um impacto significativo, pelo que corrigir estas vulnerabilidades leva à mitigação dos riscos e garante a continuidade dos serviços e a segurança dos dados que dependem dessas soluções.
