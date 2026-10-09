# Use Case Pharma — simulation des flux de palettes : objectifs et hypothèses

> **Source :** page Confluence « Use Case Pharma — Simulation des flux de palettes :
> objectifs et hypothèses 2026-09-23 »
> ([88604673](https://c-pal.atlassian.net/wiki/pages/viewpage.action?pageId=88604673), version 2),
> importée le 2026-10-09 ([KAN-104](https://c-pal.atlassian.net/browse/KAN-104)).
> Ce fichier est désormais la référence ; la page Confluence n'est plus mise à jour.
>
> Le modèle décrit ici est implémenté dans
> [`cpaltracker.web/tools/demo-data/pharma`](../../cpaltracker.web/tools/demo-data/pharma/README.md)
> (`simulate.py`, `run_scenarios.py`). Jeu de données qui en découle :
> [pharma-use-case-dataset.md](pharma-use-case-dataset.md).

> **Objet :** ce document explique le Use Case Pharma (Innopharm) tel qu'il est simulé : ses objectifs, ses hypothèses principales, puis ses hypothèses détaillées. Il accompagne deux livrables vivants — un **tableau de bord interactif** et une **Google Sheet de paramètres** — référencés en fin de page.
> 
> **Note de version :** le modèle a évolué depuis le cadrage initial de la page parente. Il porte désormais sur **10 produits** (au lieu de 4), une **seule vague planifiée** suivie de vagues déclenchées par le stock, une **volatilité de consommation active**, et un horizon prolongé au **31/05/2029**. La page parente reste la référence pour la traduction dans le schéma `cpaltracker`.

# 1\. Objectifs

## 1.1 Ce que la démonstration doit prouver

Le détenteur du contrat MAIN (Innopharm) doit pouvoir, à partir des seules données des palettes connectées :

1.  faire l'**inventaire** d'un lot de production ou d'un produit à une date donnée ;

2.  identifier les **anomalies** de température, humidité, poids et accélération, sur l'ensemble du contrat, d'un lot ou d'un produit ;

3.  visualiser l'**évolution de la consommation** chez ses clients, par produit ou par lot ;

4.  identifier les **palettes vides** et leur localisation.

## 1.2 La question supply chain posée au modèle

Au-delà de la traçabilité, la simulation répond à une question de pilotage :

> **Quel est l'effet de l'anticipation des clients sur le stock de l'entrepôt de Lyon ?**

Concrètement : si chaque client repasse commande plus tôt (en conservant un stock de sécurité au lieu d'attendre l'épuisement), que devient le stock que l'entrepôt central doit porter ? Trois niveaux d'anticipation sont simulés — seuil de réapprovisionnement à **0, 1 et 2 tonnes** — **à demande client identique**, de façon à isoler l'effet du seul pilotage.

# 2\. Hypothèses principales

## 2.1 La boucle physique

Le flux suit une boucle unique, identique pour chaque produit :

**Production (Aix) → Contrôle qualité → Entrepôt Lyon → Livraison client → Consommation → Palette vide → Retour Lyon → Réemploi**

Chaque palette c-PAL est identifiée et suivie individuellement à chacune de ces étapes : c'est cette donnée qui alimente le tableau de bord et la base `cpaltracker`.

## 2.2 Les cinq hypothèses structurantes

| \# | Hypothèse                               | Principe                                                                                                                                                                                                                                                            |
| -- | --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1  | **Production en flux tiré**             | Seule la vague 1 est planifiée (un lot par produit). Les vagues suivantes ne sont lancées que lorsqu'un produit vient réellement à manquer — aucun stock tampon décidé à l'avance.                                                                                  |
| 2  | **Lots fixes de 20 tonnes**             | Toute campagne produit un lot de 20 palettes × 1 000 kg, quelle que soit la quantité manquante.                                                                                                                                                                     |
| 3  | **Demande client fixe**                 | La consommation de chaque ligne suit un échéancier figé, indépendant du scénario, et se consomme **en série** (un client finit une livraison avant d'entamer la suivante). La demande totale est donc identique dans les trois scénarios : **≈ 1 110 t**.           |
| 4  | **Pilotage à deux échelons**            | Lyon n'est pas un stock d'avance mais un **stock de sécurité piloté** : la production est déclenchée sur le stock d'échelon (ce qui est vivant à Lyon **et** chez les clients), de sorte que plus les clients détiennent de stock, moins Lyon en fabrique d'avance. |
| 5  | **Seuil de réapprovisionnement client** | Une ligne de commande repasse commande dès que son stock cumulé passe sous le seuil du scénario (0 / 1 / 2 t). C'est le paramètre testé.                                                                                                                            |

## 2.3 Ce que montrent les résultats

| Seuil de réappro. | Demande client | **Stock mensuel moyen à Lyon** | Écart-type mensuel | CV   | Stock moyen chez les clients |
| ----------------- | -------------- | ------------------------------ | ------------------ | ---- | ---------------------------- |
| 0 t (épuisement)  | 1 124 t        | **32 646 kg**                  | 11 205             | 34 % | 163 034 kg                   |
| 1 t               | 1 055 t        | **31 269 kg**                  | 13 802             | 44 % | 163 343 kg                   |
| 2 t               | 1 101 t        | **27 053 kg**                  | 11 642             | 43 % | 166 976 kg                   |

**Lecture.** À demande égale, plus les clients anticipent, plus le stock moyen de l'entrepôt de Lyon baisse (**−17 % entre 0 et 2 t**) et se reporte vers l'aval : le stock de sécurité descend d'un échelon. La **variabilité**, elle, ne baisse pas : elle est portée par la taille de lot fixe de 20 t, qui fait arriver le stock par à-coups indépendamment du pilotage. C'est un enseignement en soi — l'anticipation agit sur le **niveau** de stock, la taille de lot sur sa **régularité**.

Aucune rupture structurelle n'est observée en cœur d'horizon ; les palettes non servies se concentrent en fin de simulation (effet de bord de l'horizon).

# 3\. Hypothèses détaillées

## 3.1 Périmètre et paramètres physiques

| Paramètre                             | Valeur                                                                                                         |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| Horizon simulé                        | 01/10/2026 → 31/05/2029 (32 mois)                                                                              |
| Produits                              | 10 (Produit 1 → Produit 10), un lot initial chacun                                                             |
| Clients                               | 7 (Cust01 → Cust07)                                                                                            |
| Taille de lot                         | 20 t = 20 palettes × 1 000 kg                                                                                  |
| Flotte                                | 1 000 palettes instrumentées (`cpaldev001` → `cpaldev1000`) ; ≈ 300 à 320 effectivement chargées sur l'horizon |
| Contrôle qualité                      | 6 jours (libération = fin de production + 6 j, arrivée Lyon le même jour)                                      |
| Livraison Lyon → client               | J+1, reportée au lundi si elle tombe un dimanche                                                               |
| Péremption                            | 12 mois **à compter du début de production** ; un lot périmé à la date de livraison n'est pas expédié          |
| Palette vide                          | enlèvement 5 j après la fin de consommation, retour à Lyon en 1 j                                              |
| Transfert des vides Lyon → Aix        | 1 j (la production démarre à l'arrivée)                                                                        |
| Inactivité minimale entre deux vagues | 7 j                                                                                                            |
| Couverture de sécurité amont (Lyon)   | 60 jours de demande                                                                                            |
| Graine aléatoire                      | 2026 (résultats reproductibles)                                                                                |

## 3.2 Profils de consommation

Chaque ligne de commande (un client, un produit) consomme selon un profil :

| Code    | Dynamique                                                                        |
| ------- | -------------------------------------------------------------------------------- |
| `LinN`  | décroissance linéaire régulière du stock sur N mois — consommation régulière     |
| `FDscN` | stock constant puis chute brutale à 0 au N-ième mois — consommation par campagne |

Profils utilisés : `Lin3`, `Lin6`, `Lin9`, `Lin12`, `FDsc2`, `FDsc4`, `FDsc6`, `FDsc8`. Chaque produit est commandé par au moins 2 clients, et chaque client mélange plusieurs dynamiques.

**Volatilité.** La durée réelle de consommation ajoute à la durée nominale un décalage tiré par la simulation :

`durée (mois) = N + volatilité × choix`, avec `choix ∈ {−1, 0, +1}` et `volatilité ∈ {2, 3}` selon le profil.

Exemple : un profil `Lin12` de volatilité 3 avec un choix `−1` est consommé en 9 mois au lieu de 12. Le choix est fixé pour les deux premières commandes de chaque ligne, puis tiré au hasard (graine fixe). Garde-fou : la durée ne descend jamais sous 1 mois.

## 3.3 Règles de réapprovisionnement et de production

**Échelon aval — le client.** Une ligne repasse commande dès que son **stock cumulé** — la somme des palettes qui lui sont affectées, celles en transit comprises — passe sous le seuil du scénario. À seuil 0, elle recommande à stock nul.

**Échelon amont — Lyon.** Une nouvelle vague de production est lancée dès que le **stock d'échelon** d'un produit — palettes vivantes à Lyon **et** chez les clients, non encore consommées, plus la production en cours — ne couvre plus la demande attendue sur (délai d'obtention du lot + 60 jours de couverture). Le stock détenu par les clients comptant dans l'échelon, plus les clients anticipent, moins Lyon fabrique d'avance.

**Affectation.** Les palettes sont affectées aux commandes en **FIFO** sur les lots du produit, du plus ancien au plus récent.

**Numérotation des lots.** Format `{année}{séquence sur 3 chiffres}` — `2026001`, `2026002`… puis `2027001` l'année suivante. Les lots déclenchés par le stock poursuivent la séquence de l'année sans réutiliser un numéro déjà pris.

## 3.4 Indicateurs

Le stock de l'entrepôt de Lyon est mesuré en **moyenne et écart-type mensuels** (variabilité mois à mois), et non en moyenne journalière : c'est la maille de pilotage pertinente pour un responsable supply chain. Tous les stocks sont des photos de fin de journée.

Deux contrôles de cohérence sont vérifiés à chaque exécution : le **bilan matière** (produit = stock + consommé) et la **conservation de la flotte** (1 000 palettes comptabilisées chaque jour).

# 4\. Documents de référence

## 4.1 Tableau de bord interactif

**→** [**Ouvrir le tableau de bord**](https://claude.ai/artifact/VHe7A3qGnxvS8znc7Toiwy)

Page web autonome, à jour de la dernière simulation. Elle permet de :

  - basculer entre les **trois scénarios** de seuil (0 / 1 / 2 t) via le sélecteur en haut de page — tous les graphiques et tableaux suivent ;

  - comparer en permanence le **stock mensuel moyen à Lyon** des trois scénarios sur un même graphique ;

  - filtrer les stocks **par lieu, par produit, par lot** et suivre les **palettes vides**, en cliquant sur les éléments de légende ;

  - consulter le détail des **alertes, vagues, lots et commandes** dans les onglets du bas.

*Accès : le lien est privé ; il s'ouvre pour son propriétaire et pour les personnes à qui il a été explicitement partagé (menu Partager de la page).*

## 4.2 Google Sheet des paramètres

**→** [**Ouvrir la feuille de paramètres**](https://docs.google.com/spreadsheets/d/1kcEzEGtCWHwbsVkN1bZbqVL__WD2n2C_9ol5Owkg7X4/edit)

Source de vérité des entrées de la simulation, en trois sections sur une seule feuille :

  - `## PARAMETRES` — tous les paramètres du §3.1 ;

  - `## CAMPAGNES` — la vague 1 planifiée (un lot par produit) ;

  - `## COMMANDES` — les 34 lignes de commande : client, produit, quantité, profil, volatilité, choix V1/V2.

Elle est rangée dans le dossier Drive du Use Case, avec la présentation : **→** [**Dossier Drive « Use Case Pharma »**](https://drive.google.com/drive/folders/1G2mxDV5ZTm9gJU9_vUWz1bnsTlNG0ZI6)

## 4.3 Chaîne de production des livrables

La simulation est un programme Python (`simulate.py`, `rapport.py`, `entrees_defaut.py`, `run_scenarios.py`). Une exécution lit la feuille de paramètres, joue les trois scénarios, et produit le classeur de résultats détaillé ainsi que le tableau de bord. Modifier un paramètre dans la feuille et relancer suffit à régénérer l'ensemble.
