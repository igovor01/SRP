# Lab 1 : Man-in-the-middle attack (ARP spoofing)

Iskorištavanjem ranjivosti ARP protokola (*Adress Resolution Protocol*) realizirali smo Man-in-the-middle i Denial-of-service napade. 

Koristili smo se virtualnom mrežom Docker sa 3 virtualna containera, od toga dvije žrtve (*station-1* i *station-2*) i jedan napadač (*evil-station*).

Pokrenuli smo Windows terminal odakle smo komandom ***wsl*** otvorili Ubuntu terminal na WSL (*Windows subsystem for Linux*) sistemu.

Naredbom **mkdir** (*make directory*) iz terminala smo napravili mapu s imenom predmeta te podmapu s našim imenom i u taj direktorij klonirali git repozitorij:

`git clone https://github.com/mcagalj/SRP-2022-23`

Zatim smo ušli u direktorij arp-spoofing komandom **cd** (*change directory*):

`cd SRP-2022-23/arp-spoofing/`

U direktoriju se nalaze sredstva potrebna da izvedemo napad, između ostalog skripte `start.sh` i `stop.sh` koje služe za pokretanje i zaustavljanje docker containera.

(Ako želimo vidjeti što se nalazi u skriptama, naredbom **code .** možemo otvoriti trenutni direktorij u VS Code-u)

Slijedi nam pokretanje virtualnog programa :

`./start.sh`.

Zatim pokrećemo shell za station-1:

`docker exec -it station-1 bash`

Provjeravamo nalazi li se station-2 na istoj mreži naredbom **ping:**

```````````````ping station-2```````````````.

Dijelimo terminal na dva dijela (**alt shift+**) i pokrećemo shell i za station-2:

`docker exec -it station-2 bash`

U station-2 dijelu terminala pišemo:

`netcat -1 8080`

A u station-1 dijelu terminala:

`netcat station-2 8080`

i time uspostavljamo komunikaciju između station-1 i station-2.

Dalje dijelimo terminal horizontalno(**alt shift -**) i pokrećemo interaktivni shell za evil-station:

`docker exec -it evil-station bash`

Naredbom **arpspoof** presrećemo komunikaciju između station-1 i station-2:

````````````````````````````````arpspoof -t station-1 station-2```````````````````````````````` (evil-station se stationu-1 predstavlja kao station-2)

Ovime smo omogućili presretanje i prisluškivanje komunikacije između dva stationa od strane evil-stationa, čime smo narušili **povjerljivost** i **integritet**.

Naredbom **tcpdump** pratimo promet između dva containera.

Za kraj smo napravili i DoS napad onemogućavanjem komunikacije stationa-1 i stationa-2 (evil-station zaustavio je prosljeđivanje podataka):

`echo 0 > /proc/sys/net/ipv4/ip_forward`

Ovime narušavamo **dostupnost**.

Za ponovnu uspostavu komunikaciju i nastavak prosljeđivanja podataka koristimo:

`echo 1 > /proc/sys/net/ipv4/ip_forward`