# CTF - Semana 12

**Objetivo**  
Explorar o mecanismo RSA quando existe algum conhecimento sobre os números primos utilizados para o problema matemático associado.

São dadas algumas informações sobre a cifra usada:
- O expoente público (*e*)
- O módulo (*n*)
- O criptograma

Sabemos ainda que:
- O texto está no formato habitual das flags: ```flag{...}```
- p é um primo próximo de 2^(500 + (((t-1)*10 + g) // 2))
- q é um primo próximo de 2^(501 + (((t-1)*10 + g) // 2))

*Nota: t é a nossa turma (9) e g é o nosso grupo (3).*

## Procedimento
Com ajuda de Inteligência Artifical, escrevemos um script capaz de executar todo o processo de obtenção de p e q e, consequentemente, da decifração do texto. Encontra-se abaixo:

``` py
import random, re

t = 9
g = 3
e = 65537
n = 103629953693343036596477617520085661209373004679240764598733495310044333858813617608683716455135807741535783866530207698631582073541526260155910860769658117079841719242437499117069108452206091898720850821894125639585399196229679110437130159233568245108299395338938054070206675456917141258977284264312570142490475146609372882023
ciphertext = bytes.fromhex("7976c9833907a20f6b7a60e5a0745edea85e59fb88930c4b8fcede6e7c6edcf8cd0a469e15e89ac83efffebf9b6c7941d19f72de97b350c6ea3aa0bf1529f2a3e2c5ce97ccdd47148acf2136173546314cf5ef1c6e67226c5dc4b5c93fd195c4c9b414f5da1dcfe1fc32361a81f71515f1297ee9324bc008daa0538f5670381803b136b6cd2b6501000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000")

# Testa a primalidade de números
def is_prime(n, k=5):
    if n == 2 or n == 3:
        return True
    if n % 2 == 0:
        return False
    
    r, s = 0, n - 1
    while s % 2 == 0:
        r += 1
        s //= 2
    
    for _ in range(k):
        a = random.randint(2, n - 2)
        x = pow(a, s, n)
        if x == 1 or x == n - 1:
            continue
        for _ in range(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True

# Obter a chave privada
def extended_euclidean(a, b):
    old_r, r = a, b
    old_s, s = 1, 0
    old_t, t = 0, 1
    
    while r != 0:
        quotient = old_r // r
        old_r, r = r, old_r - quotient * r
        old_s, s = s, old_s - quotient * s
        old_t, t = t, old_t - quotient * t
    
    return old_s % b

p = 2**(500 + (((t-1)*10 + g) // 2))
q = 2**(501 + (((t-1)*10 + g) // 2))

phi_n = (p - 1) * (q - 1)
d = extended_euclidean(e, phi_n)

# Obter os primos próximos de p e q
def find_closest_primes(center, count=5):
    primes_below = []
    primes_above = []
    candidate_below = center - 1
    candidate_above = center + 1
    
    while len(primes_below) < count:
        if is_prime(candidate_below):
            primes_below.append(candidate_below)
        candidate_below -= 1
    
    while len(primes_above) < count:
        if is_prime(candidate_above):
            primes_above.append(candidate_above)
        candidate_above += 1
    
    return primes_below[::-1] + primes_above

# Descriptografar
def dec(x, d, n):
    int_x = int.from_bytes(x, "little")
    m = pow(int_x, d, n)
    return m.to_bytes(256, 'little')

primes_p = find_closest_primes(p)
primes_q = find_closest_primes(q)

# Testa combinações de p e q
for p in primes_p:
    for q in primes_q:
        if p == q:
            continue
        
        phi_n = (p - 1) * (q - 1)
        d = extended_euclidean(e, phi_n)
        
        decrypted_message = dec(ciphertext, d, n)
        try:
            decoded_message = decrypted_message.decode('utf-8', errors='ignore')
            flag_pattern = re.compile(r'flag{.*?}')
            match = flag_pattern.search(decoded_message)
            if match:
                print(f"p: {p}\nq: {q}")
                print("Flag encontrada:", match.group())
        except UnicodeDecodeError:
            continue
```

As partes essenciais do script são:
- is_prime(): testa se um número é primo, usando o algoritmo de Miller-Rabin;
- extended_euclidean(): algoritmo estendido de Euclides. Usado para encontrar a chave privada *d* a partir do expoente *e* e de *ϕn = (p-1)***(q-1)*;
- find_closest_primes(): obtém intervalos de números primos próximos de p e q. Podemos ajustar o tamanho dos intervalos caso a solução não seja encontrada;
- o loop principal, que testa combinações de p e q e, em cada iteração, compara o texto decifrado com o formato das flags.

## Questões
**Como consigo usar a informação que tenho para inferir os valores usados no RSA que cifrou a flag?**  
Como são dados o *e*, *n* e valores aproximados para *p* e *q*, podemos inferir os valores dos dois primos tal que n=p*q.

**Como consigo descobrir se a minha inferência está correta?**  
A forma mais direta é descriptografar a mensagem, comparando-a, neste caso, com o formato ```flag{...}```. É necessário termos também o valor de *d*.

**Finalmente, como posso extrair a minha chave do criptograma que recebi?**  
Usando os valores inferidos (*n*, *p* e *q*) e os valores de *ϕ(n)* e *d*, descriptografamos a mensagem e obtemos a flag.

## Resultados  
### Valores de *p* e *q*:
```
p: 7198262071269114212496861612297570974191515389283066612961208916178940129074380592510465097766225371439873457013633432197133225688790879502413624289384262168216917
q: 14396524142538228424993723224595141948383030778566133225922417832357880258148761185020930195532450742879746914027266864394266451377581759004827248578768524336431819
```

### Flag
flag{ujhimxpadeznadow}
