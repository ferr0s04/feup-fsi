# 1- Introdução

## Índice
[1) Comprometer a Máquina de um Utilizador](#1-comprometer-a-máquina-de-um-utilizador)  
[2) Comprometer um Servidor](#2-comprometer-um-servidor)  
[3) Segurança](#3-segurança)  
[4) Atores](#4-atores)  
[5) Atacante](#5-atacante)  
[6) Modelos de Segurança](#6-modelos-de-segurança)  
[7) Vulnerabilidades](#7-vulnerabilidades)  
[8) Ataques](#8-ataques)  
[9) Ameaças](#9-ameaças)  
[10) Mecanismos e Políticas de Segurança](#10-mecanismos-e-políticas-de-segurança)  
[11) Modelo de Confiança](#11-modelo-de-confiança)  
[12) Engenharia de Segurança](#12-engenharia-de-segurança)



## 1) Comprometer a Máquina de um Utilizador
Razões:
- Roubar credenciais/passwords
    - Engenharia social (ex: phishing)
    - Ataques man-in-the-middle
- Ransomware
    - Roubar/bloquear dados para exigir um pagamento
- Roubar poder de processamento
    - Ex: minerar bitcoins
- Usurpação do endereço de rede
    - Spam
    - Denial of Service
    - Cliques em anúncios no browser

## 2) Comprometer um Servidor
Razões:
- Data Breaches
    - Acesso a credenciais de utilizadores em massa
- Motivações políticas e geo-estratégicas
- Para infetar máquinas dos utilizadores
    - supply-chain attacks: infetar servidores que distribuem software e distribuir, por sua vez, malware
    - web-server attacks: infetar servidores web para comprometer browsers

## 3) Segurança
- É a propriedade de um sistema que se comporta como esperado.
- Traduz-se na proteção de sistemas de computadores/redes de fugas de informação, roubo ou danos ao hardware, software ou dados eletrónicos, assim como da disrupção ou alteração dos serviços que fornecem.
- É relativa - depende que quem a define
- A sua definição muda consoante o contexto
- É defensiva - deve-se assumir que tudo pode acontecer

## 4) Atores
- Entidades que intervêm no sistema
- Pessoas, organizações, empresas, máquinas...
- Muitas vezes deposita-se confiança em alguns atores ou componentes
    - ex: Trusted Third Party (TTP), Trusted Agent (TA)
    - Considera-se o sistema "seguro" se pressupostos de confiança se verificarem

## 5) Atacante
- Atores com intenção explícita de utilizar o sistema/recursos de forma maliciosa e/ou impossibilitar a sua utilização
- Nenhum sistema é seguro contra todos os atacantes
- Tipos de adversários:
    - "Script-kiddies"
    - Atacantes ocasionais que visam compreender o sistema
    - Atacantes com intenção de causar danos/destruição
    - Grupos organizados e tecnologicamente sofisticados (ex: cibercrime)
    - Competidores (ex: espionagem industrial)
    - Países/estados/governos

## 6) Modelos de Segurança
### Modelo Binário
- Típico em criptografia e sistemas confiáveis
- Define formalmente capacidades e objetivos
- Limitações:
    - Não escala para sistemas complexos
    - Os modelos formais podem estar errados (ex: side-channels)

### Modelo de Gestão de Risco
- Típico em engenharia de software e segurança no mundo real
- Minimiza o risco em função das ameaças mais prováveis
- Otimizar os custos das medidas de segurança vs potenciais perdas
- Análise de risco através de uma matriz de dois fatores
- Limitações:
    - A análise de risco pode estar errada
    - Uma ameaça mal classificada pode causar exposição severa
- **Ativos:**
    - Recursos valiosos para atores de um sistema (ex: informação, reputação, dinheiro...)
- **CIA: Confidencialidade, Integridade e Disponibilidade**

## 7) Vulnerabilidades
- Falhas que estão acessíveis a atacantes que poderão ter a capacidade de as explorar
- Têm geralmente origem em erros de concepção:
    - Software de má qualidade
    - Análise de requisitos inadequada
    - Configurações erradas
    - Utilização errada

## 8) Ataques
- Ocorrem quando um atacante tenta explorar uma vulnerabilidade
- Ataque bem sucedido -> Sistema **comprometido**
- **Exploit/Método:** forma de explorar a vulnerabilidade
- Tipos de ataques:
    - Passivos: monitorização de comunicações/sistemas/informações
    - Ativos: adivinhar passwords, Denial-of-Service

## 9) Ameaças
- Possíveis incidentes que podem trazer consequências negativas para um sistema, pessoa ou organização
- Definem-se pelo seu tipo e origem
    - Tipo: dano físico, perda de serviços, fuga de informação, falhas técnicas, abuso de funcionalidades
    - Origem: ação deliberada, ação negligente, acidente, evento ambiental, falha, ator externo
- Identificadas e classificadas quanto à relevância

## 10) Mecanismos e Políticas de Segurança
### Mecanismo de Segurança
- Método, ferramenta ou procedimento que permite implantar uma (parte de uma) política de segurança
- Exemplos:
    - Mecanismos de autenticação (ex: biometria, OTP)
    - Mecanismos de controlo de acessos (ex: RBAC)
    - Criptografia (ex: cifras, assinaturas, MACs)
    - Controlos físicos (ex: cofres, torniquetes)
    - Auditorias (ex: penetration testing)

### Política de Segurança
- Conjunto de processos que devem ser seguidos para garantir segurança num determinado modelo de ameaças
- Exemplos:
    - Segurança física
    - Controlo de acessos
    - Política de passwords
    - Política de email
    - Política de acesso remoto

## 11) Modelo de Confiança
- Define o conjunto de atores confiáveis e que temos como pressuposto garantido que não são possíveis adversários
- Um sistema é confiável se fizer exatamente (e apenas) aquilo que foi especificado

## 12) Engenharia de Segurança
**1) Definir o problema**
- Análise de requisitos de segurança
- Definir o modelo de ameaças
- Matriz de gestão de risco

**2) Definir o modelo de confiança**
- Em quais componentes/atores confiamos?
- O que fazem eles?

**3) Definir a solução**
- Conceber as políticas de segurança que se aplicam ao sistema (o quê)
- Identificar os mecanismos de segurança necessários e suficientes (como)

**4) Validar e justificar a solução**
- Se os modelos de confiança são adequados para o sistema
- Demonstrar que os mecanismos e o modelo de confiança são suficientes para implantar as políticas de segurança

**5) Auditorias de segurança**
- Testes de penetração e Ethical hacking
- Simulação de um ataque de procura de vulnerabilidades que poderiam ser exploradas
    - **Black box:** semelhante a um ataque real
    - **White box:** com conhecimento privilegiado