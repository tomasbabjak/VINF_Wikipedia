# Zhlukovanie článkov Wikipédie do kategórií na základe ich vedecko-spoločenskej oblasti

##### Vypracoval: Tomáš Babjak
##### Predmet: Vyhľadávanie informácii

## Zadanie projektu

Zadaním projektu je extrahovať články anglickej wikipédie, z týchto článkov extrahovať dôležité črty a na ich základe ich zaradiť do určených vedecko-spoločenských oblastí.

Do týchto oblastí budeme zaradzovať napríklad osoby, miesta alebo aj udalosti, teda napríklad či sa daná osoba venuje literatúre, vedeckej činnosti, politike alebo športu. A následne aj podrobnejšie akému vednému oboru alebo akému druhu literatúry sa venuje. Podobne pri udalostiach môžeme určiť či bola udalosť vojnovou udalosťou, športovou alebo vedeckou. Taktiež tieto objekty môžeme zhlukovať podľa štátov alebo miest kde sa odohrávali resp. kde pôsobili.

Podmienkou na vypracovanie témy bolo použitie distribuovaného spracovania na čo sme sa rozhodli použiť knižnicu `PySpark` a použitie programovacieho jazyka `Python`.

## Dáta 

Pri práci na projekte budeme používať dump anglickej Wikipédie, ktorý môžeme nájsť tu: [Dumpy Anglickej Wikipédie](dumps.wikimedia.org/enwiki/latest) 
Konkrétne budeme pracovať so súborom `enwiki-latest-pages-articles.xml.bz2`, ktorý si rozbalíme a budeme pracovať s XML súborom o celkovej veľkosti 74,1 GB.

#### Príklad dát
Ako príklad dát slúži článok z daného XML súboru o Aristotelovi, kde na začiatku vidíme Infobox, obsahujúci základné informácie o ňom a, jeho pôsobení, dátumoch a miestach narodenia a úmrtia ako aj jeho hlavné vedné obory, ktoré použijeme aj v tejto práci. Nasleduje samotný článok, ktorý tu už nie je celý a na konci sú ešte odkazy na iné súvisiace články.
```
Infobox philosopher
 | name=Aristotle
 | caption=Roman copy in marble of a Greek bronze [[Bust (sculpture)|bust]] of Aristotle by [[Lysippos]], [[Circa|c.]] 330 BC, with modern [[alabaster]] [[mantle (clothing)|mantle]]
 | birth_date=384 BC
 | birth_place=[[Stagira (ancient city)|Stagira]], [[Chalcidian League]]
 | death_date={{nowrap|322 BC (aged 61–62)}}
 | death_place=[[Euboea]], [[Macedonia (ancient kingdom)#Empire|Macedonian Empire]]
 | predecessor=[[Plato]]
 | spouse=[[Pythias]]
 | era=[[Ancient Greek philosophy]]
 | region=[[Western philosophy]]
 | notable_students  = [[Alexander the Great]], [[Theophrastus]]
 | main_interests={{Flatlist}}
* [[Biology]]
* [[Zoology]]
* [[Physics]]
* [[Metaphysics]]
* [[Logic]]
* [[Ethics]]
* [[Rhetoric]]
* [[Aesthetics]]
* [[Music]]
* [[Poetry]]
* [[Economics]]
* [[Politics]]
{{Endflatlist}}
 | influences={{Flatlist}}
* [[Plato]]
* [[Socrates]]
* [[Heraclitus]]
* [[Parmenides]]
* [[Empedocles]]
* [[Phaleas of Chalcedon|Phaleas]]
* [[Hippodamus of Miletus|Hippodamus]]
* [[Hippias]]
{{Endflatlist}}
'''Aristotle''' ({{IPAc-en|ˈ|ær|ɪ|s|t|ɒ|t|əl}};{{sfn|Collins English Dictionary}} {{lang-grc-gre|Ἀριστοτέλης}
''Aristotélēs'', {{IPA-grc|aristotélɛːs|pron}}; 384–322&amp;nbsp;BC) was a Greek [[philosopher]] and [[polymath]] during the [[Classical Greece|Classical period]] in [[Ancient Greece]]. Taught by [[Plato]], he was the founder of the [[Lyceum (Classical)|Lyceum]], the [[Peripatetic school]] of philosophy, and the [[Aristotelianism|Aristotelian]] tradition. His writings cover many subjects including [[Physics (Aristotle)|physics]], [[biology]], [[zoology]], [[metaphysics]], [[logic]], [[ethics]], [[aesthetics]], [[Poetics (Aristotle)|poetry]], [[theatre]], [[music]], [[rhetoric]], [[psychology]], [[linguistics]], [[economics]], [[politics]], and government. Aristotle provided a complex synthesis of the various philosophies existing prior to him. It was above all from his teachings that the West inherited its intellectual [[lexicon]], as well as problems and methods of inquiry. As a result, his philosophy has exerted a unique influence on almost every form of knowledge in the West and it continues to be a subject of contemporary philosophical discussion.
```

#### Návrh riešenia:

Po rozbalení komprimovaného súboru môžeme začať s jeho spracovaním, potrebujeme rozdeliť daný XML súbor na samostatné Wiki články, indexovať jednotlivé články ktoré budeme ďalej spracovávať napríklad takto:

Nasledujúca postupnosť krokov sa bude opakovať ako je naznačené v kroku 7 a 8 a samozrejme ešte sa môže zmeniť, doplniť nejaké operácie nad dátami alebo niektoré zrušiť:

1. Vytvoriť testovaciu vzorku dát, na ktorej budeme prvotne projekt realizovať
2. Vytvoriť zoznam (strom) spoločensko-vedných oblastí, do ktorých budeme jednotlivé stránky zaraďovať, ku každej oblasti nájsť aj slová, ktoré sa s ňou spájajú
3. Články vhodne predspracovať - stemming, tokenizácia, odstránenie stop slov
4. Z článkov testovacej sady vyhľadať dôležité pojmy - zamerať sa na Infobox, kde sa nachádzajú dôležité informácie o článku
5. Vyhľadať odkazy na iné články Wikipédie (anchor text), ktoré môžu smerovať priamo na oblasť alebo aspoň priblížiť kontext článku
6. Z tela článku vyhľadať najčastejšie používané termy a tie, ktoré boli identifikované v ''kroku 2''
7. Na základe získaných údajov z predošlých krokov, teda slov z Infoboxov, textu článku a podľa Anchor textov vyhodnotiť úspešnosť riešenia testovacej sady a podľa toho prejsť kroky ''od 3 po 6'' a vylepšiť ich
8. Následne túto metódu použiť na celej vzorke článkov a vyhodnotiť relatívnu úspešnosť a znova zopakovať predošlé kroky za účelom jej zvýšenia.

Flow procesu pre jednu vzorku článku nájdete na nasledujúcom obrázku, kde je zobrazené, že nemusia sa opakovať všetky kroky, ale iba krok aktualizovania gazeteeru. Taktiež ukazuje, že najprv plánujeme vyhľadávať štatistickou metódou iba na základe slov, ktorú budeme vylepšovať a až následne sa budeme zameriavať na metódu Anchor článkov a prípadne aj porovnať ich presnosť. 

![Flow Diagram](https://github.com/tomasbabjak/VINF_Wikipedia/blob/main/diagram.png?raw=true)
  
#### Použité nástroje a frameworky:
GitHUb, Jupyter Notebook, Python, PySpark a ďalšie Python knižnice
  
#### Popis projektu 

V tomto projektu skúmame kategorizovanie článkov anglickej Wikipédie do vlastných kategórií, pričom porovnávame rôzne prístupy k trénovaniu modelu. Hlavnou motiváciou k tejto práci bolo sprehľadnenie článkov Wikipédie a ich zatriedenie do rôznych kategórií za účelom rýchlejšej orientácie a vyhľadávania medzi nimi. Taktiež sme chceli použiť čo najúčinnejší spôsob spracovania článkov, ako aj trénovania modelov, pričom používame často využívaný TF-IDF model a na vyhľadanie vektorovej vzdialenosti kosínusovú podobnosť. 

Ďalším zámerom tohto projektu bolo zistiť, ktorá časť článkov Wikipédie najlepšie vystihuje článok a teda pomocou ktorej dokážeme najpresnejšie zaradiť článok do kategórie. Konkrétne sme vybrali na porovnanie 4 črty článku a to infoboxy, teda stručné zhrnutia článkov, anchor texty, teda texty nachádzajúce sa pri odkazoch na iné články Wikipédie, vlastné kategórie Wikipédie a samotné texty článkov. Pre každú z týchto čŕt sme vytvorili osobitný natrénovaný model pozostávajúci z článkov Wikipédie s rovnakým názvom ako naše kategórie a vyhodnotili úspešnosť s trénovacou skupinou článkov. 

#### Existujúce riešenia 

Analyzované súčasné riešenia tejto problematiky sa zameriavajú výhradne na kategorizovanie článkov do tzv. menných entít, ako napríklad osoba, miesto, spoločnosť a podobne. Je to spôsobené najmä tým, že už samotná Wikipédia obsahuje vlastné kategórie, do ktorých jednotlivé články zaradzuje. V tomto projekte sme sa snažili vytvoriť náhradu tejto kategorizácia pomocou vlastnej techniky. 

Podobné práce zaoberajúce sa kategorizovaním článkov je napríklad od Shavarani et al. [1], kde sa snažia kategorizovať články Wikipédie v piatich rôznych jazykoch, pričom okrem tradičných metód normalizácie a tokenizácie používajú na určenie kategórií neurónové siete. Podobnou prácou je aj tá od Higashinaka et al. [2], v ktorej autori vytvorili dokonca 200 typov menných entít, do ktorých zaradzovali články Wikipédie. 

#### Popis riešenia 

Pri riešení projektu sme sa viac menej držali postupu riešenia, ktorý sme načrtli v časti Návrh riešenia.

##### Extrahovanie a parsovanie článkov 

Prvým krokom riešenia bolo vytvoriť testovaciu vzorku dát. Dáta sme testovali na vzorke 30 článkov zo začiatku Wikipédie. Na týchto dátach sme dokázali prejsť všetkými fázami projektu, aj tými náročnejšími na výpočtový výkon, ako bolo napríklad spracovanie textu článku. Túto vstupnú, ako aj výstupnú vzorku dát nájdete aj priloženú pri tomto projekte vo forme zip adresára.

Extrahovanie textov všetkých článkov prebiehalo pomocou čítania vstupného XML súboru po riadku a pomocou regex výrazov sme vyhľadávali <page> tagy a podľa nich sme text delili na články. Z týchto článkov sme podobným spôsobom vyhľadávali aj názvy článkov, texty a v nich anchor texty, kategórie a infoboxy obdobne pomocou regex výrazov. Od článkov sme následne oddelili redirect, teda články slúžiace na presmerovanie, ktoré pre nás v tomto projekte nemali priamy význam, takže sme s nimi už ďalej nepracovali.

##### Vytvorenie gazeteerov 

Taktiež sme vytvorili zoznam spoločensko-vedných oblastí do ktorých sme jednotlivé stránky zaradzovali, tento zoznam nájdete tiež ako prílohu k tomuto dokumentu. Pri vytváraní tohto zoznamu sme postupovali iteratívne, najprv sme vytvorili menší zoznam so siedmymi kategóriami, avšak to nám dostatočne nepostačovalo, takže sme ho rozšírili o ďalšie, až sme získali zoznam s 30 kategóriami, avšak aj v tomto zozname ešte stále  môžu chýbať niektoré kategórie, na ktoré sme zabudli.

Ku každej kategórii sme priradili slová a tak vytvorili gazeteer a to hneď dvoma spôsobmi:

  * pomocou Datamuse knižnice, ktorá priradzuje k jednotlivým slovám ich vektorovo najbližšie záznamy z ich databázy slov, následne sme takto nájdené výrazy tokenizovali a upravili tak, aby bol pri každej kategórii rovnaký počet slov
  * pomocou článkov Wikipédie s rovnakým názvom ako kategória. Týmto spôsobom sme vytvorili až štyri gazeteere, na základe slov v infoboxoch, anchor textov, vlastných Wikipédia kategórií a samozrejme textov týchto článkov

##### Predspracovanie textov 

Slová v gazeteeroch ako aj všetky trénovacie a testovacie články sme predspracovali rovnako. Na stemming slov sme použili Porterov stemmer, na tokenizáciu textov sme využili word_tokenizer knižnice NLTK a následne sme odstránili nepoužiteľné tokeny ako aj stop slová. 

##### Trénovanie a testovanie modelov

Na natrénovanie TF-IDF modelov sme použili všetky štyri trénovacie gazeteery a vlastné funkcie s pomocou knižnice scikit-learn a jej tried. 

Následne sme vykonávali testovanie modelov a to pomocou kosínusovej podobnosti, teda tak, že sme vektorizovali testovacie články podobne ako aj testovacie a vypočítavali sme ich kosínusovú podobnosť k článkom trénovacím za pomoci TF-IDF modelu. Túto podobnosť sme vypočítavali pre všetky 4 časti článku, teda infoboxy, anchor texty, kategórie a texty.

##### Indexácia a vyhľadávanie 

Za účelom vyhľadávania medzi článkami sme vytvorili vlastný invertovaný index pomocou hash mapy, teda defaultdict, pričom sme pridali aj možnosť vyhľadávať pomocou viacerých termov zároveň ako aj možnosť či chceme aby sa dané termy v článku vyskytovali súčasne alebo aspoň jeden z nich. Hľadané slová sú následne predspracované rovnako ako aj ostané články predtým a vyhľadané v indexe.   

#### Testovanie a vyhodnotenie 

Najprv sme testovali kategorické články Wikipédie, na ktorých sme si skúšali funkčnosť našich natrénovaných modelov. Následne sme testovali všetkých 30 článkov testovacej množiny. Tie sme najprv manuálne anotovali, teda priradili sme im kategórie, ktoré sme považovali za vhodné a až potom sme ich nechali ohodnotiť modelom. Rozdiel medzi našim ohodnotením, ktoré sme brali ako "ground truth", a ohodnotením natrénovaným modelom sme porovnali a vypočítali pre každý článok a pre každú porovnávanú črtu percentuálnu úspešnosť priradenia kategórie, a pre celý testovací korpus taktiež priemer, presnosť, úplnosť a F1 štatistiky. Testovali sme 4 modely, natrénoavané pomocou gazeteeru Datamuse, infoboxov, kategórií a textov Wikipédia kategorických článkov. Celé výsledky a štatistiky všetkých trénovaných modelov nájdete v prílohe k tomuto dokumentu.

Na základe vyhodnotenia testovacej sady článkov sme zistili, že najúspešnejšou metódou - gazeteerom bol Datamuse model, teda s vektorovo blízkymi slovami, ktorého priemernou úspešnosťou zistenia kategórii 60%, presnosťou 0.49, pokrytím 0.64 a metrikou F1 0.5, pričom iba o čosi menej úspešná bola aj metóda natrénovaná pomocou Infoboxov kategorických článkov s úspešnosťou 49%, presnosťou 0.58, pokrytím 0.49 a metrikou F1 na úrovni 0.47. 

Ostatné 2 metódy mali neporovnateľne slabšie výsledky, čo sa týka metódy trénovanej na kategóriách, tam bola príčinou nízky počet slov pre jednotlivé články a teda aj riedka matica, zatiaľ čo pri metóde trénovanej na textoch článkov bol problém opačný, teda veľmi rozsiahly text, ktorý nebol vždy práve veľmi reprezentatívny a teda mohlo nastať až pretrénovanie týchto modelov. 

#### Spustenie a použitie 

Celý zdrojový kód programu sa nachádza v súbore typu Jupyter Notebook VINF_Project_Wikipedia.ipynb, ktorý je celý spustiteľný po jednotlivých bunkách. Kód programu je rozdelený do niekoľkých logických častí a je psaný vo forme funkcii, ktoré je potrebné spúšťať za sebou v poradí ako sú v kóde, alebo je potrebné spustiť kód v poslednej časti programu - Spustenie.

V prvej bunke sa nachádzajú knižnice, ktoré je potrebné importovať pre správny bez programu.

Prvom časťou programu je kód na čítanie článkov Wikipédie zo súboru a extrahovanie textov článkov, ktoré sa zapíšu do súboru. Druhú časť tvorí nájdenie infoboxov, anchor textov a kategórii článkov. Tretia časť slúži na inicializáciu kategórii a vytvorenie gazeteera pomocou Datamuse knižnice a Wiki článkov kategórií. Nasleduje časť obsahujúca funkcie na predspracovanie článkov a po nej funkcie na trénovanie a testovanie TF-IDF modelov s vytvorením invertovaného indexu a vyhľadávania nad článkami.  

Poslednú časť tvorí samotné spustenie programu a vyhodnotenie na testovacích, ale aj väčších dátach spolu s testovaním vyhľadávača.

#### Konzultácia č. 3

  * Vytvorenie testovacej vzorky dát
  * Extrahovanie nadpisu a textu článku
  * Nájdenie Infoboxov a Anchor textov z textu článku
  * Oddelenie Redirect článkov od plnohodnotných článkov
  * Vytvorenie kategórii
  * Vytvorenie gazeteerov pre jednotlivé kategórie pomocou Datamuse
  * Exact match priradenie kategórii k článkom podľa slov v gazeteeroch
  
#### Konzultácia č. 4

  * Extrahovanie //Category// poľa z článkov
  * Rozbitie vlastných kategórii na menšie jednoslovné kategórie
  * Úprava gazeteerov - Datamuse
  * Spracovanie článkov Wikipédie s rovnakým názvom ako má moja kategória a vytvorenie gazeteeru
  * Tokenizácia, Stemming a odstránenie stop slov z článkov
  * Predspracovanie slov v gazeteeroch
  * TF-IDF model nad slovami v gazeteeroch
  * Kosínusová podobnosť s kategorickými a testovacími článkami
  * Skúsiť Spark - pyspark na rýchlejšie spracovanie

#### Konzultácia č. 5 

  * Anotovanie testovacej vzorky dát
  * Testovanie a vyhodnotenie úspešnosti natrénovaných modelov na testovacej vzorke
  * Vytvoriť jednoduchý index nad všetkými článkami pomocou hash mapy 
  * Vyhľadávanie nad článkami - nájsť názov článku podľa kategórie a textu článku
  * Spustenie nad všetkými článkami Wikipédie - nájsť text, extrahovať infobox, anchors, kategórie a odčleniť redirect články


#### Konzultácia č. 6 

  * Rozdelenie testovacej vzorky na menšie časti
  * Spustenie celého procesu na čo najväčšej vzorke dát (ideálne na všetkých) - extrahovanie článkov a z nich text, anchors, infobox, predspracovanie - odstránenie stop slov, normalizácia, tokenizácia, testovanie pomocou modelov, vytvorenie indexu a vyhľadávanie nad nimi
  * Vylepšenie modelov na základe 5. konzultácie

#### Literatúra 

[1] SHAVARANI, Hassan S.; SEKINE, Satoshi. Multi-class Multilingual Classification of Wikipedia Articles Using Extended Named Entity Tag Set. arXiv preprint arXiv:1909.06502, 2019.

[2] HIGASHINAKA, Ryuichiro, et al. Creating an extended named entity dictionary from Wikipedia. In: Proceedings of COLING 2012. 2012. p. 1163-1178.
