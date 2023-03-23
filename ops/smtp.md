# Bou jou eie SMTP-pos stuur bediener

## aanhef

SMTP kan dienste direk van wolkverkopers koop, soos:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali wolk e-pos druk](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Jy kan ook jou eie posbediener bou - onbeperkte versending, lae algehele koste.

Hieronder demonstreer ons stap vir stap hoe om ons eie posbediener te bou.

## Bediener keuse

Die SMTP-bediener wat self aangebied word, vereis 'n openbare IP met poorte 25, 456 en 587 oop.

Algemeen gebruikte publieke wolke het hierdie poorte by verstek geblokkeer, en dit kan moontlik wees om dit oop te maak deur 'n werkbestelling uit te reik, maar dit is tog baie lastig.

Ek beveel aan om by 'n gasheer te koop wat hierdie poorte oop het en die opstel van omgekeerde domeinname ondersteun.

Hier beveel ek [Contabo](https://contabo.com) aan.

Contabo is 'n gasheerverskaffer gebaseer in München, Duitsland, gestig in 2003 met baie mededingende pryse.

As jy Euro as die geldeenheid van aankoop kies, sal die prys goedkoper wees ('n bediener met 8GB geheue en 4 SVE's kos ongeveer 529 yuan per jaar, en die aanvanklike installasiefooi is gratis vir een jaar).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

As u 'n bestelling plaas, moet u `prefer AMD` , en die bediener met AMD CPU sal beter werkverrigting hê.

In die volgende sal ek Contabo se VPS as voorbeeld neem om te demonstreer hoe om jou eie posbediener te bou.

## Ubuntu-stelselkonfigurasie

Die bedryfstelsel hier is Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

As die bediener op ssh vertoon `Welcome to TinyCore 13!` (soos in die figuur hieronder getoon), beteken dit dat die stelsel nog nie geïnstalleer is nie. Ontkoppel asseblief ssh en wag vir 'n paar minute om weer aan te meld.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Wanneer `Welcome to Ubuntu 22.04.1 LTS` verskyn, is die inisialisering voltooi, en jy kan voortgaan met die volgende stappe.

### [Opsioneel] Inisialiseer die ontwikkelingsomgewing

Hierdie stap is opsioneel.

Gerieflik plaas ek die installasie en stelselkonfigurasie van ubuntu-sagteware in [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Voer die volgende opdrag uit om met een klik te installeer.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Chinese gebruikers, gebruik asseblief eerder die volgende opdrag, en die taal, tydsone, ens. sal outomaties ingestel word.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo maak IPV6 moontlik

Aktiveer IPV6 sodat SMTP ook e-posse met IPV6-adresse kan stuur.

wysig `/etc/sysctl.conf`

Verander of voeg die volgende reëls by

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Volg op met [die kontak-tutoriaal: Voeg IPv6-verbinding by jou bediener](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Wysig `/etc/netplan/01-netcfg.yaml` , voeg 'n paar reëls by soos in die figuur hieronder getoon (Contabo VPS verstek konfigurasielêer het reeds hierdie lyne, maak net kommentaar daarop).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

`netplan apply` dan toe om die gewysigde konfigurasie in werking te laat tree.

Nadat die konfigurasie suksesvol is, kan jy `curl 6.ipw.cn` gebruik om die ipv6-adres van jou eksterne netwerk te sien.

## Kloon die konfigurasiebewaarplek ops

```
git clone https://github.com/wactax/ops.soft.git
```

## Genereer 'n gratis SSL-sertifikaat vir jou domeinnaam

Die stuur van pos vereis 'n SSL-sertifikaat vir enkripsie en ondertekening.

Ons gebruik [acme.sh](https://github.com/acmesh-official/acme.sh) om sertifikate te genereer.

acme.sh is 'n oopbron outomatiese sertifikaatondertekeninginstrument,

Voer die konfigurasiepakhuis ops.soft in, hardloop `./ssl.sh` , en 'n `conf` lêergids sal in **die boonste gids** geskep word.

Vind jou DNS-verskaffer vanaf [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , wysig `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Begin dan `./ssl.sh 123.com` om `123.com` en `*.123.com` -sertifikate vir jou domeinnaam te genereer.

Die eerste lopie sal outomaties [acme.sh](https://github.com/acmesh-official/acme.sh) installeer en 'n geskeduleerde taak vir outomatiese hernuwing byvoeg. Jy kan sien `crontab -l` , daar is so 'n lyn soos volg.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Die pad vir die gegenereerde sertifikaat is iets soos `/mnt/www/.acme.sh/123.com_ecc。`

Sertifikaathernuwing sal `conf/reload/123.com.sh` script noem, wysig hierdie script, jy kan opdragte soos `nginx -s reload` byvoeg om die sertifikaatkas van verwante toepassings te verfris.

## Bou SMTP-bediener met chasquid

[chasquid](https://github.com/albertito/chasquid) is 'n oopbron SMTP-bediener wat in Go-taal geskryf is.

As 'n plaasvervanger vir die ou posbedienerprogramme soos Postfix en Sendmail, is chasquid eenvoudiger en makliker om te gebruik, en dit is ook makliker vir sekondêre ontwikkeling.

Run `./chasquid/init.sh 123.com` sal outomaties met een klik geïnstalleer word (vervang 123.com met jou stuurdomeinnaam).

## Stel e-poshandtekening DKIM op

DKIM word gebruik om e-pos handtekeninge te stuur om te verhoed dat briewe as strooipos hanteer word.

Nadat die opdrag suksesvol uitgevoer is, sal u gevra word om die DKIM-rekord op te stel (soos hieronder getoon).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Voeg net 'n TXT-rekord by jou DNS (soos hieronder getoon).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Bekyk diensstatus en logs

 `systemctl status chasquid` Bekyk diensstatus.

Die toestand van normale werking is soos getoon in die figuur hieronder

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` of `journalctl -xeu chasquid` kan die foutlogboek sien.

## Omgekeerde domeinnaamkonfigurasie

Die omgekeerde domeinnaam is om toe te laat dat die IP-adres na die ooreenstemmende domeinnaam opgelos word.

Die opstel van 'n omgekeerde domeinnaam kan voorkom dat e-posse as strooipos geïdentifiseer word.

Wanneer die pos ontvang word, sal die ontvangende bediener omgekeerde domeinnaam-analise op die IP-adres van die stuurbediener uitvoer om te bevestig of die stuurbediener 'n geldige omgekeerde domeinnaam het.

As die stuurbediener nie 'n omgekeerde domeinnaam het nie of as die omgekeerde domeinnaam nie ooreenstem met die IP-adres van die stuurbediener nie, kan die ontvangende bediener die e-pos as strooipos herken of dit verwerp.

Besoek [https://my.contabo.com/rdns](https://my.contabo.com/rdns) en stel in soos hieronder getoon

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Nadat u die omgekeerde domeinnaam gestel het, onthou om die voorwaartse resolusie van die domeinnaam ipv4 en ipv6 na die bediener op te stel.

## Wysig die gasheernaam van chasquid.conf

Verander `conf/chasquid/chasquid.conf` na die waarde van die omgekeerde domeinnaam.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Begin dan `systemctl restart chasquid` om die diens te herbegin.

## Rugsteun conf na git-bewaarplek

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Ek rugsteun byvoorbeeld die conf-lêergids na my eie github-proses soos volg

Skep eers 'n privaat pakhuis

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Voer die conf-gids in en dien dit in by die pakhuis

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Voeg sender by

hardloop

```
chasquid-util user-add i@wac.tax
```

Kan sender byvoeg

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Verifieer dat die wagwoord korrek ingestel is

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Nadat die gebruiker bygevoeg is, sal `chasquid/domains/wac.tax/users` opgedateer word, onthou om dit by die pakhuis in te dien.

## DNS voeg SPF-rekord by

SPF (Sender Policy Framework) is 'n e-posverifikasietegnologie wat gebruik word om e-posbedrog te voorkom.

Dit verifieer die identiteit van 'n possender deur te kontroleer dat die sender se IP-adres ooreenstem met die DNS-rekords van die domeinnaam wat dit beweer om te wees, wat verhoed dat bedrieërs vals e-posse stuur.

Deur SPF-rekords by te voeg, kan dit soveel as moontlik voorkom dat e-posse as strooipos geïdentifiseer word.

As jou domeinnaambediener nie SPF-tipe ondersteun nie, voeg net TXT-tipe rekord by.

Byvoorbeeld, die SPF van `wac.tax` is soos volg

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF vir `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Let daarop dat ek `include:_spf.google.com` hier het, dit is omdat ek later `i@wac.tax` as die stuuradres in die Google-posbus sal opstel.

## DNS-konfigurasie DMARC

DMARC is die afkorting van (Domain-based Message Authentication, Reporting & Conformance).

Dit word gebruik om SPF-bons op te vang (miskien veroorsaak deur konfigurasiefoute, of iemand anders maak asof hy jy is om strooipos te stuur).

Voeg TXT-rekord `_dmarc` ,

Die inhoud is soos volg

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Die betekenis van elke parameter is soos volg

### p (Beleid)

Dui aan hoe om e-posse te hanteer wat SPF (Sender Policy Framework) of DKIM (DomainKeys Identified Mail) verifikasie misluk. Die p-parameter kan op een van drie waardes gestel word:

* geen: Geen aksie word geneem nie, slegs die verifikasieresultaat word deur die e-posverslagmeganisme aan die sender teruggevoer.
* Kwarantyn: Plaas die pos wat nie die verifikasie geslaag het nie in die strooiposlêer, maar sal nie die pos direk verwerp nie.
* verwerp: Verwerp direk e-posse wat nie verifikasie nie.

### fo (mislukkingsopsies)

Spesifiseer die hoeveelheid inligting wat deur die verslagdoeningsmeganisme teruggestuur word. Dit kan op een van die volgende waardes gestel word:

* 0: Rapporteer valideringsresultate vir alle boodskappe
* 1: Rapporteer slegs boodskappe wat verifikasie misluk
* d: Rapporteer slegs domeinnaam verifikasie mislukkings
* s: rapporteer slegs SPF-verifikasiemislukkings
* l: Rapporteer slegs DKIM-verifikasiemislukkings

### rua & ruf

* rua (Reporting URI for Aggregated Reports): E-posadres vir die ontvangs van saamgestelde verslae
* ruf (Reporting URI for Forensic reports): e-posadres om gedetailleerde verslae te ontvang

## Voeg MX-rekords by om e-posse na Google Mail aan te stuur

Omdat ek nie 'n gratis korporatiewe posbus kon vind wat universele adresse ondersteun nie (Catch-All, kan enige e-posse ontvang wat na hierdie domeinnaam gestuur word, sonder beperkings op voorvoegsels), het ek chasquid gebruik om alle e-posse na my Gmail-posbus aan te stuur.

**As jy jou eie betaalde besigheidsposbus het, moet asseblief nie die MX wysig nie en slaan hierdie stap oor.**

Wysig `conf/chasquid/domains/wac.tax/aliases` , stel aanstuurposbus in

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` dui alle e-posse aan, `i` is die e-posadresvoorvoegsel van die stuurgebruiker wat hierbo geskep is. Om e-pos aan te stuur, moet elke gebruiker 'n reël byvoeg.

Voeg dan die MX-rekord by (ek wys direk na die adres van die omgekeerde domeinnaam hier, soos getoon in die eerste reël in die figuur hieronder).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Nadat die opstelling voltooi is, kan jy ander e-posadresse gebruik om e-posse te stuur na `i@wac.tax` en `any123@wac.tax` om te sien of jy e-posse in Gmail kan ontvang.

Indien nie, gaan die chasquid-logboek na ( `grep chasquid /var/log/syslog` ).

## Stuur 'n e-pos na i@wac.tax met Google Mail

Nadat Google Mail die pos ontvang het, het ek natuurlik gehoop om te antwoord met `i@wac.tax` in plaas van i.wac.tax@gmail.com.

Besoek [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) en klik "Voeg nog 'n e-posadres by".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Voer dan die verifikasiekode in wat ontvang is deur die e-pos waarna aangestuur is.

Laastens kan dit as die versteksenderadres gestel word (saam met die opsie om met dieselfde adres te antwoord).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Op hierdie manier het ons die vestiging van die SMTP-posbediener voltooi en gebruik terselfdertyd Google Mail om e-posse te stuur en te ontvang.

## Stuur 'n toets-e-pos om te kyk of die konfigurasie suksesvol is

Voer `ops/chasquid` in

Begin `direnv allow` om afhanklikhede te installeer (direnv is in die vorige eensleutel-inisialiseringsproses geïnstalleer en 'n haak is by die dop gevoeg)

hardloop dan

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Die betekenis van die parameters is soos volg

* gebruiker: SMTP-gebruikersnaam
* slaag: SMTP wagwoord
* aan: ontvanger

Jy kan 'n toets-e-pos stuur.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Dit word aanbeveel om Gmail te gebruik om toets-e-posse te ontvang om te kyk of die konfigurasies suksesvol is.

### TLS standaard enkripsie

Soos in die onderstaande figuur getoon, is daar hierdie klein slot, wat beteken dat die SSL-sertifikaat suksesvol geaktiveer is.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Klik dan "Wys oorspronklike e-pos"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Soos in die figuur hieronder gewys, vertoon die oorspronklike Gmail-posbladsy DKIM, wat beteken dat die DKIM-konfigurasie suksesvol is.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Gaan die Ontvangs in die kop van die oorspronklike e-pos na, en jy kan sien dat die senderadres IPV6 is, wat beteken dat IPV6 ook suksesvol gekonfigureer is.
