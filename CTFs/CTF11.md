# CTF - Semana 11

**Objetivo**  
Decifrar um criptograma sem ter acesso à sua chave simétrica, explorando o facto de se tratar de criptografia mais antiga.

**Script de Geração de Chaves**  
É dado o script em Python responsável por gerar chaves AES-CTR. Contém ainda os algoritmos para criptografar e descriptografar texto.

``` py
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)

def enc(k, m, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph

def dec(k, c, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg
```

## Tarefa 1
**Como consigo usar esta ciphersuite para cifrar e decifrar dados?**  
Para cifrar dados:
- Utilizar gen() para gerar a chave
- Gerar um nonce aleatório de 16 bytes (adequado para AES-CTR)
- Encriptar a mensagem com enc(), passando a chave, a mensagem e o nonce como argumentos

Para decifrar dados:
- Utilizar a mesma chave e nonce como argumentos para dec()

**Como consigo fazer uso da vulnerabilidade que observei para quebrar o código?**  
Na função gen(), é definido um offset de 3 bytes. Assim, a chave é gerada com valores fixos nos primeiros 13 bytes, sendo variáveis apenas os 3 últimos bytes correspondentes ao offset.  
Num cenário de um ataque de **bruteforce**, esta configuração diminui consideravelmente o número de tentativas necessárias para obter a chave e, tratando-se apenas de 3 bytes variáveis, a utilização deste método de ataque torna-se viável em relação ao tempo e ao poder computacional necessários.

**Como consigo automatizar este processo, para que o meu ataque saiba que encontrou a flag?**  
Sabemos que as flags seguem um padrão específico: ```flag{...}```. Logo, podemos dizer ao programa para esperar por uma mensagem com esse formato.

Elaborámos um script que automatiza todo o processo. São passados o nonce e o criptograma que foram dados e o programa faz um ataque de bruteforce aos últimos bytes da chave. Na execução, vai comparando com o formato das flags, retornando a flag caso a encontre. O script encontra-se abaixo:

``` py
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from itertools import product

nonce = bytes.fromhex("cad8f873f4ba5ca502f24e2e815fc56e")
ciphertext = bytes.fromhex("c705340aa49a3feac43017eb3a25299ac284741a14f0")

def dec(k, c, nonce):
    cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
    decryptor = cipher.decryptor()
    msg = decryptor.update(c) + decryptor.finalize()
    return msg

fixed_part = b'\x00' * 13
for key_suffix in product(range(256), repeat=3):
    key = fixed_part + bytes(key_suffix)
    try:
        plaintext = dec(key, ciphertext, nonce)
        if b"flag{" in plaintext:
            print(f"Chave encontrada: {key.hex()}")
            print(f"Mensagem decifrada: {plaintext.decode(errors='ignore')}")
            break
    except Exception as e:
        pass
```

Ao executar o script, encontrámos a flag muito rapidamente, em menos de 1 segundo.

### Flag
flag{khbfdkjewsmlykhw}

*Nota: caso o texto não seguisse o formato das flags, poderíamos verificar se o output é legível (se é ASCII).*

## Tarefa 2
Adaptámos o nosso script para medir o tempo e o número de tentativas necessárias:
- Tempo: 0.783685 segundos
- Nº de tentativas: 31235

Com estes dados, podemos calcular o tamanho necessário do offset para inviabilizar um ataque de bruteforce.

**Cálculo**  
Tentativas por byte: 256  
Segundos em 10 anos: 315360000  
Tentativas/segundo: 31235/0.783685 = 39856  
Tentativas em 10 anos: 315360000 * 39856 = 1.257 * 10^13

Valor de n, tal que 256^n > tentativas em 10 anos: 6 (2.81 * 10^14 tentativas)

Assim, bastaria alterar o offset de 3 para **6 bytes** para inviabilizar um ataque de bruteforce.

*Nota: é considerado o pior caso para o bruteforce e a paralelização não é considerada*

## Tarefa 3
O uso de um nonce de 1 byte não resolve os problemas de fundo que o algoritmo tem, nomeadamente o offset demasiado curto. O único aspeto que muda é que o atacante tem de tentar 256 combinações para o nonce em cada tentativa de decifrar a chave. Considerando o offset e o nonce, o pior caso seria de 4,294,967,296 tentativas, o que continua a ser viável para um ataque de bruteforce:

Tentativas/segundo: 39856  
Tempo necessário (pior caso): 4,294,967,296 / 39856 = 107,762 segundos -> aprox. 75 dias

Este tempo, apesar de parecer grande, pode ser reduzido caso o atacante tenha informações adicionais sobre a forma como o algoritmo de cifra funciona ou se tem acesso a vários criptogramas.  
Assim, a segurança deve ser focada na chave e no ajuste do tamanho do offset, e não em esconder o nonce de 1 byte.