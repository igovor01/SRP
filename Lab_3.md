# Lab 3: Symmetric key cryptography (a crypto challenge)

****IZAZOV - dešifriranje ciphertexta korištenjem brute-force napada****

Za početak izazova, dobili smo niz datoteka koji su naši potencijalni izazovi. Morali smo hashirati naše ime i prezime kako bi usporedbom te vrijednosti i naziva datoteka saznali koji je baš naš izazov.

```python
		# Hash of the name
    filename = hash('govorusic_ivana') + ".encrypted"
```

Preuzeli smo datoteku(izazov) čije je ime naše hashirano ime, a čiji sadržaj čeka na dekripciju. 

Izazov je enkriptiran simetričnim ključem entropije 22 bita, pa nam je cilj bio brute force napadom doći do točnih 22bita ključa. **Tražimo ključ.**

Spremamo sadžaj naše datoteke(izazova) u varijablu ciphertext.

```python
		# Create a file with the filename if it does not already exist
    if not path.exists(filename):
        with open(filename, "wb") as file:
            file.write(b"")
            
    #Open your challenge file and read in your challenge
    with open(filename, "rb") as file:
        ciphertext = file.read()
```

Generirat ćemo ključeve, redom ih provjeravati, ali kako provjeravati?

Znamo da je enkriptirani izazov zapravo png slika, a za taj format ćemo uvijek imati nekakve karakteristične podatke tj. bytove. 

Za svaki generirani ključ ispitivat ćemo počinje li dokument sa specifičnim formatom za png dok ne dođemo do podudaranja.

```python
# Konkretno testiranje podudara li se header dokumenta s formatom png-a
def test_png(header):
    if header.startswith(b"\211PNG\r\n\032\n"):
        return True
    return False
```

Cjeloviti kod:

```python
import base64
from os import path
from pydoc import plain
from cryptography.hazmat.primitives import hashes
from cryptography.fernet import Fernet

def hash(input):
    if not isinstance(input, bytes):
        input = input.encode()

    digest = hashes.Hash(hashes.SHA256())
    digest.update(input)
    hash = digest.finalize()

    return hash.hex()

def test_png(header):
    if header.startswith(b"\211PNG\r\n\032\n"):
        return True
    return False

def brute_force(ciphertext):
    ctr = 0
    while True:
        key_bytes = ctr.to_bytes(32, "big")
        key = base64.urlsafe_b64encode(key_bytes)

        # Now initialize the Fernet system with the given key
        # and try to decrypt your challenge.
        # Think, how do you know that the key tested is the correct key
        # (i.e., how do you break out of this infinite loop)?
        
        try:
            plaintext = Fernet(key).decrypt(ciphertext) 
            header = plaintext[:32]
            
            if test_png(header):
                print(f"BINGO:{key}")
                with open("BINGO.png", "wb") as file:
                    file.write(plaintext)
                break
        except Exception:
            pass

        ctr += 1
        if not ctr % 1000:
            print(f"[*] Keys tested: {ctr:,}", end="\r")

if __name__ == "__main__":
    filename = hash('govorusic_ivana') + ".encrypted"
    
    # Create a file with the filename if it does not already exist
    if not path.exists(filename):
        with open(filename, "wb") as file:
            file.write(b"")
            
    #Open your challenge file and read in your challenge
    with open(filename, "rb") as file:
        ciphertext = file.read()
    
    #print(ciphertext)
    
    #Start the attack
    brute_force(ciphertext)
```

Rezultat: 
![BINGO](https://user-images.githubusercontent.com/92445348/205782604-7dfcc851-78cb-4af4-b743-a091c33cd65e.png)
