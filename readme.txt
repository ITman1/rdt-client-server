----------------------------------------------------------------------------
))                     O APLIKACI/AUTOROVI                                ((
----------------------------------------------------------------------------

Název               : RDT klient/server
Verze               : 1.0
OS                  : FreeBSD/Linux
Datum vydání        : 21. 4. 2011
Jazyk               : Angliètina

Autor               : Radim Loskot
Kontakt             : xlosko01(at)stud.fit.vutbr.cz

Aplikace implementují spolehlivý pøenos dat na lokálním rozhraní.

Pro spu¹tìní je nutno v¾dy specifikovat zdrojový port a cílový port pro pøenos.

U¾ití:
    rdtclient –s source_port –d dest_port
    rdtserver –s source_port –d dest_port        

----------------------------------------------------------------------------
))                         STRUKTURA PAKETU                               ((
----------------------------------------------------------------------------

Struktura paketu je zobrazena v následující tabulce:

       -----------------------------------------------------------------
      | ZAROVNÁNÍ  |       DATOVÝ TYP        |    DATOVÁ REPREZENTACE   |
       -----------------------------------------------------------------
      |     0      |     unsigned short      |    kontrolní souèet      |
      |     2      |      unsigned int       |    èíslo sekvence        |    
      |     6      |     unsigned short      |    velikost dat          |
      |     8      |     unsigned short      |    vlajeèky (flags)      |
      |    10      |    (nespecifikovaný)    |    data                  |
       -----------------------------------------------------------------

Popis jednotlivých polo¾ek:
    1) První polo¾ku pøedstavuje 16-bitový kontrolní souèet. 
       Metodika byla pøevzata z normy RFC 1071.
    2) Èíslo sekvence je jedineèný identifikátor pro specifikaci pøená¹ených
       dat (klientem) nebo potvrzování ACK/NACK (serverem). 
    3) Velikost dat znaèí poèet bytù pøená¹ených dat.
    4) Vlajeèky (flags) slou¾í pro specifikaci typu paketu.
       Pou¾ity byly tyto vlajeèky:
       * ACK  (0x01) - Jsou potvrzována data s poslaným èíslem sekvence.
       * NACK (0x02) - Negativnì potvrzována data s poslaným èíslem sekvence.
       * END  (0x04) - Ukonèení pøenosu dat se serverem.
    4) Data jsou nespecifikovaného typu - lze tedy zasílat i binarní data.        
    
----------------------------------------------------------------------------
))                               KLIENT                                   ((
----------------------------------------------------------------------------

Klient ète øádky ze STDIN do délky 80 znakù, pokud délka pøesahuje limit 
dojde k oøíznutí øádku. Øádek je poté poslán serveru. 

Popis bìhu klienta:
    
    Po spu¹tìní za¹le první paket s daty serveru, spustí se èasovaè
    a èeká se na STDIN vstup nebo pøíjem paketu od serveru.
    
    1) Klient dostane odpovìï ACK - data s daným èíslem sekvence jsou 
       odstranìna z okna, co¾ zpùsobí pøípadné posunutí okna.
    2) Klient dostane odpovìï NACK - data do daného èísla sekvence jsou 
       odstranìna z okna a je zaslán opìt paket s po¾adovaným èíslem sekvence.
    3) Klient dostane paket se ¹patným kontrolním souètem - dojde k 
       poslání prvního paketu z okna.
    4) Vypr¹í doba èasovaèe - dojde k poslání v¹ech paketù z okna, 
       u kterých do¹lo k vypr¹ení limitu pro potvrzení. Èasovaè se opìt zapne.
    5) Ète se STDIN vstup a posílá se paket serveru.
       Pokud dojde k zaplnìní okna, je vstup pozastaven dokud nedojde k jeho
       zpøístupnìní - spu¹tení invokuje server potvrzením první zprávy v oknì.
    6) Bylo dosa¾eno konce STDIN - ukonèení ètení ze STDIN a odstranìní STDIN 
       z mno¾iny popisovaèù souborù pro ètení. Nyní se èeká na potrvrzení 
       poslední zprávy serverem.
    7) Klient dostal potvrzení poslední zprávy - dochází k ukonèení 
       pøenosu se serverem a zaslání paketu s END vlajeèkou.  

----------------------------------------------------------------------------
))                               SERVER                                   ((
----------------------------------------------------------------------------

Server pøijímá data od klienta a¾ do ukonèení pøenosu klientem. 

Popis bìhu serveru:
    
    Server bì¾í a èeká.
    
    1) Server dostane nová data - dojde k vlo¾ení do bufferu. Pokud byla vlo¾ena 
       data na zaèátek, dojde k vytisknutí bufferu a¾ do prvního prázdného místa.
       Nakonec dojde odeslání paketu s vlajeèkou ACK a daným èíslem sekvence. 
    2) Server dostane data podruhé - dojde pouze k poslání potvrzení ACK 
       s daným èíslem sekvence dat.
    3) Server dostane paket se ¹patným kontrolním souètem - dojde k 
       poslání paketu s NACK vlajeèkou a èíslem sekvence po¾adovaných dat.
    4) Server dostane paket s vlajeèkou END - aplikace se øádnì ukonèí.
       