# **PCLP3 PROJECT - PART 1**
##  Predictia meciurilor de fotbal
314CA Stefan Alexandru Vladut

## Introducere
Scopul proiectului este de a prezice rezultatele unor meciuri internationale de fotbal pe baza unor caracteristici precum: `home_team`, `away_team`, `rank_difference`, `last_5_games`, `match_level`, `neutral_field`. Variabila target
este dupa cum probabil v-ati fi gandit `result`, care inmagazineaza rezultatul final ce poate avea 3 outcome-uri:
-   `0` = victorie echipa oaspete
-   `1` = egal
-   `2` = victorie echipa gazda

Iar modelul trebuie sa invete pe baza datelor din `train.csv`, sa prezica rezultatul unui meci din `test.csv`

`1. Tipul problemei`
-   Problema aleasa este una de *clasificare multiclasa*, deoarece variabila tinta asa cum am spus mai sus poate lua 3 valori, iar modelul trebuie sa aleaga dintre acestea.

`2. Structura setului de date`
-   Am creat asa cum zice cerinta 700 de instante, 500 de antrenare si 200 de test in dataset-ul final, fiecare instanta reprezentand un meci.
-   Dataset-ul este impartit in 2 subseturi:
-   `train.csv` = subset de antrenare
-   `test.csv` = subset de testare
-   Impartirea a fost realizata prin extragere randomizata, folosind functia `train_test_split` din biblioteca `scikit-learn`
-   In plus, am folosit parametrul `stratify=df["result"]` pentru a pastra distributia celor 3 posibile valori ale variabilei tinta in ambele subseturi echilibrata, sa aiba proportii asemanatoare de victorie, egal, respectiv infrangere.
-   Astfel, rezultatele din dataset-ul final se vor regasi si in cel de antrenare si in cel de test.

`3. Caracteristicile din tabel`
-   Dataset-ul final contine 8 coloane inclusiv cu coloana tinta *result*
-       home_team -> valoare categoriala de tip string, echipa gazda
-       away_team -> valoare categoriala de tip string, echipa oaspete
-       rank_difference -> numar intreg, diferenta in valoare absoluta dintre rank-urile celor 2 echipe
-       home_form_last_5 -> numar real, forma recenta a echipei gazda in ultimele 5 meciuri, pe o scara de la 0 la 1 intrucat asa evit valori numerice mari si e usor de citit in forma normalizata
-       away_form_last_5 -> numar real, forma recenta a echipei oaspete in ultimele 5 meciuri, pe o scara de la 0 la 1
-       neutral_location -> numar intreg, valoare booleana arata daca meciul este pe teren neutru sau nu(0 = nu, 1 = da)
-       importance_level -> valoare categoriala de tip string, indica tipul competitiei in care ia parte meciul
-       result -> valoare categoriala de tip intreg, ia 3 posibile valori(0,1 sau 2) 
-   Am respectat cerinta de a avea minim 8 coloane relevante inclusiv coloana tinta, dar si 3 tipuri de date diferite(numere intregi, numere reale, valori categoriale si stringuri)

`4. Mod de construire dataset si salvare`
-   Metoda aleasa pentru construirea datasetului este cea de `generare sintetica`, nu provine dintr-un fisier existent, ci a fost generat prin simulare unor meciuri fictive internationale
-   Am importat o lista de 42 de tari, iar pentru fiecare echipe am atribuit un rank, bazandu-ma pe clasamentul FIFA de la ora realizarii proiectului.
-   Pentru fiecare meci au fost generate o echipa gazda si o echipa oaspete. Rank-ul fiecarei echipe a fost obtinut pornind de la rank-ul de baza, peste care s-a adaugat o variatie mica intre -3 si +3 pozitii. Aceasta variatie simuleaza fluctuatii normale de forma sau clasament.
-   Pentru fiecare meci au fost generate:
-       forma recenta a echipei gazda
-       forma recenta a echipei oaspete
-       daca meciul este sau nu pe teren neutru
-       tipul competitiei
-       valori interne pentru ofensiva/defensiva echipelor
-       expected_goals pe baza caracteristicilor
-   De asemenea, in generarea ofensivei si apararii am generat valori rotunjite la 2 zecimale intre `0.4 - 1` ca sa nu existe echipe complet slabe, la fel in alegerea terenului probabilitatea fiind mai mare (`75%`) de a nu avea meciuri neutre, acest lucru poate fi inversat in vederea World Cup 2026 care sta sa inceapa :)
-   Rezultatul final a fost simulat folosind valorile expected_goals si folosind distributia Poisson
### P.S: Iar pentru a face dataset-ul mai apropiat de realitate am introdus artificial un procent mic de valori lipsa in coloanele `home_form_last_5`, `away_form_last_5` si `importance_level`. Nu au fost introduse valori lipsa in coloana tinta `result`.

`5. Analiza valorilor lipsa`
-   In urma analizei valorilor lipsa au fost gasite valori lipsa in urmatoarele coloane:
-   `home_form_last_5`
-   `away_form_last_5`
-   `importance_level`

-   In subsetul de antrenare, au fost identificate 17 valori lipsa, adica un procent de 3,4% din date. Iar in subsetul de testare, au fost identificate 4 valori lipsa, adica un procent de 2% din date.
-   Am analizat doua strategii de tratare a valorilor lipsa:
### 5.1 Stergerea randurilor incomplete
-   Metoda e intuitiva, empirica doar ca poate duce la pierderea datelor si la reducerea datasetului.
-   M-am folosit de functia `dropna()` care sterge randurile care au cel putin o valoare lipsa(NaN)
### 5.2 Imputarea valorilor lipsa
-   Aceasta strategie e si mai "profi", dar ajuta si la integritatea oricarui dataset, deci pe aceasta am mizat in continuarea proiectului.
-   Fiind tipuri diferite de date am avut 2 alegeri in functie de tipul de date al valorilor din coloane
-   Pentru coloanele numerice `home_form_last_5` si `away_form_last_5`, valorile lipsa au fost inlocuite cu mediana calculata pe subsetul de antrenare. Am ales mediana deoarece este mai robusta la valori extreme.
-   Pentru coloana categorica `importance_level`, valorile lipsa au fost inlocuite cu cea mai frecventa valoare din subsetul de antrenare.

`6. Statistici descriptive`
-   Pentru valorile numerice am folosit functia `describe()` pe subseturile de antrenare si testare
-   Observam si campurile din tabelul generat
-       `count` -> numarul de valori
-       `mean`  -> media 
-       `std`   -> deviatia standard(cat de imprastiate sunt valorile fata de medie) 
-       `min`   -> valoarea minima
-       `25%`   -> primul quartil, 25% dintre valori sunt mai mici sau egale decat o valoare
-       `50%`   -> jumatatea, 50% dintre valori sunt mai mici sau egale decat o valoare
-       `75%`   -> al treilea quartil, 75% dintre valori sunt mai mici sau egale decat o valoare
-       `max`   -> valoarea maxima

-   Pentru valorile categoriale am folosit tot functia `describe()`, alaturi de parametrul `object`
-       `count`  -> numarul de valori
-       `unique` -> numarul de valori unice/distincte
-       `top`    -> cea mai frecventa tara
-       `freq`   -> de cate ori apare valoarea de la top

`7. Analiza distributiilor variabilelor`
-   `Coloana rank_difference`
-       Histograma arata ca cele mai multe meciuri au o diferenta de rank intre 0-15, sugerand ca majoritatea meciurilor au loc intre echipe apropiate ca nivel. Acest lucru este esential in antrenarea modelului pentru a-l ajuta sa distinga meciurile echilibrate de cele dezechilibrate.
-   `Coloanele home/away_form_last_5`
-       Aici forma echipei este pe o scara normalizata de la 0 la 1, fiind usor de citit si usor de integrat in calculele folosite pentru a nu avea valori mari. Se observa diversitate echipele au atat forme bune(cele apropiate de 0, cat si forme excelente cele apropiate de 1).
-       Masuram atat forma echipei gazda, cat si forma echipei oaspete pentru a asigura corectitudine si a avea toti parametrii necesari pentru prezicerea cat mai corecta a scorului final
-   `Neutral location`
-       Probabilitatea setata la inceput confirma ca majoritatea meciurilor nu se joaca pe teren neutru, ceea ce arata ca echipa gazda are un avantaj
-   `Result`
-       In distributia celor 3 clase ale variabilei result observam ca echipa gazda e favorita ceea ce confirma parametrii alesi in favoarea lor
-       In plus, pentru a pastra distributia claselor variabilei atat la train, cat si la test am folosit parametrul *stratify* asa cum am adaugat si mai sus.
-   `Importance Level`
-   Se confirma parametrii setati initial, cele mai multe meciuri fiind de tip `friendly`
-   Fiind o valoare categoriala, ea va trebuie encodata inainte de antrenarea modelului
`   Distributia echipelor gazda/oaspete in subsetul de antrenare`
-   De asemenea, necesita encodare
-   Se observa o continuitate ceea ce arata ca nu sunt diferente catrastrofale in vederea imparitii echipelor

`8. Detectarea outlierilor`
-   Pentru detectarea outlierilor am folosit boxplot-uri pentru coloanele numerice:
-       rank_difference
-       home_form_last_5
-       away_form_last_5  
-   De adaugat ca pentru coloanele categorice nu am analizat prin boxplot, deoarece pentru acestea sunt potrivite grafice de tip countplot. De asemenea, `neutral_location` si `result` nu au fost analizate ca outlieri, deoarece sunt variabile categoriale codificate numeric.
-   Pentru `rank_difference` majoritatea meciurilor au diferente de rank mici, insa sunt si cateva puncte izolate reprezentant posibili outlieri. Totusi, sunt situatii posibile si intalnite in fotbal, deci pot oferi informatii utile modelului. De asemenea, mediana este in jur de 12 ceea ce inseamna ca majoritatea meciurilor au diferenta mai mica sau egala decat 12.

![Rank_diff_boxplot](Pictures/rank_diff_boxplot.png)

-   Pentru `home_form_last_5` si `away_form_last_5`, valorile sunt in intervalul 0-1. Valorile apropiate de 0 sau 1 nu reprezinta erori, ci situatii valide: forma foarte slaba sau foarte buna.

`9. Analiza corelatiilor(heatmap)`
-   Am construit o matrice de corelatii(heatmap) pentru variabilele numerice pentru a vedea posibile relatii intre campuri, insa majoritatea sunt apropiate de 0 ceea ce arata ca nu exista relatii liniare puternice intre variabile.
-   Spre exemplu, rank_difference si result au o corelatie aproape de zero, sugerand ca nu exista o relatie directa. Totusi, acest rezultat nu reflecta realitatea absoluta. Rank_difference marcheaza diferenta de nivel, nu care echipa e mai buna.
-   Nu a fost necesara eliminarea unei coloane pe baza heatmap-ului intrucat valorile sunt apropiate de 0.

![Heatmap](Pictures/heatmap.png)

## P.S Am folosit si prescurtari pentru bibliotecile folosite sns de la seaborn si pd de la pandas

`10. Analiza relatiilor cu varianta tinta(result)`
-   Pentru analize relatiilor dintre variabile si target am folosit `violin plot-uri` si `count plot-uri` cu hue = result
-   violin_plot = vedem distributia unei variabile in functie de o categorie
-   count_plot = vedem numarul de aparitii al unei categorii, iar hue = result imparte rezultatele dupa valoarea lui result
-   `Coloana rank_difference`
-       Distributiile sunt asemanatoare, medianele la fel. Asta inseamna ca diferenta absoluta dintre rank-urile celor 2 echipe nu influenteaza categoric deznodamantul meciului

![Rank_diff_result](Pictures/rank_diff_result.png)

-   `Coloanele home/away_form_last_5`
-       Se observa distributia formei echipei gazda/oaspete recenta. Pentru cele 3 clase, aceste distributii se suprapun destul de mult ceea ce inseamna ca nu e un parametru definitoriu care sa impinga meciul intr-o directie.

![Home_result](Pictures/home_result.png)
![Away_result](Pictures/away_result.png)

-   `Coloana neutral location`
-       Acest count plot este util deoarece terenul poate influenta rezultatul dupa cum vedem, sansele ca gazdele sa castige cresc enorm in cazul in care joaca pe teren propriu

![Neutral_loc](Pictures/neutral_loc.png)


-   `Coloana importance_level`
-       Asemenea ca cea de mai sus, o singura competitie influenteaza rezultatul si aceea tine tot de terenul pe care se desfasoara majoritatea meciurilor. Intr-o cupa mondiala situatia e in balanta intrucat terenul e neutru si nu mai e avantajata echipa gazda.

![Importance_level](Pictures/imp_level.png)


`10. Antrenarea si evaluarea unui model de baza`

-   Pentru antrenarea unui model am folosit `Random Forest` din biblioteca `scikit-learn`. Am ales acest model deoarece este potrivit pentru probleme de `clasificare multiclasa` si poate invata relatii neliniare intre caracteristici.
-   Inainte de antrenare, variabilele categoriale au fost encodate in variabile numerice folosind `OneHotEncoder`
-   Modelul a fost antrenat pe subsetul de train si evaluat pe subsetul de test
-   Metricele au fost urmatoarele:
-      acuratete
-      precizie
-      recall = cat de bine detecteaza modelul o clasa
-      F1-score = precision + recall

-   Drept grafic relevant am introdus si o matrice de confuzie care arata pe diagonala principala ce exemple au fost prezise corect si in rest ce marja de eroare avem. Egalurile sunt cele mai greu de separat si de prezis corect avand in vedere numarul redus de parametri in care se poate obtine aceasta conjuctura.
-   Random Forest e un model foarte bun de inceput, insa poate fi imbunatatit prin caracteristici suplimentare.

`11. Concluzii`
-   In acest proiect am construit un dataset tabelar sintetic pentru predictia rezultatului unui meci international de fotbal. Datasetul contine 700 de instante si 8 coloane relevante, incluzand coloana tinta
-   Am realizat analiza exploratorie a datelor, am tratat valorile lipsa, am analizat distributiile, outlierii, corelatiile si relatiile cu variabila tinta. La final, am antrenat un model de baza folosind `RandomForestClassifier`
-   Rezultatele obtinute arata ca modelul poate identifica partial victoriile gazdei si ale oaspetilor, dar are dificultati in prezicerea egalurilor. Acest lucru evidentiaza limitarile datasetului si arata directii posibile de imbunatatire