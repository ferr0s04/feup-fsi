# 2 - Segurança de Software

## Índice
[1) Descoberta de Vulnerabilidades](#1-descoberta-de-vulnerabilidades)  
[2) Buffer Overflow](#2-buffer-overflow)  
[3) Buffer Overflow na Stack (Stack Smashing)](#3-buffer-overflow-na-stack-stack-smashing)  
[4) Buffer Overflow na Heap](#4-buffer-overflow-na-heap)  
[5) Outros tipos de Overflow](#5-outros-tipos-de-overflow)  
[6) Bibliotecas](#6-bibliotecas)  
[7) Medidas de Proteção](#7-medidas-de-proteção)  

## 1) Descoberta de Vulnerabilidades
- São geralmente erros de gestão de memória por parte dos programadores

- O primeiro sinal é um crash do programa
    - **Fuzzing/fuzz:** técnica que fornece inputs aleatórios e inesperados a um programa. Se o programa crashar em algum momento, investiga-se porquê

- **Ethical hacking:** são necessários muitos cuidados ao reportar e divulgar uma vulnerabilidade. Os sistemas de *bug bounty* e *vulnerability reporting* são desenhados para evitar problemas tanto para os hackers como para as empresas

## 2) Buffer Overflow
- Vulnerabilidade mais comum em software

- Consiste em injetar código malicioso na memória (shellcode) e alterar o controlo da execução para saltar para essa zona

## 3) Buffer Overflow na Stack (Stack Smashing)
- Preencher o buffer com código, substituir o endereço de retorno e executar código arbitrário, geralmente um shellcode

- **Shellcode:** sequência de instruções em binário que executa comandos shell

*Nota: shellcode não pode conter "0x00" uma vez que este caractere denota o fim de uma string e impede o atacante de copiar o shellcode para o buffer*

- Evitar adivinhar o endereço específico onde reside shellcode:
    - Disposição na stack: após a execução do *return*, o *sp* aponta para o endereço mais acima
    - Buffer overflow normal: é preciso colocar no sítio do *return address* um endereço que aponta para o shellcode (difícil de adivinhar)
    - Truque que usa *jmp sp*: Evita ter de adivinhar o endereço. Se o opcode estiver num endereço conhecido, reescrevemos o *return address* com esse endereço. Quando a função retorna, executa o shellcode

- Outros exemplos:
    - fingerid
    - libpng
    - realpath

- Porque é que stack smashing acontece:
    - Manipulação de strings/buffers usando bibliotecas tipo libc
    - Funções unsafe que não garantem escrita limitada à região de memória (strcpy, strcat, scanf...)
    - Código mal escrito


## 4) Buffer Overflow na Heap
- Nas linguagens de tratamento de exceções, caso haja um buffer overflow na heap, escreve-se por cima de um apontador da tabela de apontadores para as rotinas de tratamento de exceções

- Se a exceção ocorrer, a execução prosseguirá para o endereço da nossa escolha

- **Heap spraying:** inundar a memória com cópias do shellcode. O buffer overflow coloca o apontador algures na zona alvo, aumentando a probabilidade de o atacante obter controlo. Evita ter de adivinhar o endereço exato do código malicioso

- **Use after free:** preencher uma zona de memória recentemente libertada (com *free*) com endereços, normalmente para execução de shellcode
    - ex: um programa onde ocorre *free(ptr)* e depois há um acesso a ptr. O acesso errado permite a tomada de controlo pelo atacante

## 5) Outros tipos de Overflow
### Overflow de inteiros
- Quando há perda de informação devido à representação de inteiros do processador
    - Comparação que esquece a representação com sinal
    - Truncatura por passagem para tipo mais pequeno

### Format strings
- Quando o programador não usa format strings (%s), é o input que controla o formato do output. Assim, o atacante consegue escrever em localizações arbitrárias da memória

## 6) Bibliotecas
- A biblioteca libc é usada para quase todos os programas
- Funções úteis: system (correr comando shell) e mprotect (alterar permissões de memória)

## 7) Medidas de Proteção
- Geralmente, implicam um overhead significativo, afetando por vezes a performance de um programa

- Proteção na própria plataforma: impedir a execução de código malicioso (DEP, ASLR, KASLR, KARL, PIO)

- Proteção no executável: Stack Canaries, Memory tagging, CFI

- **Data Execution Prevention (DEP)** ou **Executable Space Protection (W^X)**
    - Incluído em sistemas modernos, como o Windows
    - Uma página de memória executável não pode ser escrita
    - Uma página de memória que pode ser escrita não pode ser executável
    - Implementado em hardware ou emulado em software

- **Address Space Layout Randomization (ASLR):** a localização em memória de partes críticas de um programa é baralhada aleatoriamente em cada execução

- **Kernel Address Space Layout Randomization (KASLR):** a localização do código do kernel é randomizada no boot

- **Kernel Access Randomized Link (KARL):** o próprio código do kernel é baralhado

- **Position-Independent Executables (PIO):** código que executa bem independentemente de onde se encontre na memória. Torna *return oriented programming* muito mais difícil

- **Stack Canaries:** deteta modificações à stack e previne a injeção de código
    - Canário aleatório: é gerado um array de bytes aleatório a cada execução. Se houver alguma alteração a esse array, o programa termina
    - Canário de terminação: usam '\0', '\n', EOF em vez de bytes aleatórios. As funções que manipulam strings (como strcpy) irão terminar sempre nestes valores

- Mitigações adicionais:
    - Buffers sempre juntos ao canário, para que qualquer tentativa de overflow seja detetada
    - Copiar os argumentos no topo da stack para abaixo dos buffers locais, não sendo possível alterar estes valores
    - Shadow control stack: ter uma stack redundante e comparar à stack original antes de retornar

- Memory tagging
    - Um apontador tem a mesma tag que uma zona de memória. O programa verifica a tag antes de aceder à zona de memória e lança uma exceção se detetar que a tag foi alterada por overflow (uso de *free*)

- **Control flow integrity (CFI):** confirmar que a chegada a qualquer função é resultado de uma *call* e não de um endereço criado dinamicamente

### Program safety
- Um programa é considerado seguro se todas as suas execuções têm significado, ou seja, que o programa se comporta exatamente como esperado. Quando ocorre um erro, essa execução não tem significado, pelo que o programa deixa de ser seguro

- Funções unsafe têm resultados "undefined". No caso do C, "tudo pode acontecer" se usarmos uma variável que não foi inicializada, pelo que o C não é considerada uma linguagem muito segura

- Apesar de as linguagens não serem desenhadas para garantir segurança, algumas linguagens, como o Java ou o Haskell, verificam um conjunto alargado de condições em tempo de compilação, não compilando se detetarem operações unsafe