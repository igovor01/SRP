# Lab 4: Message authentication and integrity

**Zadatak 1: Zaštita integriteta sadržaja poruke putem autentikacije i MAC algoritma**

1. dio posla: kreirati ili potpisati poruku (glumimo nekoga tko šalje poruku)
    1. otvaramo file koji treba potpisat
    2. potpisujemo content
    3. dodat nekako potpis na file, nekako ih mergat, (na koji način implementirat?)
    -- potpis kao niz bitova ćemo spremit u odvojeni file(i to je valjano)

Druga strana primi poruku

1. dio posla: trebamo verificirati dobivenu poruku
    - 1. pročitat ćemo file koji smo dobili
    - 2.  pročitat potpis
    - 3.1. na našem računalu primatelja potpišemo (poznatim ključem) dobiveni file
    - 3.2. usporedimo potpis primljeni i potpis generiran lokalno
    
    (Kodiramo po zadanom planu)
    

Kod:

```python
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.exceptions import InvalidSignature

def generate_MAC(key, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    signature = h.finalize()
    return signature

def verify_MAC(key, signature, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    try:
        h.verify(signature)
    except InvalidSignature:
        return False
    else:
        return True

if __name__ == "__main__":
    # # 1.Sign the file content

    # # 1.1. Read the file content
    # with open("message.txt", "rb") as file:
    #     content = file.read()
        
    # # 1.2. Sign the content
    # key = "my super secure secret".encode()
    # signature = generate_MAC(key, content)
    # print(signature)
    
    # # 1.3. Save the signature into a file
    # with open("message.sig", "wb") as file:
    #     file.write(signature)
        
        
    # 2. Verify message authenticity

    # 2.1. Read the received file
    with open("message.txt", "rb") as file:
        contentRec = file.read()
        
    # 2.2. Read the received signature
    with open("message.sig", "rb") as file:
        signature = file.read()
        
    # 2.3.1  Sign the received file
    # 2.3.2. Compare locally generated signature with the received one
    key = "my super secure secret".encode()
    is_authentic = verify_MAC(key=key, signature=signature, message=contentRec)
    print(f"Message is {'OK' if is_authentic else 'NOK'}")
```

**Zadatak 2: Sve transakcije poredat po freshnessu i vidjeti jesu li autentične**

Postupak:

- provjerimo autentičnost poruke (iskoristimo 1.zadatak)
- ispišemo rezultat provjere(OK/NOK)
- one koje se pokažu autentičnima sortiramo u sekvencu

Problem može nastati ako poruke/file-ovi ni ne dođu, to možemo riješiti dodavanjem rednih brojeva na pakete.

Kod:

```python
import datetime
import re
from pathlib import Path

from cryptography.hazmat.primitives import hashes, hmac
from cryptography.hazmat.primitives import hashes, hmac
from cryptography.exceptions import InvalidSignature

def generate_MAC(key, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    signature = h.finalize()
    return signature

def verify_MAC(key, signature, message):
    if not isinstance(message, bytes):
        message = message.encode()

    h = hmac.HMAC(key, hashes.SHA256())
    h.update(message)
    try:
        h.verify(signature)
    except InvalidSignature:
        return False
    else:
        return True

if __name__ == "__main__":
#     # # 1.Sign the file content

#     # # 1.1. Read the file content
#     # with open("message.txt", "rb") as file:
#     #     content = file.read()
        
#     # # 1.2. Sign the content
#     # key = "my super secure secret".encode()
#     # signature = generate_MAC(key, content)
#     # print(signature)
    
#     # # 1.3. Save the signature into a file
#     # with open("message.sig", "wb") as file:
#     #     file.write(signature)
        
        
#     # 2. Verify message authenticity

#     # 2.1. Read the received file
#     with open("message.txt", "rb") as file:
#         contentRec = file.read()
        
#     # 2.2. Read the received signature
#     with open("message.sig", "rb") as file:
#         signature = file.read()
        
#     # 2.3.1  Sign the received file
#     # 2.3.2. Compare locally generated signature with the received one
#     key = "my super secure secret".encode()
#     is_authentic = verify_MAC(key=key, signature=signature, message=contentRec)
#     print(f"Message is {'OK' if is_authentic else 'NOK'}")

    PATH = 'challenges/g2/govorusic_ivana/mac_challenge/'
    KEY = "govorusic_ivana".encode()
    authentic_messages = []
    for ctr in range(1, 11):
        msg_filename = f"order_{ctr}.txt"
        sig_filename = f"order_{ctr}.sig"    
        # print(msg_filename)
        # print(sig_filename)
        
        msg_file_path = Path(PATH+msg_filename)
        with open(msg_file_path, "rb") as file:
            message = file.read()
        
        
        sig_file_path = Path(PATH+sig_filename)
        with open(sig_file_path, "rb") as file:
            signature = file.read()
            
        #print(message)
        #print(signature)
        
        is_authentic = verify_MAC(key=KEY, signature=signature, message=message)
        # Printamo i vidimo koje su datoteke OK a koje NOK
        # print(f'Message {message.decode():>45} {"OK" if is_authentic else "NOK":<6}')
        
        # Dodajemo samo autentične podatke u listu authentic_messages
        if is_authentic:
            authentic_messages.append(message.decode())
            
    #print(authentic_messages)
    
    
    #Sortiraj
    authentic_messages.sort(
        key= lambda m: datetime.datetime.fromisoformat(
            re.findall(r'\(.*?\)', m)[0][1:-1]
        )
    )
    
    for m in authentic_messages:
        print(f'Message {m:>45} {"OK":<6}')
```