# 4 - Segurança Web

## Índice
[1) Protocolo HTTP](#1-protocolo-http)  
[2) Cookies](#2-cookies)  
[3) Modelo de Execução](#3-modelo-de-execução)  
[4) Modelos de Ataque](#4-modelos-de-ataque)  
[5) Modelo de Segurança](#5-modelo-de-segurança)  
[6) Vulnerabilidades, Ataques e Mitigação](#6-vulnerabilidades-ataques-e-mitigação)  

## 1) Protocolo HTTP
- HTTP: Hyper Text Transfer Protocol

- Protocolo que usa hyperlinks (um documento pode referenciar outros), que o utilizador pode seguir, clicando nesse link

- Estrutura de um URL:
    - Esquema ou protocolo (http, https...)
    - Domínio (aaa.bbb.cc), potencialmante com uma porta específica
    - Caminho para o recurso
    - Informação extra

- Tipos de pedidos:
    - GET: obter recurso numa URL
    - POST: criar um novo recurso numa URL
    - PUT: substituir conteúdo existente com outro conteúdo
    - PATCH: alterar parte de um recurso
    - DELETE: apagar recurso numa URL

## 2) Cookies
- Pedaço de informação que uma aplicação web pede ao browser para armazenar

- Útil para gestão de sessões, personalização, rastreamento/profiling

- Passos:
    1. O servidor pede ao browser para armazenar informação
    2. Essa informação é guardada num ficheiro
    3. Sempre que o browser volta a pedir um recurso ao mesmo servidor, devolve esse ficheiro (cookie)

- Cookies definidos num domínio são acessíveis por todos os seus subdomínios

## 3) Modelo de Execução
- Modelo complexo devido às tarefas do browser de gerir contextos de confiança

- Origem: protocolo (ex: HTTP) e domínio

- JavaScript consegue aceder a todos os recursos que vêm da mesma origem, mas também consegue executar código de outra origem, se for executado em nome do utilizador.

- Servidor:
    - bases de dados, gestão de sessões, personalização...

- Cliente:
    - Browsers: processam HTML, executam JavaScript, pede recursos (CSS, imagens...), responde a eventos do utilizador ou definidos pelo site

- Cada tab é uma **frame** e possui várias sub-frames ou **iFrames**

- **iFrames:** elementos HTML que permitem embeber uma página web dentro de outra página web

## 4) Modelos de Ataque
- Atacante externo/rede: controla apenas o meio de comunicação

- Atacante interno/web: controla parte da aplicação web
    - Adversário que controla servidor
    - Adversário que controla cliente
    - Adversário que controla uma página no cliente
    - Adversário que controla um objeto embebido numa página
    - Adversário que controla uma página e que pode interferir com o conteúdo embebido nessa página

## 5) Modelo de Segurança
### Same Origin Policy (SOP)
- Política de isolamento imposta pelo browser
    - Confidencialidade: dados não podem ser acedidos por código de origem diferente
    - Integridade: dados não podem ser alterados por código de origem diferente

- SOP no DOM: cada frame tem uma origem (esquema, domínio, porta)

- SOP para mensagens: frames podem comunicar entre si

- SOP para cookies:
    - A origem é só o domínio e o path (esquema é opcional)
    - A cookie declara a origem que é associada pelo browser a essa cookie

- SOP para comunicações com o servidor:
    - Escrita geralmente permitida

### Cross-Origin Resource Sharing (CORS)
- Permite a uma página requisitar recursos de outra página com domínio diferente

- Versão simples:
    - Pedidos simples que não devem causar side-effects
    - O browser faz o pedido e verifica se o código pode aceder aos recursos do outro servidor
    - Permissão de uso de recursos através de "Access-Control-Allow-Origin"

- Versão mais elaborada:
    - Pedidos pre-flighted que não devem causar side-effects
    - O browser faz um pedido *dummy* e verifica-o antes de fazer o pedido verdadeiro

## 6) Vulnerabilidades, Ataques e Mitigação
### SQL Injection
- Input do utilizador que altera a semântica do comando SQL construído dinamicamente

- Mitigação:
    - Nunca criar comandos SQL dinamicamente como strings
    - Usar instrumentos como comandos parametrizados (estáticos) ou bibliotecas ORM (criação de classes/objetos e sanitização de inputs)

### Session Hijacking
- O atacante interceta comunicação entre o cliente e o servidor

- Basta provocar o envio de uma cookie e intercetar o seu envio

### Cross-Site Request Forgery (CSRF)
- Tentar fazer pedidos fora da sua origem e tirar partido dos efeitos desses pedidos (ex: envio de uma cookie)

- Mitigação:
    - Mecanismo que garanta ao servidor que o pedido vem de uma página confiável
    - Impedir login: uso de tokens dinâmicos
    - Impedir uso de cookies: usar "SameSite=Strict"
    - Validação explícita no servidor de atributos Referer e Origin

### Cross-Site Scripting (XSS)
- Injeção de código malicioso no site, que será executado mais tarde no browser de um visitante

- Diferença com CSRF: com XSS, o atacante tenta conseguir que um site legítimo (não de outra origem) provoque a execução de código malicioso no cliente. CSRF explora a confiança do utilizador no browser, enquanto que XSS explora a confiança do utilizador no website

- Tipos de XSS:
    - **Reflected XSS** (nonpersistent XSS): acontece instantaneamente
    - **Stored XSS** (persistent XSS): ex: código guardado numa entrada da base de dados, que será executado quando o utilizador pesquisa por essa entrada

- Mitigação:
    - Filtrar inputs/pedidos: validar headers/cookies/queries, codificar código malicioso...
    - **Content Security Policy (CSP):** Apenas scripts de origens de uma white-list são executados
    - **Subresource Integrity:** scripts alterados não são executados
