+++
date = "2016-05-04T09:31:00+02:00"
menu = "git"
module = "git"
title = "GIT"

lastmod = "2016-08-18T12:54:33+02:00"
+++

Git är ett revisionshanteringssystem, främst tänkt att användas för
kod. Det innebär att det är ett verktyg för att samarbeta kring kod,
spåra ändringar -- och historik. Systemet utvecklades från början av
Linus Torvalds till Linuxkärnans källkod, men är idag ett av de
vanligaste verktygen för att samarbeta kring kod både inom fri
programvaruvärlden och industrin. I princip är Git alltså en bättre
version av att hela tiden maila nya versioner av ett program man arbetar
på till hela gruppen (eller lägga koden i Dropbox), och räkna upp en
siffra för varje ny version.

I princip arbetar Git med ett trädliknande modell. Varje textfil delas
upp i ett antal mindre block, och ändringar till dessa block lagras. Det
betyder att ändringar kan spolas både framåt och bakåt, jämföras och
slås ihop. Vi kommer här att visa några exempel på hur man kan arbeta
med Git för att samarbeta i ett kodprojekt liknande de som ni kommer att
arbeta i under er utbildning.

## Innehåll


+ [Viktiga principer](#viktiga-principer)
+ [En kort kom-igång-guide](#en-kort-kom-igång-guide)
	+ [Hämta kod med git clone](#hämta-kod-med-git-clone)
		+ [Klona till specifik mapp](#klona-till-specifik-mapp)
	+ [Visa historik med git log](#visa-historik-med-git-log)
	+ [Jämföra ändringar med git diff](#jämföra-ändringar-med-git-diff)
	+ [Starta en egen repository med git init](#starta-en-egen-repository-med-git-init)
	+ [Lägg till filer med git add](#lägg-till-filer-med-git-add)
	+ [Registrera ändringar med git stage och git commit](#registrera-ändringar-med-git-stage-och-git-commit)
	+ [Ångra sig med git reset](#ångra-sig-med-git-reset)
	+ [Skapa- och byta mellan branches](#skapa-och-byta-mellan-branches)
	+ [Merge-konflikter](#merge-konflikter)
		+ [Ångra en merge](#ångra-en-merge)
+ [Arbeta mot fjärr-repositories](#arbeta-mot-fjärr-repositories)
	+ [Om repositoryt inte redan är klonat](#om-repositoryt-inte-redan-är-klonat)
	+ [Om repositoryt redan klonats](#om-repositoryt-redan-klonats)
	+ [Klonade repositories](#klonade-repositories)
	+ [Ladda upp lokalt repository till fjärr-repository](#ladda-upp-lokalt-repository-till-fjärr-repository)
	+ [Vad är `origin`?](#vad-är-origin)
	+ [`Push` - ladda upp commits till fjärr-repository](#push-ladda-upp-commits-till-fjärr-repository)
	+ [`Pull` - ladda hem commits från fjärr-repository](#pull-ladda-hem-commits-från-fjärr-repository)
+ [GitHub](#github)
+ [Hur man brukar arbeta med Git i grupp](#hur-man-brukar-arbeta-med-git-i-grupp)
+ [Git på din egen dator](#git-på-din-egen-dator)
+ [Avancerade funktioner att lära sig på egen hand](#avancerade-funktioner-att-lära-sig-på-egen-hand)
	+ [Travis och annan automatisering](#travis-och-annan-automatisering)
+ [Läs mer](#läs-mer)

## Viktiga principer

En "git-repository" är en samling av filer och mappar som Git håller
koll på- och hanterar ändringar i. Varje git-repository har en egen
mapp, som kan heta t.ex. `ospp-projekt-grupp-1` eller något liknande,
egentligen vad som helst. Varje fil i ett git-repository kan vara i ett av
tre tillstånd:

1. Inte incheckad
2. Stage:ad
3. Committad

En fil (eller mer precis: en _ändring_ på en fil) rör sig från 1 till 3
i listan. Först _stage_:as ändringen, vilket innebär att tiden fryses
och filen sparas så som den såg ut. När alla ändringar som hör ihop är
stage:ade _committar_ man dem i klump, vilket betyder att man buntar
ihop dem tillsammans med ett meddelande om vad för ändringar som har
gjorts.

När en ändring väl committats sparas den för evigt i repositoryts
historia. Även om ändringarna (committen) skulle ångras och rullas
tillbaka går det att gå tillbaka till just den versionen av repositoryts
historia och se hur saker såg ut då. Detta är mycket användbart vid
t.ex. avlusning (debugging) av kod, när man ska ta reda på hur en
bugg introducerats. Det är också mycket svårt att på riktigt råka ta
bort något från ett Git-repository när det väl committats.

Från början är _alla_ filer inte incheckade. Det betyder att Git inte
bryr sig om ändringar som görs på filerna alls. Det första steget när
man skapar en ny fil i ett git-repository är därför att lägga till och
stage:a den, så att Git vet att den finns.

Arbetsordningen med Git är alltså:

1. Redigera (eller skapa, ta bort, byt namn på) filer
2. Stage:a ändringarna för att frysa dem i tiden
3. Committa dem med ett meddelande som beskriver vad ändringarna gör

{{< figure src="/images/git/git-workflow.gif" title="Snabb inmatning: man kan stage:a och committa med ett enda kommando" >}}

Git sparar alla ändringarna i en tidslinje som det går att spola framåt
och bakåt i. Tillståndet som ett repository för tillfället står på kallas
för _HEAD_ (som i "läshuvud"). Men det går också att gå i sidled och ha
alternativa tidslinjer (precis som i science fiction). Det kallas för
_branches_ eftersom de grenar ut historiken i ett träd. Vi kommer att gå
igenom hur man kan använda branches för att samarbeta i en grupp med ett
kodprojekt senare. Egentligen görs i princip ingen skillnad mellan den
första grenen (som brukar kallas `master`) och andra branches -- det går
t.o.m. att ta bort `master` om man skulle vilja!

Det som gör Git användbart vid samarbete med andra är framför allt
möjligheten att synkronisera repositories med sina branches via servrar,
s.k. _remotes_. En remote sparar den senaste gemensamma versionen av
projektets historia så att alla kan hämta hem andras ändringar -- men
också skicka upp sina egna. Ett inbyggt låssystem ser till att det är
omöjligt att skicka upp sina ändringar utan att först ha hämtat de
senaste gemensamma ändringarna från remoten. På så sätt kan man
säkerställa att alla i projektet är överens om hur den gemensamma koden
ser ut, även om man arbetar oberoende av varandra.

Det går också att slå ihop branches till en gemensam tidslinje igen,
vilket kallas för en _merge_ (lämpligt nog). Det betyder att Git gör
sitt bästa för att passa ihop ändringarna som gjorts sedan den
gemensamma brytpunkten för två branches. Ofta kan detta göras automatiskt,
men ibland uppstår en konflikt och användaren måste hjälpa till. I
praktiken görs också en merge när kod hämtas från en remote. Det innebär
att om en gruppmedlem har hunnit göra ändringar som du inte har sett så
kommer Git att göra sitt bästa för att pussla ihop projektet ändå åt er
(och åtminstone varna om det inte går).


## En kort kom-igång-guide

Det normala arbetsflödet med git är helt kommandoradsbaserat och utgår
från programmet `git`. Många editorer har dock stöd för att interagera
med Git direkt.

### Hämta kod med git clone

Man börjar använda git på ett av två sätt. Antingen hämtar man hem kod
som redan finns genom kommandot `git clone` ("clone" eftersom man
"klonar" en kopia av koden och dess historik till sin dator):

{{< figure src="/images/git/anim/clone.gif" class="medium" >}}

```none
$ git clone https://github.com/rg3/youtube-dl
Cloning into 'youtube-dl'...
remote: Counting objects: 70370, done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 70370 (delta 8), reused 0 (delta 0), pack-reused 70349
Receiving objects: 100% (70370/70370), 38.71 MiB | 3.28 MiB/s, done.
Resolving deltas: 100% (51112/51112), done.
Checking connectivity... done.
```

Här användes adressen till programmet `youtube-dl`:s källkod, men vilken
adress som helst fungerar **så länge den pekar till ett git-repository**.

#### Klona till specifik mapp
Om man inte anger en mapp efter adressen kommer git att skapa en mapp där
du står som heter samma sak som repositoryt (oftast slutet på adressen),
i det här fallet `youtube-dl`. Om du t.ex. skulle vilja klona `youtube-dl`
till en mapp kallad *min-kopia-av-ytdl* skulle kommandot se ut så här:
```none
$ git clone https://github.com/rg3/youtube-dl min-kopia-av-ytdl
```

### Visa historik med git log

Nu kan vi ställa oss i mappen som git skapade med `cd youtube-dl` som
vanligt och börja köra kommandon. Om man t.ex. kör `git log` får man en
historik över alla commits med nyast överst, liknande den här:

{{< figure src="/images/git/anim/log.gif" class="medium" >}}

```none
commit 1094074c045140e9a91b521b0a933f394a7bba91
Author: Remita Amine <MAILADRESS BORTTAGEN>
Date:   Thu Aug 4 09:38:37 2016 +0100

    [kaltura] extract subtitles and reduce requests

commit 217d5ae0137943829db23d13eee425e5fd7c08ae
Author: Remita Amine <MAILADRESS BORTTAGEN>
Date:   Thu Aug 4 09:37:27 2016 +0100

    [vodplatform] Add new extractor
```

Här kan vi alltså se två commits. Varje commit har en unik nyckel
(ex. `1094074c045140e9a91b521b0a933f394a7bba91`) som identifierar
commiten, vem som checkat in den (*Author*), när den checkades in (*Date*),
och den kommentar som lämnades vid incheckning. Notera att båda
ändringarna är sammanhängande funktioner som lagts till programmet.

Avsluta med `q` och bläddra upp och ned med piltangenterna.

### Jämföra ändringar med git diff

Om man vill veta vad som ändrades mellan de båda commitsen kan man
använda `git diff` och deras namn. Om man inte orkar kopiera hela deras
unika namn räcker det nästan alltid med de första tecknen i dem, så här:
`git diff 217d5a 1094074`.

{{< figure src="/images/git/diff.png" title="git diff mellan två commits" class="medium" >}}

Här ser vi alltså rader som tagits bort (minus) och lagts till
(plus). På många system är också raderna färgade i rött och grönt som
ovan. Notera att ordningen på argumenten spelar roll, eftersom det är
skillnaden mellan att jämföra framåt och bakåt!

Avsluta igen med `q` och scrolla upp och ner med piltangenterna.

### Starta en egen repository med git init

För att starta en tom git-repository kör man kommandot `git init` i
mappen man vill ha den (typiskt mappen man har sin källkod i). Om du
börjar helt från början måste du alltså först skapa en mapp att ha ditt
projekt i. Om du tidigare stod i `youtube-dl`-repositoryn som vi hämtade, tänk
på att göra `cd ..` för att inte skapa din nya mapp inuti den gamla.

Vi tänker oss nu att vi sätter upp ett repository med textfiler för en
fest vi ska planera:

{{< figure src="/images/git/anim/init.gif" class="medium" >}}

```none
$ mkdir min-fest
$ cd min-fest
$ git init
Initialized empty Git repository in /Users/albin/min-fest/.git/
```

Din sökväg blir förstås annorlunda beroende på vad din användare heter
och var du gjorde den tomma mappen.

### Lägg till filer med git add

Skapa filerna `gastlista.txt` och `matar.txt`, och lägg till några rader
text i dem. Så här ser mina ut:

{{< figure src="/images/git/anim/skapa_filer.gif" class="medium" >}}

```none
$ cat matar.txt
1. Sallad
2. Grillad tofu med potatis
3. Glass

$ cat gastlista.txt
- Oppfinnarjocke
- Kalle Anka
- Störiga mostern
```

Vi kan fråga Git vad den tycker om livet med `git status`:

{{< figure src="/images/git/anim/status.gif" class="medium" >}}

``` none
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	gastlista.txt
	matar.txt

nothing added to commit but untracked files present (use "git add" to track)
```

Här får vi massor av information. Git berättar att vi är på branchen
master (som skapades automatiskt och just nu är den enda branchen vi
har), att det inte finns någon tidigare historik loggad ("Initial
commit"), och att det finns två oincheckade filer som git inte har koll
på; `gastlista.txt` och `matar.txt`. Sist av allt får vi veta att det
inte finns något stage:at för att committa.

För att lägga till våra nya filer gör vi som Git föreslår:

{{< figure src="/images/git/anim/add.gif" class="medium" >}}

``` none
$ git add gastlista.txt matar.txt
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   gastlista.txt
	new file:   matar.txt
```

Nu förklarar Git för oss att det _finns_ ändringar som är stage:ade för
att committas, och vilka ändringar som har gjorts (två nya filer har
registrerats).

Om vi bara skriver `git commit` så kommer git att öppna vår inställda editor för att skriva ett meddelande. Men eftersom den kan vara lite vad som helst på universitetets datorer föreslår vi att du skickar med ett meddelande direkt på kommandoraden istället:

{{< figure src="/images/git/anim/commit.gif" class="medium" >}}

``` none
$ git commit --message "Initial commit"
[master (root-commit) 74c2b5b] Initial commit
 2 files changed, 6 insertions(+)
 create mode 100644 gastlista.txt
 create mode 100644 matar.txt

```

Git berättar för oss vilka ändringar den sparar. Om vi nu tittar i
historiken och `git status` så ser vi att ändringarna är sparade:

``` none
$ git status
On branch master
nothing to commit, working directory clean

$ git log
commit 74c2b5bedb58e818b6c550b646ff29d13bc5950c
Author: Albin Stjerna <MAILADRESS BORTTAGEN>
Date:   Thu Aug 4 14:00:28 2016 +0200

    Initial commit

```

### Registrera ändringar med git stage och git commit

Låt oss säga att vi lägger till någon i gästlistan:

{{< figure src="/images/git/anim/modify.gif" class="medium" >}}

*`echo` är ett kommando som skriver ut texten vi ger, i detta fall "- Joakim von Anka", till
terminalen. På så vis kan vi utnyttja {{< extlink link="/terminalen/#använda-filer" title="piping till filer" >}} för att enkelt
skriva till en fil.*

``` none
$ echo "- Joakim von Anka" >> gastlista.txt
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   gastlista.txt

no changes added to commit (use "git add" and/or "git commit -a")

$ git diff
diff --git a/gastlista.txt b/gastlista.txt
index 4570554..84680c2 100644
--- a/gastlista.txt
+++ b/gastlista.txt
@@ -1,3 +1,4 @@
 - Oppfinnarjocke
 - Kalle Anka
 - Störiga mostern
+- Joakim von Anka
```

Nu kan vi notera att `git diff` utan argument jämför de ändringar som
för tillfället finns med de senaste registrerade och committade
ändringarna. Här ser vi med andra ord att vi längst ner i filen
`gastlista.txt` har lagt till en rad för Joakim von Anka.

För att spara de nya ändringarna använder vi `git stage`:

{{< figure src="/images/git/anim/modify_commit.gif" class="medium" >}}

``` none
$ git stage gastlista.txt
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   gastlista.txt
```

Nu kan vi committa ändringarna precis som när vi lade till filer:

``` none
$ git commit --message "Lade till en besökare till"
[master 57673a6] Lade till en besökare till
 1 file changed, 1 insertion(+)

$ git status
On branch master
nothing to commit, working directory clean

$ git log
commit 57673a6759f1bb395731d2778cab3b1024b083c7
Author: Albin Stjerna <MAILADRESS BORTTAGEN>
Date:   Thu Aug 4 14:15:22 2016 +0200

    Lade till en besökare till

commit 74c2b5bedb58e818b6c550b646ff29d13bc5950c
Author: Albin Stjerna <MAILADRESS BORTTAGEN>
Date:   Thu Aug 4 14:00:28 2016 +0200

    Initial commit

```

### Ångra sig med git reset

Låt oss säga att vi råkade ta bort gästlistan av misstag:
``` none
$ rm gastlista.txt

$ git status
On branch master
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	deleted:    gastlista.txt

no changes added to commit (use "git add" and/or "git commit -a")
```

Då kan vi återställa hela repositoryt som den såg ut vid den senaste
committen (alltså `HEAD`, om du minns från tidigare):

{{< figure src="/images/git/anim/reset.gif" class="medium" >}}

``` none
$ git reset --hard HEAD
HEAD is now at 57673a6 Lade till en besökare till

$ git status
On branch master
nothing to commit, working directory clean

$ cat gastlista.txt
- Oppfinnarjocke
- Kalle Anka
- Störiga mostern
- Joakim von Anka
```

Det fungerar även om man har redigerat filer och vill återställa
ändringarna, men det är också farligt eftersom det **kastar bort alla
ocommitade ändringar**!

### Skapa- och byta mellan branches

Vi kan visa vilken branch vi för tillfället är på med hjälp av `git
branch`:

{{< figure src="/images/git/branch.png" class="medium" >}}

``` none
$ git branch
* master
```

Kanske inte så spännande -- det finns en branch och den är vi på. Vi kan
skapa en ny med hjälp av `git checkout`:

{{< figure src="/images/git/anim/branch_create.gif" class="medium" >}}

``` none
$ git checkout -b better-desserts
Switched to a new branch 'better-desserts'

$ git branch
* better-desserts
  master
```

Nu kan vi lägga till lite fler efterrätter (bara en är trots allt lite snålt!):

{{< figure src="/images/git/anim/branch_edit.gif" class="medium" >}}

``` none
$ din-favorit-editor matar.txt
$ git diff
diff --git a/matar.txt b/matar.txt
index 38e1bc6..3711a8a 100644
--- a/matar.txt
+++ b/matar.txt
@@ -1,3 +1,6 @@
 1. Sallad
 2. Grillad tofu med potatis
 3. Glass
+4. Crème brûlée
+5. Mer glass
+6. Kladdkaka

$ git stage matar.txt
$ git commit --message "Mer efterrätt!"
[better-desserts 29e50b5] Mer efterrätt
 1 file changed, 3 insertions(+)
```

Låt oss säga att vi är klara med de här ändringarna nu och vill ha
tillbaka dem till `master`. Då måste vi först byta tillbaka till
`master` från `better-desserts` och göra en `merge` mellan dem:

{{< figure src="/images/git/anim/branch_merge.gif" class="medium" >}}

``` none
$ git checkout master
Switched to branch 'master'

$ git merge better-desserts
Updating 57673a6..29e50b5
Fast-forward
 matar.txt | 3 +++
 1 file changed, 3 insertions(+)

$ git status
On branch master
nothing to commit, working directory clean
```

Eftersom skillnaden mellan brancharna var trivial kunde Git räkna ut hur
de skulle slås ihop.

### Merge-konflikter

Merge-konflikter är något som du troligtvis kommer stöta på en hel del.
Till en början kan de verka skrämmande och komplicerade, men har man väl
förstått hur dessa fungerar kan man lösa de flesta konflikter utan större
svårigheter.

Låt oss simulera en lite mer komplicerad interaktion som kan uppstå när
man samarbetar på samma kod. Ofta märker man av det när man arbetar mot
fjärr-repositories med flera personer och använder `git pull` för att dra
hem de senaste ändringarna (mer om detta i sektionen [Arbeta mot fjärr-repositories](#arbeta-mot-fjärr-repositories)).

Här fortsätter vi dock med våra två branchar:

{{< figure src="/images/git/anim/mergeconflict_setup.gif" class="medium" >}}

``` none
$ din-favorit-editor matar.txt
$ git diff
diff --git a/matar.txt b/matar.txt
index 3711a8a..d6a2168 100644
--- a/matar.txt
+++ b/matar.txt
@@ -4,3 +4,4 @@
 4. Crème brûlée
 5. Mer glass
 6. Kladdkaka
+7. Sesamkakor

$ git commit -a --message "Ännu mer efterrätt!"
[master 6b5c412] Ännu mer efterrätt
 1 file changed, 1 insertion(+)

$ git checkout better-desserts
Switched to branch 'better-desserts'

$ cat matar.txt
1. Sallad
2. Grillad tofu med potatis
3. Glass
4. Crème brûlée
5. Mer glass
6. Kladdkaka

$ din-favorit-editor matar.txt
$ git diff
diff --git a/matar.txt b/matar.txt
index 3711a8a..7d33dc4 100644
--- a/matar.txt
+++ b/matar.txt
@@ -4,3 +4,4 @@
 4. Crème brûlée
 5. Mer glass
 6. Kladdkaka
+7. Honungspudding

$ git commit -a --message "OM NOM NOM"
[better-desserts 039a4dc] OM NOM NOM
 1 file changed, 1 insertion(+)
```

Nu har vi alltså lagt till en ny rad på samma ställe i båda brancharna,
oberoende av varandra. Det betyder att vi kommer att stöta på problem
när vi ska slå ihop dem senare:

``` none
$ git checkout master
Switched to branch 'master'

$ git merge better-desserts
Auto-merging matar.txt
CONFLICT (content): Merge conflict in matar.txt
Automatic merge failed; fix conflicts and then commit the result.
```

Git berättar för oss att ändringarna i `matar.txt` är inkompatibla och
måste slås ihop för hand. Om vi öppnar den i valfri editor ser vi det
här:

``` none
1. Sallad
2. Grillad tofu med potatis
3. Glass
4. Crème brûlée
5. Mer glass
6. Kladdkaka
<<<<<<< HEAD
7. Sesamkakor
=======
7. Honungspudding
>>>>>>> better-desserts
```

Vilket ska läsas som "på den här raden i HEAD finns "7. Sesamkakor"
medan det i better-desserts finns "7. Honungspudding". Vi får helt
enkelt öppna filen i vår editor och ändra de berörda raderna till något
rimligt (och ta bort Gits tillägg). Sen gör vi som Git föreslår och
committar den lagade versionen:
``` none
$ din-favorit-editor matar.txt
$ git diff
diff --cc matar.txt
index d6a2168,7d33dc4..0000000
--- a/matar.txt
+++ b/matar.txt
@@@ -4,4 -4,4 +4,5 @@@
  4. Crème brûlée
  5. Mer glass
  6. Kladdkaka
 -7. Honungspudding
 +7. Sesamkakor
++8. Honungspudding

$ git commit -a --message "Fix merge conflict"
[master 0fc4b75] Fix merge conflict
```

{{< figure src="/images/git/anim/mergeconflict_fix.gif" class="medium" title="Löser en merge-konflikt" >}}


#### Ångra en merge
Om allt blev jättjobbigt och vi ångrar oss finns alltid möjligheten att
köra `git merge --abort` för att återställa tillståndet som det var
innan vi försökte oss på en merge.

{{< figure src="/images/git/anim/mergeconflict_abort.gif" class="medium" >}}

## Arbeta mot fjärr-repositories
En stor fördel med Git, utöver vad vi redan gått genom, är att man kan
skapa s.k. fjärr-repositories (remote repository) mot vilka flera
personer kan arbeta --- på så vis är det väldigt enkelt att arbeta tillsammans
i grupp på ett projekt. Ett fjärr-repository kan ses som en server där man
lagrar sit repository, från vilken man hämtar hem filer/ändringar till sin dator.
Generellt sett följer arbetsprocessen med fjärr-repositories följande:

#### Om repositoryt inte redan är klonat
1. Klona ett repository till datorn (lokalt)
2. Arbeta med ändringar lokalt
3. Commita ändringar lokalt
4. Ladda upp commits till fjärr-repository via `push`

#### Om repositoryt redan klonats
1. Hämta commits/ändringar från fjärr via `pull`
2. Arbeta med ändringar lokalt
3. Commita ändringar lokalt
4. Ladda upp commits till fjärr-repository via `push`

Det finns ett flertal verktyg för att skapa- och hantera fjärr-repositories,
exempel är [GitHub](#github) som tas upp senare i modulen. Fjärr-repositories
är med andra ord inget man kan skapa via kommandon i git.

### Klonade repositories
Om man klonar ett repository följer det automatiskt med vilket fjärr-repository
repositoryt tillhör. I och med detta är det enklaste sättet att få ett repository som
är förinställt med fjärr-möjlighet att skapa ett repository via valfri tjänst
(ex. [GitHub](#github)) för att sedan klona det.

### Ladda upp lokalt repository till fjärr-repository
Om du har ett lokalt repository du vill ladda upp till ett fjärr-repository
måste fjärr-repositoryt kopplas till ditt lokala repository. **Först måste du
se till att du skapat ett nytt fjärr-repository via valfri tjänst** och därefter kan
du koppla ditt lokala repository till fjärr-repositoryt via:

```none
$ git remote-add origin <URL TILL FJÄRR-REPOSITORY BORTTAGEN>
```

Detta ställer in fjärr-repository till det repository du angav URL till.

### Vad är `origin`?
`origin` är standardnamnet som används för att koppla fjärr-repository. Tekniskt sett
skulle man kunna döpa fjärr-repositoryn till andra namn, men det är inte rekommenderat
då origin är "industristandarden".

### `Push` - ladda upp commits till fjärr-repository
`git push` är det kommando som används för att ladda upp commits/ändringar som gjorts-
och sparats lokalt. Kommandot ser ut som följer:
```none
$ git push origin <branch-name>
```
där *branch-name* är namnet på den branch vi vill ladda upp ändringar för. Om
vi arbetar i branchen *master* och vill ladda upp ändringar för denna skriver
vi:
```none
$ git push origin master

Counting objects: 3, done.
Writing objects: 100% (3/3), 280 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To <URL TILL FJÄRR-REPOSITORY BORTTAGEN>
   44aab50..e1a086f  master -> master
```

Om vi däremot är inne i en annan branch, låt säga *min-branch* och vill ladda upp
dess ändringar till fjärr-repositoryt skriver vi:
```none
$ git push origin min-branch

Total 0 (delta 0), reused 0 (delta 0)
To <URL TILL FJÄRR-REPOSITORY BORTTAGEN>
 * [new branch]      min-branch -> min-branch
```
Om branchen *min-branch* inte redan finns i fjärr-repositoryt kommer det automatiskt
skapas och ändringarna laddas upp, som i exemplet ovan.

**För att vår branch skall komma ihåg vilken fjärr-branch den skall arbeta mot**
kan vi ställa in detta genom att lägga till `-u`-flaggan när vi pushar:
```none
$ git push -u origin min-branch

Branch min-branch set up to track remote branch min-branch from origin.
Everything up-to-date
```

därefter behöver vi ej heller skriva vilket fjärr-repository och branch vi vill push:a
till, eftersom vi ställt in det för vår branch:

```none
$ git push

Counting objects: 3, done.
Writing objects: 100% (3/3), 280 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To <URL TILL FJÄRR-REPOSITORY BORTTAGEN>
   44aab50..e1a086f  min-branch -> min-branch
```


### `Pull` - ladda hem commits från fjärr-repository
Om man är flera som arbetar mot samma fjärr-repository eller om man använder fler än
en dator behöver man hämta ändringar som andra laddat upp. Detta gör man via
kommandot `git pull origin <branch-name>`.

```none
$ git pull origin master

Updating e1a086f..c3c96b6
Fast-forward
 enfil.txt | 3 +++
 1 file changed, 3 insertions(+)
```

Om vi är i branch _master_ **eller** har ställt in att vår branch vet vilken
fjärr-branch den skall arbeta mot behöver vi bara skriva `git pull`:
```none
$ git pull

Updating e1a086f..c3c96b6
Fast-forward
 enfil.txt | 3 +++
 1 file changed, 3 insertions(+)
```

## GitHub

{{< extlink link="https://github.com" title="GitHub">}} är en av de
största tjänsterna på webben för att samarbeta över Git. Flera kurser på
Uppsala universitet använder GitHub för att koordinera
grupparbeten. Förutom att tillhandahålla remotes för git-repositories så
har GitHub också ett antal egna funktioner. Att registrera sig är
gratis, men för att få ha privata repositories (d.v.s. sådana som inte
syns för allmänheten) måste man betala eller {{< extlink link="https://education.github.com/pack" title="registrera sig med ett studentkonto">}}. Tänk på att regler kring fusk och plagiat kan göra
det _nödvändigt_ att använda privata repositories för grupparbeten! Om
du är osäker -- fråga din lärare!

Förutom att GitHub låter dig husera din repository hos sig, visa
historik och commits för ett repository, och annat som redan finns i
kommandoradsklienten så erbjuder de också två funktioner som saknas i
kommandorads-Git: _forks_ och _pull requests_. Det finns också stöd för
att hantera kommentarer, buggrapporter, med mera, men vi kommer inte att
gå in på det här.

En fork är helt enkelt en fullständig kopia av en annan repository som
hamnar under ditt konto. De används speciellt när man vill fortsätta på
någon annans kod ostört, lite som en branch. Säg t.ex. att jag skulle
vilja lägga till stöd för att spara ner video från SVT Play till
`youtube-dl`, som vi laddade ner ovan. Då skulle jag kunna gå till
`youtube-dl`:s sida på GitHub, klicka på "fork" och få en egen kopia
under mitt egna konto som jag kan arbeta med.

När jag är klar med SVT Play-ändringarna vill jag gärna ha ett sätt att
få tillbaka dem till den officiella versionen av `youtube-dl`, och det
är här pull requests kommer in. En pull request är helt enkelt ett
standardiserat sätt att snällt be någon "vänligen gör en merge
härifrån". Om en merge kan göras utan konflikter finns det dessutom en
funktion i GitHub för att göra den direkt från webben.

När man arbetar mot GitHub -- precis som vilken annan Git-server som
helst -- så används kommandona `git pull` respektive `git push` för att
hämta hem- respektive skicka upp sina commits till servern. Tänk på att
det bara är committade ändringar som skickas! Att göra en `git pull`
innan man börjar arbeta och en `git push` så fort man är klar är en bra
vana att skaffa sig!

## Hur man brukar arbeta med Git i grupp

Till att börja med så är ett viktigt råd att _bara versionshantera kod_!
Genererad dokumentation, PDF:er, tillfälliga filer skapade av din
editor, programfiler och så vidare ska _inte_ `git add`:as! Annars
kommer kommer konstiga merge-konflikter att dyka upp när alla i gruppen
har kompilerat varsitt lite olika program. En tumregel är: har en
människa skrivit filen bör den versionshanteras, annars inte (undantag:
bilder, eventuella filer ni kanske vill ha nära tillhands men
inte ändra på).

För små projekt med få personer och lite kod brukar det gå bra att bara
utveckla direkt mot `master`-branchen och synkronisera mot samma
repository på t.ex. GitHub. När projekten blir lite större (i antal
personer speciellt) börjar lite mer avancerade metoder krävas.

Ett vanligt arbetssätt i sådana situationer är feature-branches. Det
innebär att bara mer eller mindre färdig kod som är testad och fungerar
tillåts på `master`-branchen. All egentlig utveckling sker sedan i
branches, som döps till något relaterat till vad de gör
(exv. `feature/svt-play-support`). Tanken är att en enskild funktion i
programmet man tillverkar utvecklas och mognar i en branch, och att
denna branch hålls uppdaterad mot `master`, men inte andra parallellt
utvecklade funktioner. På så sätt kan flera grupper inom projektet jobba
på olika saker utan att hela tiden kliva varandra på tårna, och det
finns alltid en senast fungerande version av programmet att jämföra med
om något går sönder.

När en funktion på en branch är klar skickas sedan en Pull Request
(detta förutsätter förstås att GitHub används) mot `master` från
branchen. Ofta brukar GitHub säga till om det går skapa en pull-request
från ex. nyligen uppskickade committs eller liknande.

{{< figure src="/images/git/git-compare-and-pull-request.png"
title="GitHub är hjälpsam nog att föreslå att jag skickar en Pull Request från branchen jag precis push:ade till." >}}

Någon annan än den/de som utvecklat funktionen får då ansvaret
att gå igenom alla ändringar (som dessutom går att se med hjälp av både
GitHubs verktyg och `git diff` som vi tittade på tidigare) och försöka
hitta brister i koden. Först när alla är nöjda och övertygade om att
allt fungerar mergas branchen in i master. En ny branch skapas för nästa
funktion -- och processen börjar om.

{{< figure src="/images/git/git-create-new-pull-request.png"
title="Dialogruta på GitHub för att göra en Pull Request för att merga in den här guiden i resten av materialet." >}}


## Git på din egen dator

Du kan ladda ner Git för macOS, Windows och GNU/Linux
{{< extlink link="https://git-scm.com/downloads" title="på Gits nedladdningssida">}}. Där finns
också närmare instruktioner. De flesta GNU/Linux-distributioner har
också Git i sina pakethanterare. Om du har installerat XCode till macOS
så får du också Git på köpet.

## Avancerade funktioner att lära sig på egen hand

Nedan följer en lista på funktioner som kan vara värda att lära sig på
egen hand, men som inte tas upp här:

- `amend` -- ändra och formulera om commits. Användbart om man råkat
  skriva fel.
- `patch` -- generera patchfiler från git-ändringar
- `rebase` -- som `merge`, men skriver om historien. Gör det möjligt att
  slå ihop commits, ta bort commits helt, och att flytta om t.ex. en
  feature branch på en `master` som uppdaterats sedan den startade.
- `cherry-pick` -- välj specifika commits från en branch och lägg dem på
  en annan branch

### Travis och annan automatisering


{{< extlink link="http://travis-ci.org" title="Travis">}} är en tjänst som erbjuder automatisering
för projekt som också använder GitHub. Den är gratis att använda för
personer med studentkonton på GitHub och för Open Source-projekt (i princip:
alla med publika repositories). Bland annat kan den användas för att
automatiskt köra tester mot nya ändringar på ett projekt -- och mot Pull
Requests.

## Läs mer

- {{< extlink link="https://www.atlassian.com/git/" title="Atalassians Git-guider">}}
- {{< extlink link="https://git-scm.com/book/en/v2" title="Den officiella Git-manualen" >}}
- {{< extlink link="https://help.github.com/categories/bootcamp/" title="GitHubs officiella manual: Bootcamp (kom-igång-guider)" >}}
