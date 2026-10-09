# Use Case Pharma — jeu de données

> **Source :** page Confluence « 2026-09-15 : Use Case Pharma - Jeu de données »
> ([82345985](https://c-pal.atlassian.net/wiki/pages/viewpage.action?pageId=82345985), version 9),
> importée le 2026-10-09 ([KAN-104](https://c-pal.atlassian.net/browse/KAN-104)).
> Ce fichier est désormais la référence ; la page Confluence n'est plus mise à jour.
>
> **Production vs local.** Les identifiants cités ici (`ClientID` 3 et 4,
> `UserID` 4, Frédéric Marteau) et les étapes de chargement du §10 / §12 décrivent
> le chargement en **production** du 29/09/2026. L'outil local
> ([`cpaltracker.web/tools/demo-data/pharma`](../../cpaltracker.web/tools/demo-data/pharma/README.md))
> n'utilise aucun identifiant fixe et crée son propre utilisateur — voir
> [pharma-demo-dataset.md](pharma-demo-dataset.md). Journal du chargement en
> production : page Confluence [91848706](https://c-pal.atlassian.net/wiki/pages/viewpage.action?pageId=91848706).
>
> Hypothèses de simulation : [pharma-use-case-simulation.md](pharma-use-case-simulation.md).

> **Statut :** périmètre révisé le 24/09/2026 (flotte 350 palettes, modèle v9, horizon mai 2029). **Les arbitrages bloquants sont tranchés** et le fichier `T_DATA` est **généré et contrôlé** : 1 401 155 lignes. Restent les schémas des 8 autres tables et deux conventions mineures (§13).
> 
> **Cible :** base `cpaltracker` (MariaDB 10.11, VPS Hostinger). Données de simulation entièrement réversibles (voir §12).
> 
> **Scénario chargé :** celui **sans anticipation** — seuil de réapprovisionnement à 0 kg, le client repasse commande quand le stock de sa ligne atteint zéro. Les scénarios « 1 t » et « 2 t » restent consultables dans le tableau de bord à des fins de comparaison, mais ne sont pas injectés.

# 0\. Préambule

dans la table T\_DDLOT il faut ajouter un champ texte nommé « Product\_Name »

dans le firmware il faut revoir la gestion de l'alarme de poids

dans cpaltracker il faut une table des anomalies

# 1\. Objectifs de la démonstration

Le détenteur d'un contrat MAIN (Innopharm) doit pouvoir :

1.  faire l'inventaire d'un lot de production ou d'un « Product\_Name » à une date donnée ;

2.  identifier les anomalies de température, humidité, poids et accélération sur l'ensemble des palettes du contrat, ou bien pour un lot, ou bien pour un produit ;

3.  visualiser l'évolution de la consommation chez ses clients pour un produit ou pour un lot ;

4.  identifier les palettes vides et leur localisation.

# 2\. Acteurs et contrats

| Acteur               | Rôle                                                 | Traduction base                                                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **Innopharm**        | Fabricant pharmaceutique, détenteur du contrat       | `T_CLIENTS.ClientID = 3` + contrat **MAIN** du 01/09/2026 au 01/09/2030                  |
| **Cust01**           | Client d'Innopharm, bénéficie d'une visibilité c-PAL | `T_CLIENTS.ClientID = 4` + contrat **STD** enfant du MAIN Innopharm                      |
| **Cust02 → Cust07**  | Clients d'Innopharm, sans compte c-PAL               | Destinations GPS uniquement — aucune ligne `T_CLIENTS`                                   |
| **Frédéric Marteau** | Exploitant des données côté Innopharm                | `T_USER.UserID = 4`, compte existant, réutilisé comme `CreatedBy` des 57 dossiers de lot |

Innopharm travaille avec **7 clients** (Cust01 à Cust07), tous présents dans le tableau de commandes. Les comptes Diego Espino et Inès Calori sont abandonnés.

# 3\. Flotte de palettes

  - **350 palettes** instrumentées, `ChipID` de `cpaldev001` à `cpaldev350`. Périmètre révisé le 24/09/2026 : la valeur précédente de 1 000 palettes générait un volume de mesures sans contrepartie pour la démonstration.

  - Capacité unitaire **1 000 kg**. `MASSE` mesure la **charge nette** : 0 kg palette vide, pas de tare.

  - Livraison de la flotte chez Innopharm le → événement `Créé` dans `T_DEVICE_HIST`, rattachement au MAIN dans `T_CONTRATS_DEVICES`.

  - Sur l'horizon simulé, **303 palettes** sont chargées au moins une fois. Le pic de palettes simultanément chargées est de **303, atteint le 03/03/2028** : il reste alors 47 palettes vides disponibles.

  - Le parc de palettes vides à Lyon oscille entre **47 et 159 palettes** sur toute la durée : il alimente l'objectif n°4 en continu.

  - La flotte est conservative : à chaque date, palettes chargées + palettes vides = 350, sans exception sur les 974 jours.

# 4\. Sites et géographie

| Site                                                              | Latitude  | Longitude | Usage                                 |
| ----------------------------------------------------------------- | --------- | --------- | ------------------------------------- |
| Atelier de production Innopharm — ZAC des Milles, Aix-en-Provence | 43.486267 | 5.373669  | Remplissage des palettes              |
| Zone de stockage / quarantaine CQ                                 | 43.482708 | 5.378727  | Contrôle qualité 6 jours              |
| Entrepôt logistique Lyon                                          | 45.668680 | 4.911053  | Stock libéré + parc de palettes vides |
| Cust01 — Mairie de Bourg-en-Bresse (01)                           | 46.205234 | 5.225304  | Livraison                             |
| Cust02 — Mairie de Laon (02)                                      | 49.565046 | 3.620619  | Livraison                             |
| Cust03 — Mairie de Moulins (03)                                   | 46.566133 | 3.333154  | Livraison                             |
| Cust04 — Mairie de Digne-les-Bains (04)                           | 44.093419 | 6.235416  | Livraison                             |
| Cust05 — Mairie de Gap (05)                                       | 44.558742 | 6.079752  | Livraison                             |
| Cust06 — Mairie de Nice (06)                                      | 43.695981 | 7.271484  | Livraison                             |
| Cust07 — Mairie de Privas (07)                                    | 44.735222 | 4.599058  | Livraison                             |

# 5\. Produits et lots

**10 produits** (Produit 1 à Produit 10), péremption **12 mois à compter du début de production**. Un lot = **20 tonnes = 20 palettes de 1 000 kg**, toutes rattachées au même dossier de lot. Les numéros `2026001`, `2027001`, … sont les **numéros de lot Innopharm** (`T_DDLOT.Cust_Lot_Number`), incrémentés par année ; le code `cPAL_Lot` est généré par la base et n'a pas à être choisi.

`Number_Of_Products` : unité de conditionnement de 5 kg, soit **4 000 unités par lot** de 20 t.

## 5.1 Une seule vague planifiée, puis un flux tiré

Contrairement au cadrage initial qui prévoyait deux vagues planifiées, **seule la vague 1 est planifiée** : elle fabrique un lot par produit, en séquence, du 01/10/2026 au 19/12/2026. Toutes les campagnes suivantes sont **déclenchées par le stock réel**, produit par produit.

Une vague est lancée pour un produit quand son **stock d'échelon** — palettes vivantes à Lyon **et** déjà chez les clients mais non encore consommées, plus la production en cours — ne couvre plus la demande sur (délai d'obtention + 60 jours de couverture). C'est un pilotage à deux échelons : ce que l'aval détient, l'amont n'a plus à le porter.

Sur l'horizon : **40 vagues** et **57 lots** (10 planifiés + 47 déclenchés par le stock), soit 1 140 tonnes produites.

## 5.2 Plan de production — vague 1

| Lot     | Produit    | Production         | Fin CQ / libération / arrivée Lyon | Péremption | Palettes       |
| ------- | ---------- | ------------------ | ---------------------------------- | ---------- | -------------- |
| 2026001 | Produit 1  | 01/10 → 08/10/2026 | 14/10/2026                         | 01/10/2027 | cpaldev001–020 |
| 2026002 | Produit 2  | 09/10 → 16/10/2026 | 22/10/2026                         | 09/10/2027 | cpaldev021–040 |
| 2026003 | Produit 3  | 17/10 → 24/10/2026 | 30/10/2026                         | 17/10/2027 | cpaldev041–060 |
| 2026004 | Produit 4  | 25/10 → 01/11/2026 | 07/11/2026                         | 25/10/2027 | cpaldev061–080 |
| 2026005 | Produit 5  | 02/11 → 09/11/2026 | 15/11/2026                         | 02/11/2027 | cpaldev081–100 |
| 2026006 | Produit 6  | 10/11 → 17/11/2026 | 23/11/2026                         | 10/11/2027 | cpaldev101–120 |
| 2026007 | Produit 7  | 18/11 → 25/11/2026 | 01/12/2026                         | 18/11/2027 | cpaldev121–140 |
| 2026008 | Produit 8  | 26/11 → 03/12/2026 | 09/12/2026                         | 26/11/2027 | cpaldev141–160 |
| 2026009 | Produit 9  | 04/12 → 11/12/2026 | 17/12/2026                         | 04/12/2027 | cpaldev161–180 |
| 2026010 | Produit 10 | 12/12 → 19/12/2026 | 25/12/2026                         | 12/12/2027 | cpaldev181–200 |

Le remplissage est progressif : 20 palettes réparties sur les 8 jours de production, soit 2 à 3 palettes closes par jour. Une palette passe de 0 à 1 000 kg le jour où elle est remplie.

Répartition finale des 57 lots par produit : P1 = 4, P2 = 7, P3 = 4, P4 = 9, P5 = 5, P6 = 7, P7 = 4, P8 = 10, P9 = 4, P10 = 3. L'écart reflète les dynamiques de consommation des clients, pas une règle de production.

La capacité de l'atelier reste théorique : la production réellement simulée suit strictement la demande, soit **1 140 t sur 32 mois** (57 lots de 20 t) pour une demande client de **≈ 1 124 t**.

## 5.3 Péremption — cas recherchés pour la démonstration

La péremption n'est pas un blocage : un lot périmé **à la date de livraison** n'est pas expédié, mais une palette déjà livrée est consommée jusqu'au bout et le dépassement est signalé. Sur l'horizon :

  - **127 palettes** sur les 1 100 livrées terminent leur consommation **après** la date de péremption de leur lot ;

  - **1 lot** conserve un reliquat périmé à Lyon (3 palettes).

C'est le matériau de l'objectif n°2.

# 6\. Commandes et profils de consommation

Source de vérité : la feuille de paramètres v7, rangée dans le dossier Drive du Use Case — [c-PAL Simu Pharma — Paramètres v7](https://docs.google.com/spreadsheets/d/1kcEzEGtCWHwbsVkN1bZbqVL__WD2n2C_9ol5Owkg7X4/edit). Elle remplace la feuille « Profil de consommation Produits » du cadrage initial.

**34 lignes de commande** (couple client + produit), chaque client travaillant sur 4 à 6 produits et chaque produit servant au moins 2 clients. Sur l'horizon, ces 34 lignes produisent **179 commandes** successives.

## 6.1 Grammaire des profils

| Code                                  | Signification                   | Courbe de masse                                                          |
| ------------------------------------- | ------------------------------- | ------------------------------------------------------------------------ |
| `Lin3` / `Lin6` / `Lin9` / `Lin12`    | Linéaire sur N mois             | Décroissance régulière de 1 000 kg à 0 sur N mois                        |
| `FDsc2` / `FDsc4` / `FDsc6` / `FDsc8` | Front descendant au N-ième mois | Masse constante à 1 000 kg, puis chute brutale à 0 au N<sup>e</sup> mois |

**La colonne Volatilité est désormais utilisée** — elle ne l'était pas au cadrage initial. Elle vaut **2 ou 3** selon le profil et se combine au **choix de simulation** (`-1`, `0`, `+1`) :

    durée effective (mois) = durée nominale du profil + (volatilité × choix)

Exemple : un `Lin12` de volatilité 3 avec un choix `-1` est consommé en 9 mois. La durée est bornée à 1 mois minimum. Les commandes 1 et 2 d'une ligne utilisent les choix V1 / V2 saisis dans la feuille ; à partir de la 3<sup>e</sup>, le choix est tiré au hasard avec une graine fixe (2026), ce qui rend la simulation reproductible.

La courbe est tracée sans bruit aléatoire superposé : la volatilité joue sur la **durée**, pas sur la forme.

## 6.2 Consommation en série

Une ligne de commande consomme ses livraisons **en série** (FIFO) : un client termine une livraison avant d'entamer la suivante. C'est ce qui garantit que la demande client totale est la même quel que soit le scénario de seuil, et que seul le pilotage change.

## 6.3 Premières livraisons (novembre 2026)

| Date       | Client | Palettes | Lots                      |
| ---------- | ------ | -------- | ------------------------- |
| 02/11/2026 | Cust01 | 1        | 2026002                   |
| 02/11/2026 | Cust02 | 14       | 2026001                   |
| 02/11/2026 | Cust03 | 9        | 2026002                   |
| 02/11/2026 | Cust04 | 9        | 2026002                   |
| 02/11/2026 | Cust05 | 10       | 2026002 (1) + 2026003 (9) |
| 02/11/2026 | Cust06 | 6        | 2026003                   |
| 02/11/2026 | Cust07 | 6        | 2026001                   |
| 09/11/2026 | Cust02 | 4        | 2026004                   |
| 09/11/2026 | Cust03 | 9        | 2026004                   |
| 09/11/2026 | Cust06 | 7        | 2026004                   |

**Règle de livraison :** livraison le lendemain de la plus tardive des deux dates — date de commande ou disponibilité du lot à Lyon ; une livraison tombant un dimanche est reportée au lundi.

Fin de consommation d'une palette = date d'épuisement de sa ligne. La palette devient alors vide, est enlevée 5 jours plus tard et repart vers Lyon en 1 jour.

## 6.4 Ruptures

**5 commandes** ne sont pas entièrement servies, pour **24 palettes** au total, toutes entre le 14/11/2028 et le 22/05/2029 — c'est-à-dire en fin d'horizon, quand une dernière vague de production n'a plus le temps d'être lancée. Ce ne sont pas des ruptures systémiques du modèle.

# 7\. Transports

| Trajet                                              | Cadence de mesure | Comportement des capteurs               |
| --------------------------------------------------- | ----------------- | --------------------------------------- |
| Lyon → Aix (palettes vides à réemployer)            | 1 mesure / 20 min | Masse à 0 kg                            |
| Aix (CQ) → Lyon, par autoroute                      | 1 mesure / 20 min | Température, humidité, masse constantes |
| Lyon → client, route la plus rapide                 | 1 mesure / 30 min | Température, humidité, masse constantes |
| Client → Lyon (retour à vide)                       | 1 mesure / 30 min | Masse à 0 kg                            |
| Tout site à l'arrêt (atelier, CQ, entrepôt, client) | 1 mesure / 6 h    | Position fixe                           |

Le tronçon Lyon → Aix n'existait pas au cadrage initial : il apparaît dès lors que les palettes vides sont **réemployées** pour les vagues déclenchées par le stock. Sur l'horizon, 940 des 1 140 remplissages sont assurés par une palette de retour.

## 7.1 Décalage de transmission en transport

Aucune passerelle LoRa n'est disponible sur les axes routiers. Les relevés de transport sont mémorisés par le device et transmis à l'arrivée, avec un **décalage de 8 heures** :

  - `RECORD_TIME` (reconstruit depuis `YEAR`…`SEC`) = heure réelle de la mesure ;

  - `received_at_ttn` = `RECORD_TIME` + 8 h ;

  - `timestamp_utc` = `received_at_ttn` + 0 à 3 s.

Contrôlé sur le fichier produit : `DIFF_TTN_RECORD_SEC` vaut exactement 28 800 s sur les 40 175 relevés de transport et 0 s sur les 1 360 980 relevés au repos ; `DIFF_UTC_TTN_SEC` reste entre 0 et 3 s. La règle de repli EventDate (`RECORD_TIME` \< `timestamp_utc` + 1 jour) est respectée sur toutes les lignes.

## 7.2 Tracé routier — itinéraires réels retenus

Les 8 itinéraires ont été calculés sur le réseau routier réel et stockés en JSON, les retours réutilisant l'itinéraire inverse. Chaque tracé a été contrôlé automatiquement : écartement des points aberrants, puis vérification que la longueur reconstituée de la polyligne correspond à la distance annoncée par le service de calcul (écart constaté de −2,3 % à −6,8 %, la géométrie simplifiée coupant les virages — davantage sur Digne et Gap, routes de montagne).

Les durées ci-dessous sont les durées **poids lourd** : durée voiture × 1,15, plus 45 min de pause réglementaire au-delà de 4 h 30 de conduite.

| Trajet                          | Distance | Durée PL | Cadence | Relevés par palette |
| ------------------------------- | -------- | -------- | ------- | ------------------- |
| Aix (CQ) ↔ Lyon                 | 298,0 km | 3 h 35   | 20 min  | 11                  |
| Lyon → Cust01 (Bourg-en-Bresse) | 92,3 km  | 1 h 21   | 30 min  | 3                   |
| Lyon → Cust02 (Laon)            | 577,0 km | 7 h 52   | 30 min  | 16                  |
| Lyon → Cust03 (Moulins)         | 218,2 km | 3 h 00   | 30 min  | 7                   |
| Lyon → Cust04 (Digne-les-Bains) | 277,3 km | 4 h 26   | 30 min  | 9                   |
| Lyon → Cust05 (Gap)             | 199,7 km | 3 h 19   | 30 min  | 7                   |
| Lyon → Cust06 (Nice)            | 465,4 km | 6 h 25   | 30 min  | 13                  |
| Lyon → Cust07 (Privas)          | 136,1 km | 1 h 52   | 30 min  | 4                   |

Heures de départ retenues (UTC) : 05:00 pour Lyon → client, 06:00 pour CQ → Lyon, 07:00 pour Lyon → Aix à vide, 08:00 pour le retour à vide. Les relevés au repos tombant dans la fenêtre de roulage sont remplacés par la cadence de transport ; ceux d'avant départ sont positionnés à l'origine, ceux d'après arrivée à la destination.

# 8\. Anomalies injectées

Les deux anomalies du cadrage initial sont conservées, avec les effectifs de palettes du modèle v9 et un profil A1 ajusté à la durée réelle du trajet.

| ID     | Nature                                             | Localisation                                                                                                                                                                                                                                              | Enregistrements produits                                                                                                                   |
| ------ | -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **A1** | Excursion de température                           | Livraison Lyon → Cust02 (Laon) du 02/11/2026, le trajet le plus long : départ 05:00 UTC, arrivée 12:52. **14 palettes, toutes du lot 2026001.** Excursion à partir de 05:30 : montée 22 → 48 °C en 2 h, plateau 48 °C pendant 3 h, retour à 22 °C en 2 h. | 16 relevés par palette, soit 224 enregistrements de transport, dont **112 au-dessus du seuil de 40 °C** (8 par palette, de 07:00 à 10:30). |
| **A2** | Freinage brutal — palettes potentiellement abîmées | Livraison Lyon → Cust06 (Nice) du 09/11/2026, un point unique sur l'A8 à 10:30 UTC (5 h 30 après le départ). **7 palettes, lot 2026004.**                                                                                                                 | 1 relevé à `ACCEL_G` = 3,8 g (seuil 2,0 g) sur chacune des 7 palettes = **7 enregistrements**.                                             |

Hors anomalies, les valeurs restent dans les plages nominales, contrôlées sur le fichier produit : `TEMP_AMB` 15,0–25,0 °C avec variation saisonnière (minimum en janvier, maximum en juillet), `HR_AMB` 35–60 %, `ACCEL_G` 0,0–0,3 g au repos et 0,3–0,9 g en roulage. Pendant un trajet, température et humidité sont constantes, conformément au §7.

# 9\. Seuils capteurs (`T_REGLAGES`)

Un jeu de réglages identique est poussé sur les **350** palettes :

| Colonne              | Valeur | Commentaire                                                                  |
| -------------------- | ------ | ---------------------------------------------------------------------------- |
| `FREQ_MESURE_CL`     | 360    | 6 h, en minutes                                                              |
| `Seuil_TEMPAMB_CL`   | 40     | Maximum, °C — commun à tous les produits                                     |
| Température minimale | 5 °C   | **Aucune colonne dédiée dans** `T_REGLAGES` — convention à arbitrer (§13)    |
| `Seuil_HRAMB_CL`     | 80     | Maximum, % HR                                                                |
| `Seuil_AccelG_CL`    | 2.0    | g                                                                            |
| `Seuil_MASSE_CL`     | 100    | kg, tolérance max d'augmentation du poids par rapport à la mesure précédente |

# 10\. Traduction dans le schéma `cpaltracker`

## 10.1 Ordre d'insertion

| \# | Table                 | Contenu                                                                                                                 | Lignes        | État                   |
| -- | --------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------- | ---------------------- |
| 1  | `T_CLIENTS`           | Innopharm (`ClientID` 3), Cust01 (`ClientID` 4)                                                                         | 2             | schéma attendu         |
| 2  | `T_CONTRATS`          | MAIN Innopharm + STD Cust01 (dates incluses dans le MAIN)                                                               | 2             | schéma attendu         |
| 3  | `T_DEVICE`            | `cpaldev001` → `cpaldev350`, `STATUT` = `SIMU`                                                                          | 350           | schéma attendu         |
| 4  | `T_CONTRATS_DEVICES`  | Flotte ↔ MAIN (350) + les 110 palettes passées chez Cust01 ↔ STD                                                        | 460           | schéma attendu         |
| 5  | `T_DEVICE_HIST`       | `Créé` au 01/10/2026 (350) + un transfert aller-retour **par palette** Cust01 (110 × 2)                                 | 570           | schéma attendu         |
| 6  | `T_REGLAGES`          | Seuils §9                                                                                                               | 350           | schéma attendu         |
| 7  | `T_DDLOT`             | 57 dossiers de lot, `CreatedBy` = 4, `ClientID` = 3, `Gross_Weight` = `Net_Weight` = 20000, `Number_Of_Products` = 4000 | 57            | schéma attendu         |
| 8  | `T_DDLOT_DEVICE_HIST` | Accrochages palette ↔ lot, du remplissage à la palette vide — un par cycle                                              | 1 140         | schéma attendu         |
| 9  | `T_DATA`              | Mesures — `LOAD DATA LOCAL INFILE`                                                                                      | **1 401 155** | **généré et contrôlé** |

> `cPAL_Lot` est généré par la base (`FN_GEN_CPAL_LOT()`, 22 caractères aléatoires) et toute valeur fournie est ignorée. Les codes doivent donc être relus par `(ClientID, Cust_Lot_Number)` après l'étape 7, avant de construire les accrochages de l'étape 8.
> 
> Dans `T_DATA`, ne jamais insérer `RECORD_TIME`, `DIFF_UTC_TTN_SEC` ni `DIFF_TTN_RECORD_SEC` : ce sont des colonnes générées. On fournit `timestamp_utc`, `received_at_ttn`, `ChipID`, `YEAR`, `MONTH`, `DAY`, `HOUR`, `MIN`, `SEC`, `LONGITUDE`, `LAT`, `ACCEL_G`, `TEMP_MPU`, `TEMP_AMB`, `HR_AMB`, `MASSE`, dans cet ordre.
> 
> `T_DEVICE` doit être chargée **avant** `T_DATA` : la contrainte `T_DATA_ibfk_1` rejette tout `ChipID` inconnu.

`TEMP_MPU` est fixée à **4.00** sur toutes les lignes (décision du 24/09/2026).

Palettes distinctes passées chez chaque client sur l'horizon : Cust01 110, Cust02 137, Cust03 170, Cust04 138, Cust05 113, Cust06 105, Cust07 59. Seul Cust01 porte un contrat STD, donc seules ses 110 palettes apparaissent une seconde fois dans `T_CONTRATS_DEVICES`.

## 10.2 Évolution de schéma requise

Un lot de 20 000 kg ne tient pas dans `decimal(6,2)`, dont le maximum est 9 999,99. Deux modifications sont nécessaires **avant** l'étape 7 :

    ALTER TABLE T_DDLOT
      MODIFY `Gross_Weight` decimal(8,2) NOT NULL COMMENT 'kg',
      MODIFY `Net_Weight`   decimal(8,2) NOT NULL COMMENT 'kg';
    
    ALTER TABLE T_DDLOT
      DROP CONSTRAINT chk_ddlot_gross,
      DROP CONSTRAINT chk_ddlot_net,
      ADD  CONSTRAINT chk_ddlot_gross CHECK (`Gross_Weight` BETWEEN 0 AND 30000),
      ADD  CONSTRAINT chk_ddlot_net   CHECK (`Net_Weight`   BETWEEN 0 AND 30000);

S'y ajoute l'ajout du champ texte `Product_Name` dans `T_DDLOT`, demandé au §0.

# 11\. Volumétrie

| Paramètre            | Valeur                                                                             |
| -------------------- | ---------------------------------------------------------------------------------- |
| Horizon simulé       | 01/10/2026 → 31/05/2029, soit **974 jours**                                        |
| Flotte               | **350 palettes**, mesurées 4 fois par jour au repos                                |
| Relevés au repos     | **1 360 980**                                                                      |
| Relevés en transport | **40 175**                                                                         |
| **Total** `T_DATA`   | **1 401 155 lignes — 165 Mo en CSV, 29 Mo compressé, ≈ 345 Mo en base avec index** |

Le passage de 1 000 à 350 palettes divise la volumétrie par **2,8** à horizon comparable, sans rien retirer aux quatre objectifs : les 47 à 159 palettes vides présentes en permanence à Lyon suffisent à démontrer l'objectif n°4.

## 11.1 Contrôles passés sur le fichier produit

  - **Réconciliation des masses avec la simulation** : la masse totale présente dans `T_DATA` correspond au kilogramme près au stock simulé, sur six dates réparties de 2026 à 2029, avec les 350 palettes présentes chaque jour.

  - 350 `ChipID` distincts, tous dans `cpaldev001`–`cpaldev350` : la clé étrangère vers `T_DEVICE` passera.

  - Aucune ligne malformée sur les 16 colonnes ; `RECORD_TIME` ≤ `received_at_ttn` \< `timestamp_utc` sur la totalité des lignes.

  - Plages respectées : `MASSE` 0–1 000 kg, `TEMP_AMB` 15,0–25,0 °C hors excursion A1, `HR_AMB` 35–60 %, coordonnées dans l'emprise France.

  - Anomalies présentes et conformes : 112 relevés au-dessus de 40 °C sur 14 palettes, 7 relevés à 3,8 g sur 7 palettes.

# 12\. Réversibilité

Toutes les données de simulation sont identifiables par deux marqueurs : `ChipID LIKE 'cpaldev%'` et les `ClientID` 3 et 4. Le retrait complet tient dans un script unique, à exécuter dans cet ordre imposé par les clés étrangères — seules `T_DATA` et `T_CONTRATS_DEVICES` sont en `ON DELETE CASCADE` sur `T_DEVICE`.

    -- Sauvegarde préalable obligatoire :
    -- mysqldump -u root -p cpaltracker > cpaltracker_avant_purge_simu.sql
    
    SET @cid_inno := 3;   -- Innopharm
    SET @cid_c01  := 4;   -- Cust01
    
    DELETE FROM T_DDLOT_DEVICE_HIST WHERE ChipID LIKE 'cpaldev%';
    DELETE FROM T_DDLOT             WHERE ClientID IN (@cid_inno, @cid_c01);
    DELETE FROM T_REGLAGES          WHERE ChipID LIKE 'cpaldev%';
    DELETE FROM T_TRAITEMENT        WHERE ChipID LIKE 'cpaldev%';
    DELETE FROM T_DEVICE_HIST       WHERE ChipID LIKE 'cpaldev%';
    
    -- Cascade : supprime aussi T_DATA et T_CONTRATS_DEVICES
    DELETE FROM T_DEVICE            WHERE ChipID LIKE 'cpaldev%';
    
    DELETE FROM T_CONTRATS          WHERE ClientID IN (@cid_inno, @cid_c01);
    DELETE FROM T_CLIENTS           WHERE ClientID IN (@cid_inno, @cid_c01);

Aucun compte utilisateur n'est créé par la simulation : `T_USER` n'est pas touché, le compte de Frédéric Marteau est seulement référencé. La suppression ne laisse aucune trace hors compteurs `AUTO_INCREMENT`.

`T_DATA` contient déjà environ 14 000 lignes réelles (`AUTO_INCREMENT = 14002`). Elles sont préservées : le script de purge ne cible que les `ChipID LIKE 'cpaldev%'`.

# 13\. Points en attente

## 13.1 Tranché le 24/09/2026

>   - **Schéma de** `T_DATA` : obtenu, fichier généré et contrôlé.
> 
>   - **Tracés routiers** : les 8 itinéraires réels sont calculés et contrôlés (§7.2).
> 
>   - **Compte Frédéric Marteau** : `UserID = 4`. `ClientID` Innopharm = 3, Cust01 = 4.
> 
>   - **Tare palette** : pas de tare, `MASSE` mesure la charge nette (0 kg à vide).
> 
>   - `Number_Of_Products` : 4 000 unités de 5 kg par lot de 20 t.
> 
>   - **Profil de l'anomalie A1** : excursion de 7 h (2 h de montée, 3 h de plateau, 2 h de retour) dans un trajet de 7 h 52.
> 
>   - `TEMP_MPU` : constante à 4.00.

## 13.2 Reste à fournir ou à trancher

> 1.  **Schéma des 8 autres tables.** Un `SHOW CREATE TABLE` sur `T_CLIENTS`, `T_CONTRATS`, `T_DEVICE`, `T_CONTRATS_DEVICES`, `T_DEVICE_HIST`, `T_REGLAGES`, `T_DDLOT` et `T_DDLOT_DEVICE_HIST` permet de produire les \~2 930 lignes restantes.
> 
> 2.  **Seuil de température minimale.** `T_REGLAGES` n'a qu'une colonne `Seuil_TEMPAMB_CL`. Convention à choisir : stocker `min;max` dans le même champ, ou ajouter une colonne.
> 
> 3.  **Zone de stockage CQ.** Coordonnées 43.482708 / 5.378727 à confirmer : l'atelier et la zone CQ sont distants d'environ 500 m.
> 
> 4.  **Prérequis base avant chargement** : les `ALTER TABLE` du §10.2, l'ajout de `Product_Name` et la table des anomalies du §0, et l'activation de `local_infile` côté serveur et client pour le `LOAD DATA LOCAL INFILE`.

# 14\. Documents de référence

  - **Tableau de bord interactif**, comparant les 3 scénarios de seuil dont celui chargé en base : voir la page fille « Use Case Pharma — Simulation des flux de palettes : objectifs et hypothèses ».

  - **Feuille de paramètres v7** : [c-PAL Simu Pharma — Paramètres v7](https://docs.google.com/spreadsheets/d/1kcEzEGtCWHwbsVkN1bZbqVL__WD2n2C_9ol5Owkg7X4/edit), dans le [dossier Drive du Use Case](https://drive.google.com/drive/folders/1G2mxDV5ZTm9gJU9_vUWz1bnsTlNG0ZI6). **À corriger :** la ligne `flotte_totale` y vaut encore 1000, elle doit passer à 350.

  - **Présentation courte du Use Case** : `cPAL_UseCase_Pharma.pptx`, même dossier Drive.

  - **Fichier de chargement** : `T_DATA_use_case_pharma.csv.gz` et `charger_T_DATA.sql`.
