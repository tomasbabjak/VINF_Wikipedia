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

##### Krok 1: Zoznam spoločensko-vedných oblastí, do ktorých budeme jednotlivé stránky zaraďovať ===

  * Literature - General reference
  * Culture and the arts
  * Geography and places and states
  * Health and fitness and medicine
  * History and events
  * Human activities
  * Mathematics and logic
  * Natural and physical sciences
  * People and self
  * Philosophy and thinking
  * Religion and belief systems
  * Society and social sciences
  * Technology and applied sciences
  
#### Použité nástroje a frameworky:
GitHUb, Jupyter Notebook, Python, PySpark a ďalšie Python knižnice

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
  
