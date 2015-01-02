----------------------------------------------------------------------------
))                     O APLIKACI/AUTOROVI                                ((
----------------------------------------------------------------------------

N�zev               : RDT klient/server
Verze               : 1.0
OS                  : FreeBSD/Linux
Datum vyd�n�        : 21. 4. 2011
Jazyk               : Angli�tina

Autor               : Radim Loskot
Kontakt             : xlosko01(at)stud.fit.vutbr.cz

Aplikace implementuj� spolehliv� p�enos dat na lok�ln�m rozhran�.

Pro spu�t�n� je nutno v�dy specifikovat zdrojov� port a c�lov� port pro p�enos.

U�it�:
    rdtclient �s source_port �d dest_port
    rdtserver �s source_port �d dest_port        

----------------------------------------------------------------------------
))                         STRUKTURA PAKETU                               ((
----------------------------------------------------------------------------

Struktura paketu je zobrazena v n�sleduj�c� tabulce:

       -----------------------------------------------------------------
      | ZAROVN�N�  |       DATOV� TYP        |    DATOV� REPREZENTACE   |
       -----------------------------------------------------------------
      |     0      |     unsigned short      |    kontroln� sou�et      |
      |     2      |      unsigned int       |    ��slo sekvence        |    
      |     6      |     unsigned short      |    velikost dat          |
      |     8      |     unsigned short      |    vlaje�ky (flags)      |
      |    10      |    (nespecifikovan�)    |    data                  |
       -----------------------------------------------------------------

Popis jednotliv�ch polo�ek:
    1) Prvn� polo�ku p�edstavuje 16-bitov� kontroln� sou�et. 
       Metodika byla p�evzata z normy RFC 1071.
    2) ��slo sekvence je jedine�n� identifik�tor pro specifikaci p�en�en�ch
       dat (klientem) nebo potvrzov�n� ACK/NACK (serverem). 
    3) Velikost dat zna�� po�et byt� p�en�en�ch dat.
    4) Vlaje�ky (flags) slou�� pro specifikaci typu paketu.
       Pou�ity byly tyto vlaje�ky:
       * ACK  (0x01) - Jsou potvrzov�na data s poslan�m ��slem sekvence.
       * NACK (0x02) - Negativn� potvrzov�na data s poslan�m ��slem sekvence.
       * END  (0x04) - Ukon�en� p�enosu dat se serverem.
    4) Data jsou nespecifikovan�ho typu - lze tedy zas�lat i binarn� data.        
    
----------------------------------------------------------------------------
))                               KLIENT                                   ((
----------------------------------------------------------------------------

Klient �te ��dky ze STDIN do d�lky 80 znak�, pokud d�lka p�esahuje limit 
dojde k o��znut� ��dku. ��dek je pot� posl�n serveru. 

Popis b�hu klienta:
    
    Po spu�t�n� za�le prvn� paket s daty serveru, spust� se �asova�
    a �ek� se na STDIN vstup nebo p��jem paketu od serveru.
    
    1) Klient dostane odpov�� ACK - data s dan�m ��slem sekvence jsou 
       odstran�na z okna, co� zp�sob� p��padn� posunut� okna.
    2) Klient dostane odpov�� NACK - data do dan�ho ��sla sekvence jsou 
       odstran�na z okna a je zasl�n op�t paket s po�adovan�m ��slem sekvence.
    3) Klient dostane paket se �patn�m kontroln�m sou�tem - dojde k 
       posl�n� prvn�ho paketu z okna.
    4) Vypr�� doba �asova�e - dojde k posl�n� v�ech paket� z okna, 
       u kter�ch do�lo k vypr�en� limitu pro potvrzen�. �asova� se op�t zapne.
    5) �te se STDIN vstup a pos�l� se paket serveru.
       Pokud dojde k zapln�n� okna, je vstup pozastaven dokud nedojde k jeho
       zp��stupn�n� - spu�ten� invokuje server potvrzen�m prvn� zpr�vy v okn�.
    6) Bylo dosa�eno konce STDIN - ukon�en� �ten� ze STDIN a odstran�n� STDIN 
       z mno�iny popisova�� soubor� pro �ten�. Nyn� se �ek� na potrvrzen� 
       posledn� zpr�vy serverem.
    7) Klient dostal potvrzen� posledn� zpr�vy - doch�z� k ukon�en� 
       p�enosu se serverem a zasl�n� paketu s END vlaje�kou.  

----------------------------------------------------------------------------
))                               SERVER                                   ((
----------------------------------------------------------------------------

Server p�ij�m� data od klienta a� do ukon�en� p�enosu klientem. 

Popis b�hu serveru:
    
    Server b�� a �ek�.
    
    1) Server dostane nov� data - dojde k vlo�en� do bufferu. Pokud byla vlo�ena 
       data na za��tek, dojde k vytisknut� bufferu a� do prvn�ho pr�zdn�ho m�sta.
       Nakonec dojde odesl�n� paketu s vlaje�kou ACK a dan�m ��slem sekvence. 
    2) Server dostane data podruh� - dojde pouze k posl�n� potvrzen� ACK 
       s dan�m ��slem sekvence dat.
    3) Server dostane paket se �patn�m kontroln�m sou�tem - dojde k 
       posl�n� paketu s NACK vlaje�kou a ��slem sekvence po�adovan�ch dat.
    4) Server dostane paket s vlaje�kou END - aplikace se ��dn� ukon��.
       