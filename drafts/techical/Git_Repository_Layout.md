
http://git-scm.com/book/ch6-4.html
http://www.plantuml.com/plantuml/form

* Momenteel zijn er meerdere repos en users in gebruik, het doel is dit uiteindelijk uniform te verdelen tot het noodzakelijke.

* In verband met problemen met de site en de backup hiervan, is deze strategie opgesteld voor de live omgeving.

* Er zijn een aantal fail-safes ingebouwd waardoor het mogelijk is om praktisch zonder tijdverlies terug te gaan tot eerdere momenten in de boom.

* Door de werking van git en magento samen, vereist dit wel enige toelichting.


### BitBucket `wellbon`

Voor de live productie code is een account gemaakt bij BitBucket. Dit omdat er sprake is van een online productieomgeving waarin betalingen plaatsvinden en dat er configuratiebestanden zijn met gevoelige gegevens zoals wachtwoorden van de database, is het **strict noodzakelijk** een *private repository* te hebben. Aangezien de meest gebruikte aanbieder in de development wereld, GitHub, alleen open-source repositories gratis aanbiedt, en derhalve de bestanden door de hele wereld in te zien zijn, is gekozen voor Bitbucket.org (BB).

BitBucket, BB, ondersteunt namelijk wel - in tegenstelling tot GitHub (GH) - well een gratis private repository. Slechts 1 stuk maar dat is voor ons voldoende omdat de technologie van `git` het mogelijk maakt een complexe boom van samenhangende (branch) takken of deze seperaat van de boom (orphan) aan kan maken. Daarnaast zijn er nog een soort bookmarks (tags) aan te maken om snel in bomen te kunnen springen.

Voor de indeling van deze repository is gekozen gebruik te maken van het programma `git-flow` welke de strategie volgt die uitgewerkt is in het blogbericht [A successful Git branching model][gitflow] van <http://nvie.com>.

```sh
wellnessbon.nl@ssh1.c60 ~ $ git-flow init
Initialized empty Git repository in /home/users/wellvftp/.git/
No branches exist yet. Base branches must be created now.
Branch name for production releases: [master]     
Branch name for "next release" development: [develop] 

How to name your supporting branch prefixes?
Feature branches? [feature/] 
Release branches? [release/] 
Hotfix branches? [hotfix/] 
Support branches? [support/] 
Version tag prefix? [] 
```
Geef een `RETURN` toetsaanslag voor iedere standaard instelling. Mogelijkerwijs wil je een systeem van tags gebruiken, de prefix kun je dus wijzigen indien nodig.

Het `~` teken op de eerste regel geeft aan dat ik deze, per ongeluk, in de `$HOME` van onze gebruiker geplaatst heb. Aangezien dit niet gewenst is en dit een demo betreft verwijder ik snel deze lokale repository met `cd; rm -rf .git`.

> LET OP: Onze huidige shell begint te hoesten bij enige vorm van complexere prompt. Hoewel ik succesvol `zsh` gecompileerd heb en `oh-my-zsh` geinstalleerd heb, wordt toch op (Unicode?) tekens moeilijk gedaan. Ook een bash variant, die onze huidige .git repo, status en branch in de prompt toont, `bash-it`, wordt niet gevroten zonder performance impact die ongewenst is. Je ziet dus nergens in de prompt of je nu in een .git managed directory bent en welke branch dan + of deze map dirty is.

#### Byte prompt
`wellnessbon.nl@ssh1.c60 ~/wellnessbon.nl $ `

#### Development prompt
```sh
ijzervreter :: ~/well/wellnessbon.nl ‹resources/media› » touch a
ijzervreter :: ~/well/wellnessbon.nl ‹resources/media*› »
```
Merk op dat bij het toevoegen van een nieuw bestand, er een `*` asteriks bij de git branch naam geplaatst wordt. Dit vertelt mij: "er zijn bestanden die nog niet toegevoegd zijn aan versiebeheer in deze map". In dit geval weet ik dus: `git add --all . && git commit -am 'added a a'` zou deze `*` weghalen omdat hij dan in (lokaal) beheer zit.

Deze wijzigingen zijn eenvoudig naar remote door te voeren. Maar alvorens ik dit vertel, is het nodig even de structuur van verschillende machines en repositories te schetsen.

1. De repository is gestart op machine `wb` door het commando `git-flow init` te geven in de map waar met `~/wellnessbon.nl` symlink naar verwezen is. De `public/` folder meer specifiek.

1. Met het commando `ssh-keygen` is in de map `~/.ssh` een sleutel aangemaakt waarvan de publieke component, het bestand `~/.ssh/id_rsa.pub` betreft. Deze sleutel is aan de bitbucket account toegevoegd.

1. Hierdoor is het voor iedereen met toegang tot onze Byte gehoste shell, via de account `wellvftp` voorlopig mogelijk om, zonder wachtwoord op de RSA id, het commando `git push` te gebruiken om wijzigingen (ook van de MASTER) door te voeren.

1. Dit is niet echt te voorkomen anders dan door een wachtwoord erop te zetten en zelf telkens de wijzigingen naar de remote te pushen maar, dit is voorlopig niet nodig en indien dit wachtwoord van de byte-shell in verkeerde handen komt, is nog niet **all hope is lost???**.

We hebben dus de volgende constructie:

* BB repo 'Site' branches `master`, `develop`, `resources/media` enz.
* WB repo `~/wellnessbon.nl`
* DM repo lokaal Developer Machine (DM).

![Git repository chain](http://www.plantuml.com:80/plantuml/png/S_5LiD5L27TIi598pidFI-LoyLNGjOC859GMPt01MK2-4umF0000)

Uit dit diagram blijkt dat het wellicht niet noodzakelijk is om de push naar `BB` open te houden voor iedereen. In het geval van meerdere developers kan men er voor kiezen alleen de `WB` repo te gebruiken (NOOIT de master) en vervolgens deze goed te keuren door de verantwoordelijk(en) alvorens naar `BB` te pushen.


Vanuit development heb ik gecloond van onze WB machine en derhalve zijn changes die vanuit thuis gepushd worden dus niet direct in de remote zichtbaar. Ik neem dus bijvoorbeeld de volgende stappen:

```sh
DM :: /tmp » git clone wb:/home/users/wellvftp/wellnessbon.nl
Cloning into 'wellnessbon.nl'...
remote: Counting objects: 19637, done.
remote: Compressing objects: 100% (11793/11793), done.
remote: Total 19637 (delta 5662), reused 19556 (delta 5621)
Receiving objects: 100% (19637/19637), 125.50 MiB | 4.83 MiB/s, done.
Resolving deltas: 100% (5662/5662), done.
Checking connectivity... done.
DM :: /tmp » 
```

Je ziet in de bovenste uitvoer naar mijn `stdout` dat het een best wel grote repository is. Dit komt o.a. doordat de branch `resources/media` een flinke portie nodig heeft. Kloon in voorkomend geval alleen bijv. het strict noodzakelijke:

```sh
git clone --single-branch wb:/home/users/wellvftp/wellnessbon.nl   
Cloning into 'wellnessbon.nl'...
remote: Counting objects: 16603, done.
remote: Compressing objects: 100% (8981/8981), done.
remote: Total 16603 (delta 5575), reused 16527 (delta 5538)
Receiving objects: 100% (16603/16603), 22.05 MiB | 5.14 MiB/s, done.
Resolving deltas: 100% (5575/5575), done.
Checking connectivity... done.
```

Deze download gaat al flink sneller. 

Omdat ik een publieke sleutel tussen mijn (DM) development machine `DM` en wellnessbon.nl machine `wb` deel als volgt, in mijn `~/.ssh/config` bestand:

```sh
DM :: /tmp » cat ~/.ssh/config

# Ensure for all hosts we don't have a freezing connection by sending
# `keep alive` interval signals.
Host *
  ServerAliveCountMax 3
  ServerAliveInterval 240

# Add hosts and the keys known at their machines. Refer to secret local key
# and only provide the `*.pub` counterpart with the remote machine.
Host wb
  HostName ssh045657.bytenet.nl
  User wellnessbon.nl
```

Op de remote machine staat in het bestand `~/.ssh/known_hosts` de publieke sleutel die ik gebruik. Andere developers moeten dus voor zichzelf ook sleutels aanmaken en toevoegen.

De repository wordt dus lokaal gekloond en als geen branch opgegeven wordt standaard de `master` genomen wordt.

Laten we zeggen dat we een kopie van de master willen maken die volkomen los hangt van de main tree van gerelateerde versies die elkaar opvolgen. Hierdoor is het volkomen veilig om te rotzooien hoe dan ook:

```sh
DM :: /tmp/wellnessbon.nl ‹master› » git checkout --orphan sandbox/playground 
Switched to a new branch 'sandbox/playground'
```

We maken dus een orphan branch van de master en noemen deze `sandbox/playground` bijvoorbeeld. Ik wil deze helemaal leeg hebben en alles verwijderen maar:

> LET OP: Verwijder NOOIT de `.git` directory want dan ben je dus alles kwijt!!


Stel, je wilt een bugfix indienen dan kun je dat met de volgende oneliner doen:

```sh
cd /tmp && git clone --single-branch wb:/home/users/wellvftp/wellnessbon.nl && cd wellnessbon.nl && git checkout -b hotfix/bug12345
```

We doen onze patch toepassen bijv. `rm errfile` (;) 

```sh
git add --all . && git commit -am 'fixed issue by removing erroneous file' && git push --set-upstream origin hotfix/bug12345
[hotfix/bug12345 a050e94] fixed issue by removing erroneous file
 1 file changed, 1 insertion(+)
Counting objects: 7, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (5/5), 557 bytes | 0 bytes/s, done.
Total 5 (delta 2), reused 0 (delta 0)
To wb:/home/users/wellvftp/wellnessbon.nl
 * [new branch]      hotfix/bug12345 -> hotfix/bug12345
Branch hotfix/bug12345 set up to track remote branch hotfix/bug12345 from origin.
```

Dit klopt nog niet helemaal, omdat de master hiermee niet gefixed zou worden, maar het dient als voorbeeld om te laten zien hoe nu de repository op machine WB een nieuwe branch heeft:

```sh
wellnessbon.nl@ssh1.c60 ~/wellnessbon.nl $ git branch -l
  develop
  hotfix/bug12345 <=== DEZE IS NIEUW
* master
  refactor/rebase1401-1410
  resources/media
```

We zouden nu makkelijk onze wijzigingen door kunnen voeren. In deze demo heb ik voor het gemak een bestand gemaakt genaamd `errfile` met de inhoud `abcd`. Als ik deze nu (harmless) in de master zou willen hebben hoef ik alleen maar het volgende te doen in de `master` (NOOT: het `*` asteriks teken geeft op deze server aan dat we in die branch werken):

```sh


```



[plantuml]: <http://www.plantuml.com/plantuml/form>
[gitflow]: <http://nvie.com/posts/a-successful-git-branching-model/>






