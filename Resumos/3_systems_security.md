# 3 - Segurança de Sistemas

## Índice
[1) Princípios Fundamentais](#1-princípios-fundamentais)  
[2) Controlo de Acessos](#2-controlo-de-acessos)  
[3) Segurança de Sistemas Operativos](#3-segurança-de-sistemas-operativos)  
[4) Confinamento](#4-confinamento)  
[5) Malware](#5-malware)  

## 1) Princípios Fundamentais
- Economia nos mecanismos
    - Ter apenas as funcionalidades necessárias
    - Evitar funcionalidades "nice-to-have", pois acrescentam complexidade à segurança do sistema
    - Facilita a implementação, usabilidade, validação...

- Proteção por omissão
    - Nível de proteção conservador (ex: utilizadores novos têm o mínimo de permissões)
    - **Fail closed:** caso um sistema falhe, reduzir as permissões/privilégios.

- Desenho aberto
    - A arquitetura de segurança e os detalhes de funcionamento devem ser públicos
        - A descoberta de vulnerabilidades é mais rápida porque há uma comunidade mais alargada a procurá-las
    - Os segredos são parâmetros do sistema e podem ser alterados: tokens, chaves criptográficas...
        - Possibilidade de mudar os segredos sem resenhar todo o sistema

- Defesa em profundidade
    - Ter um vasto conjunto de medidas de segurança, pois todos os mecanismos podem falhar
    - Minimizar a probabilidade de erros na gestão de memória
    - Usar linguagens de programação que garantem *memory safety* (ex: Java, Go, Rust...)
    - Verificar que os programas estão corretos

- Privilégio mínimo
    - Cada utilizador/programa deve ter apenas os privilégios necessários para a sua função
        - Exemplo de violação: correr todos os serviços como root

- Separação de privilégios
    - As funcionalidades devem ser compartimentadas:
        - Compartimentos isolados uns dos outros
        - Encarar os outros compartimentos como estando noutro domínio de confiança

- Mediação completa
    - Um sistema gere, invariavelmente, recursos: ficheiros, dispositivos de hardware, memória...
    - Definir uma política de proteção para cada recurso
    - Todos os acessos a todos os recursos são validados relativamente a essa política

## 2) Controlo de Acessos
- Mecanismos que visam aplicar os princípios de Privilégio Mínimo e Mediação Total


## 3) Segurança de Sistemas Operativos
- **Sistema operativo:** interface entre os utilizadores de um computador e o hardware, com o intuito de gerir o acesso a recursos por parte de aplicações (armazenamento, processador, IO...)

- Aspetos de segurança a considerar:
    - Os sistemas operativos têm múltiplos utilizadorescom diferentes níveis de acesso e privilégios
    - Os utilizadores são sempre potenciais ameaças e os recursos são sempre ativos a proteger
    - As aplicações também são potenciais ameaças e devem estar protegidas da interferência de outras (separação de privilégios)

### Kernel
- Desempenha as funções mais críticas do sistema operativo

- Os dispositivos podem estar num nos seguintes modos:
    - **Kernel mode:** todas as operações são permitidas. Apenas código que corre neste modo pode executar instruções privilegiadas
    - **User mode:** não tem acesso direto aos recursos do sistema. Qualquer processo neste modo tem de aceder aos recursos do sistema usando *system calls*

- Superfície de ataque: define a fronteira entre contextos de confiança. Medida com a quantidade de system calls

### System Calls
- São os pontos críticos na segurança, pelo que existem em número limitado

- Implica implementar sistemas de monitorização de chamadas ao sistema - Reference monitor

- O que podem fazer:
    - Controlo de processos (fork, load, wait...)
    - Acesso a ficheiros (criar, ler, escrever...)
    - Gestão de dispositivos (obter acesso, escrever/ler...)
    - Configuração do sistema (hora, data, características...)
    - Comunicações (estabelecer ligação, mensagens...)
    - Proteção (alterar/obter permissões de acesso a recursos)

- O que pode fazer um atacante com system calls:
    - Identificar um ponto permitido para acesso ao kernel mode de modo a entrar num modo de funcionamento com mais privilégios

### Processos
- Modelo de confiança:
    - Código no computador (BIOS, kernel): confiável
    - Preservado por processos de hibernação
    - Alterações pelos administradores devem preservar o estado de confiança

- Modelo de ameaças:
    - A maioria dos problemas de segurança devem-se a erros de administração
    - Ataques em todos os níveis do boot: bypass completo ao modelo de confiança
        - BIOS corrompida
        - ficheiros de hibernação corrompidos/roubados
        - bootloader corrompido
        - cold boot attacks (ler a memória do computador antes de o arrancar)

- Medidas de mitigação:
    - Sistemas de monitorização (logs, ver processos em execução...)
    - Instalação de código com base em assinaturas digitais (entrave ao código malicioso)
    - Processos não podem aceder ao espaço de memória de outros processos
    - Impedir o próprio kernel de escrever em partes da memória do utilizador (previne fugas de informação em caso de kernel corrompido)
    - Virtualização da memória e comunicação usando system calls
        - TLB (Translation Lookaside Buffer)
        - Kernel mapping

### Sistemas de ficheiros
- Medidas de mitigação:
    - Não abusar do uso de root
    - Discretionary Access Control: apenas o dono de um recurso pode alterar as permissões sobre esse recurso
    - Mandatory Access Control: apenas o root pode alterar as permissões
    - Utilizar a mesma interface para outros recursos (minimiza o nº de system calls e a superfície de ataque)

## 4) Confinamento
- Confinar código não confiável para não afetar o resto do sistema
- Alguns códigos não confiáveis que pode ser necessário executar:
    - Código de fontes externas, como sites (JavaScript, extensões...)
    - Código legacy (original) mas que pode ter falhas
    - Honeypots: código vulnerável criado de propósito para atrair atacentes
    - Análise forense de malware

- Medidas que garantem confinamento:  
    - **Air gap:** isolamento físico do sistema de redes inseguras. Difícil de gerir

    - **Máquinas virtuais** ou **hypervisors**

    - **Software Fault Isolation (SFI):** isolamento de processos que partilham o mesmo espaço de endereçamento
        - Limitar a zona de memória acessível a uma aplicação
        - Operações bitwise para verificar o acesso na gama correta
        - Algumas operações perigosas que devem ser protegidas com guardas (load/store, saltos...)
    
    - **System Call Interposition (SCI):** mediação de todas as system calls de modo a monitorizá-las
        - ptrace, seccomp+bpf
    
    - **Sandboxing:** fornece virtualização (logo, proteção contra código externo) dentro de uma aplicação

    - **Reference monitor:** mediação de todos os pedidos de acesso a recursos
        - Presente em todos os sistemas anteriores (air gap, hypervisors, SFI, SCI, sandboxing)
        - Omnipresente: quando pára, todos os processos param
        - Suficientemente simples para ser analisado

## 5) Malware
- Software desenhado para causar danos através da exploração de uma vulnerabilidade ou execução de código malicioso, comprometendo o estado de confiabilidade de um sistema

- Tipos de malware:
    - **Vírus:** não se propaga sozinho. Fica latente até o utilizador fazer alguma ação específica que despolete esse malware
    - **Worm:** propaga-se sozinho e cria as condições para se executar e propagar
    - **Rootkit:** permitir a um atacante acesso privilegiado ao sistema, escondendo também a sua presença
    - **Trojan:** aparenta ser legítimo, mas tem como objetivo típico transmitir informação para um atacante

### Vírus
- Sequência:
    1. Utilizador executa um programa infetado
    2. O código do vírus fica armazenado no programa
    3. O vírus é executado quando o programa é executado
    4. O vírus localiza outro computador para infetar
    5. O vírus replica-se para o código desse outro programa
    6. Se determinada lógica for ativada, executa tarefas maliciosas
    7. No final pode desaparecer/apagar-se

- É executado com os privilégios do utilizador, mas pode explorar outras vulnerabilidades (ex: escalar privilégios, com setuid)

- Objetivos:
    - Brincadeira ou causar danos (pop-ups, apagar ficheiros, danificar hardware)
    - Vigilância/espionagem (key logging, captura de ecrã, áudio, câmara)

### Botnets
- Rede de computadores com um sistema de comando e controlo comum
    - Cada computador é um bot
    - Comandos enviados através da rede: spam, phishing, DoS...
    - Capacidade de se atualizar automaticamente

- Tipos de arquitetura:
    - Centralizada: múltiplos servidores centrais para robustez
    - Peer-to-peer: auto organizada e hierárquica

- Tipos de fluxo:
    - Push: servidor envia comandos
    - Pull: periodicamente, bot pergunta se há comandos

- Deteção:
    - Detetar o malware na máquina comprometida
    - Detetar tráfego na rede
    - Usar honeypots para expor máquinas

- Combate:
    - Limpar máquinas comprometidas ou isolá-las da rede
    - Isolar/desligar o centro de comando e controlo
    - Tomar o controlo do centro de comando e controlo e usá-lo para desativar a botnet

### Outros
- **Malvertising**
- **Engenharia social**
- Deturpar equipamento no fabrico
- Deturpar equipamento em trânsito (no caso de encomendas)
- Atacar fornecedor de software e injetar updates que contêm backdoors

### Combater malware
- **Intrusion Detection System (IDS):** deteção ocorre depois do ataque
- **Intrusion Prevention System (IPS):** intervenção rápida para evitar ataque
- **Blacklisting:** impedir que ataques conhecidos aconteçam. Todos os outros são bons
- **Whitelisting:** definir quem pode entrar e, por omissão, tudo o resto não pode entrar

### Erros de deteção
- **Falso positivo:** há alertas de deteção quando não houve intrusão
- **Falso negativo:** não há alertas de deteção e houve de facto uma intrusão
