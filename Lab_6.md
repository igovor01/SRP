# Lab 6: Online and Offline Password Guessing

**Online Password Guessing**

- Za početak, otvaramo terminal iz bilo kojeg direktorija(SRP/igovor) i upisujemo `wsl` da se prebacimo na linux.
- skeniramo mrežu preko nmapa
    - `nmap -v 10.0.15.0/28`
    
    > *nmap is an open-source Linux command-line tool that is used to scan IP addresses and ports in a network and to detect installed applications*
    > 
- vidimo : da ima određeni broj računala povezan na toj mreži i njihove ip adrese
- vidimo docker containere i vidimo portove na kojima npr. sluša sshd(servis koji je otvorio port) u docker računalu(port 22)
- svatko od nas ima za sebe jedan docker container, i u njemu korisnički račun s podacima:
    
    > username: govorusic_ivana, hostname: govorusicivana.local
    > 

pokušat ćemo s ovog lokalnog računala otvorit i spojit se na port 22 koristeći ove gore informacije

- pišemo:
    
    `ssh govorusic_ivana@govorusicivana.local`
    
- lokalno računalo nas pita vjerujemo li ovom računalu na koje se spajamo (govorusicivana.local)
- ne možemo se spojit jer nam treba password, sad trebamo naći password(to nam je zadatak)
- password dug od 4 do 6 znakova, i sve mala slova

- **Q1:** Estimate the password space.
    - 26^4 + 26^5 + 26^6 - toliko kombinacija passworda može bit - otprilike 320 milijuna
    - zanemarimo prva dva pribrojnika jer je 26^6 znatno veći
- koristimo hydru (`man hydra` za dobit više infromacija)
    
    > *hydra: password cracking tool that uses various network protocols to perform rapid dictionary attacks*.
    > 
- `hydra -l govorusic_ivana -x 4:6:a govorusicivana.local -V -t 1 ssh`
    - napadmo tu domenu, sifre of 4-6 i protokolom ssh
- **Q2:** Using the output produced by `hydra`try to estimate your effort, that is how long it would take, on average, before you succeed. You can also try to play with parameter `-t` (**IMPORTANT:** Test values 2, 3, 4 but please do not exaggerate to avoid crushing the server).
    - hydra je na strani lokalnog računala
    - predugo traje, da bi sve probali trebalo bi nam 9, 10 godina
- **Q3:** What do your do if the estimated time from the previous question is prohibitively large?
    - Pripremit ćemo dictionary(7.korak)
- pokrenemo 8.korak i probamo detektirat ispravnu lozinku iz dictionaryja
- kad dobijemo lozinku probamo se prijavit na onaj server s početka  `ssh govorusic_ivana@govorusicivana.local`

- pronađena lozinka za govorusicivana: **fiofow**
    
    ![Untitled](Lab%206%20Online%20and%20Offline%20Password%20Guessing%20d6cbf2fc7edb439ba8e1d500e093d714/Untitled.png)
    

**Offline Password Guessing**

- cat /etc/passwd - za pronaći listu svih korisničkih računa na svom ovom računalu govorusic_ivana
- ispišu nam se svi korisnici, a za jedan od računa ćemo pokušat naći lozinku offline napadom i prijavit se kasnije
    - npr. alice_cooper , username: alice_cooper, pasword: tražimo, hostname: govorusicivana.local
- password ima 6 znakova, lowercase

- gdje naći password hasheve u linuxu: (označeni dio je hash)
    
    ![Untitled](Lab%206%20Online%20and%20Offline%20Password%20Guessing%20d6cbf2fc7edb439ba8e1d500e093d714/Untitled%201.png)
    
- kopirat od alice cooper hash:
    
    > $6$rR6Ss1YUv72kRtM0$jJd2mkJnZiry/vd1c/bk/f9BAo5SMMZGCmE6ZFlb9TjTXsNejjtKsjDeqilZ2vU..XQ.2w3O4QEGLXpaTRyYu/
    > 
- otvorit još jedan terminal, doći u istu onu mapu igovor, upisat `code pwd_hash.txt`i otvorit će nam se dokument u visual studio codu i onda u taj dokument samo zalijepimo hash od alice.
- koristimo hashcat
    
    > *hashcat is a password recovery tool for offline dictionary attacks*
    > 
- pišemo:
    
    `hashcat --force -m 1800 -a 0 pwd_hash.txt dictionary/g2/dictionary_offline.txt --status --status-timer 10`
    
- nađemo lozinku od alicecooper: **typowh**
    
    ![Untitled](Lab%206%20Online%20and%20Offline%20Password%20Guessing%20d6cbf2fc7edb439ba8e1d500e093d714/Untitled%202.png)
    

- uspješno se logiramo s pronađenom lozinkom