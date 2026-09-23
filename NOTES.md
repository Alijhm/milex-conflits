# Notes de projet : conflits voisins et dépenses militaires

## 1. Question et hypothèses (figées le JJ/MM/2026, avant tout modèle)
Question : quand un conflit armé éclate ou s'intensifie près d'un pays, ses dépenses militaires augmentent-elles ensuite, de combien, avec quel délai ?
- H1 : les dépenses réelles augmentent dans les 1 à 3 ans après un conflit proche.
- H2 : l'effet est plus fort pour un conflit interétatique que pour un conflit interne.
- H3 : l'effet diminue avec la distance.
Cadrage : association temporelle, pas preuve de causalité.

## 2. Sources
- **SIPRI Military Expenditure Database**, édition 2026, fichier `SIPRI-Milex-data-1949-2025_v1.2.xlsx`, onglet « Constant (2024) US$ » (millions de USD, prix constants 2024). Téléchargé le 20/09/2026. DOI : https://doi.org/10.55163/CQGC9685. Sources ouvertes uniquement. 174 pays, 8 435 valeurs renseignées (avant retrait des entités disparues).
- **UCDP/PRIO Armed Conflict Dataset v26.1**, fichier `UcdpPrioConflict_v26_1.csv`. 2 816 lignes conflit-année, 28 colonnes. Téléchargé le 20/09/2026.
- **CShapes 2.0** (Schvitz, Girardin, Cederman, Bara, Zhukov, Cunningham, Weidmann, 2022, *Journal of Conflict Resolution*, « Mapping the International System, 1886-2019 »), fichier `CShapes-2.0.geojson`. Couverture : 1886 au 31/12/2019. Téléchargé le 21/09/2026. Utilisé pour les coordonnées des capitales et le calcul des distances (formule haversine), avec un instantané figé au 31/12/2019 appliqué à toute la période 1992-2025 (voir limites, section 5).
- Outil de conversion des noms de pays : paquet Python `country_converter` (colonne GWcode), corrigé à la main sur plusieurs cas (section 4).

## 3. Décisions de méthode

### Construction du panel
D1. Période : 1992-2025. Raison : éviter les dissolutions (URSS, Yougoslavie, Tchécoslovaquie) et bénéficier d'une meilleure couverture SIPRI. Coût : on perd la guerre froide.
D2. Codes pays : système Gleditsch-Ward (celui d'UCDP). 167 pays retenus au final, aucun code en double. Exclus : Union européenne, RDA, URSS, Yougoslavie, Tchécoslovaquie, « Yemen, North » (tous hors période ou pas des pays), Seychelles (absente de CShapes, aucun conflit). Corrections manuelles : Yémen = 678, Serbie = 340 ; après contrôle croisé avec CShapes : Tchéquie = 316, Viet Nam = 816 (le codage initial de `country_converter` désignait d'anciens États).
D3. Lieu des conflits : `gwno_loc` donne directement le théâtre pour les conflits internes. Pour les 17 conflits interétatiques depuis 1992, ce champ liste les belligérants et non le lieu des combats (ex. Irak 2003 liste États-Unis, Royaume-Uni, Australie) : codage manuel du territoire de combat, avec des règles par année quand le théâtre change en cours de conflit (ex. Russie-Ukraine : Ukraine seule en 2022, Ukraine + Russie à partir de 2023, après les raids frontaliers de Belgorod et l'offensive de Koursk).
D4. Belligérants exclus de l'échantillon exposé : tous les pays présents dans `gwno_a`, `gwno_b`, `gwno_a_2nd`, `gwno_b_2nd` (implication « toutes causes », `own_conflict`) ou seulement en tant que belligérant principal/théâtre (implication « directe », `own_primary`) — voir D12 pour la règle retenue.
D5. Ukraine : le conflit de 2014 est codé par UCDP comme conflit interne internationalisé (types 3/4), le conflit de 2022 comme interétatique. Agrégation au niveau pays-année (intensité maximale, indicateur interétatique, nombre de conflits) pour éviter de compter deux fois la même année.
D6. Contrôle de cohérence : 37 pays touchés par un conflit en 2022 (dont Ukraine, Israël, Syrie) ; France, Allemagne, États-Unis, Russie absents de cette liste, comme attendu.
D7. Tableau pays-année des conflits : 1 050 lignes (1992-2025).
D8. Distances : CShapes 2.0, instantané des États indépendants au 31/12/2019, distance de capitale à capitale par formule haversine (rayon terrestre 6 371,0088 km). Table statique appliquée à toute la période 1992-2025.

### Exposition et chocs
D9. Exposition : un pays est « exposé » l'année *t* s'il existe au moins un conflit dont le théâtre est à ≤ 1 000 km (capitale à capitale) et dont il n'est ni belligérant ni théâtre. Variable expliquée principale : variation annuelle en log des dépenses en USD constants (2024). Un « choc » (définition initiale, D9) est le premier passage à l'état exposé après au moins 3 années consécutives sans exposition.
D10. Choc « guerre » (ajout) : même principe, mais restreint aux conflits d'intensité 2 (≥ 1 000 morts dans l'année). Raison : la définition D9 capte les *débuts* d'exposition (ex. Ukraine 2014) mais pas les *escalades* pour les pays déjà exposés depuis plusieurs années (ex. la Pologne, exposée en continu depuis 2014, ne repasse jamais par « 3 ans sans » avant 2022, donc 2022 n'est pas un choc D9 pour elle). Le choc « guerre » devient la définition retenue pour l'analyse principale.
D12. Règle d'inclusion (implication) : révisée après diagnostic. La règle « aucune implication, y compris soutien extérieur » (`clean_any`) écartait à tort des pays comme la Pologne, la Lettonie ou la Roumanie, exclus uniquement parce qu'ils avaient soutenu la coalition en Afghanistan entre 2011 et 2014 — sans rapport avec le choc étudié. Règle principale retenue : exclure seulement l'implication *directe* (belligérant principal ou théâtre) de *t*-3 à *t* (`clean_prim`). La règle `clean_any` est conservée comme test de robustesse.
D15. Valeurs extrêmes de la croissance : 128 observations sur 4 827 (2,7 %) dépassent 50 % en log, surtout des séries instables (Angola, Tadjikistan 1995, RDC 1998, Soudan du Sud 2016). Ukraine 2022 (+1,79 en log, environ ×6) est un vrai basculement et non une erreur, mais hors échantillon (belligérante). Winsorisation aux percentiles 1 % et 99 % pour l'analyse principale ; croissance brute en robustesse.

### Analyse événementielle et résultats
D16-D17. Event study : fenêtre *k* = -3 à +3 autour du choc, croissance winsorisée exprimée en écart à la moyenne des pays non exposés et non impliqués directement la même année. Agrégation principale : un épisode (guerre × année) = un vote (moyenne des pays de l'épisode, puis moyenne entre épisodes), intervalle de confiance à 95 % par bootstrap sur les épisodes (1 000 tirages, graine 42). Résultat principal (79 chocs jusqu'en 2022, 28 épisodes) : aucun effet détectable, tous les intervalles de confiance incluent zéro pour *k* = -3 à +3.
D19. Le tableau épisode par épisode (exploratoire, lu après le résultat principal) montre que les trois plus gros épisodes ont un écart positif après/avant le choc (Irak 2003 +9,9 points, Ukraine 2014 +11,0, Ukraine 2022 +3,9), alors que les petits épisodes sont mélangés (Nigeria 2013 -11,2, Tchéquie/Tchétchénie 1995 -3,7, etc.). C'est ce qui explique que le signe du résultat global change selon la pondération (voir robustesse).
D21. Robustesse (sept variantes : agrégation par épisode/pays, avec/sans winsorisation, regroupement par année, sans la Biélorussie, règle `clean_any`, choc D9) : écarts allant de -2,4 à +2,1 points selon la variante, aucun intervalle n'exclut zéro.
D22. Statut des hypothèses : H1 non soutenue (aucun effet détectable dans l'analyse principale). H2 (interétatique) et H3 (distance) non testées, faute de puissance statistique sur des sous-groupes de moins d'une dizaine d'épisodes chacun.

### Figures complémentaires (ajoutées après les résultats principaux)
D26. Comparaison régionale (exploratoire, non publiée comme figure, mais citée dans l'article comme hypothèse) : en comparant les pays exposés non plus à l'ensemble du monde mais aux seuls pays européens non exposés et non impliqués directement la même année, l'écart après/avant se réduit fortement sans disparaître : +4,4 points en 2014 et +0,8 en 2022 (contre respectivement +11,0 et +3,9 points face au reste du monde). Région définie par le continent du pays exposé (via `country_converter`), et non par le lieu de la guerre : les épisodes libyens (2011, 2016) n'impliquent qu'un seul pays européen (Malte), et l'épisode turc de 2016 implique la Bulgarie, la Grèce et la Macédoine du Nord — ces cas n'ont donc pas leur place dans un récit centré sur la Russie/l'Ukraine. Décision : la figure à deux panneaux (Europe / reste du monde) est retirée de l'article (titre trompeur, groupes hétérogènes, deux des six épisodes européens ne comptent qu'un pays) ; seul le chiffre de comparaison régionale est cité en texte, comme hypothèse non tranchée.
D27/D30. Nuage distance/croissance 2022 (n = 115) : Spearman = -0,085 [IC 95 % : -0,273 ; 0,111]. Deux valeurs extrêmes repérées avant la version finale : Soudan du Sud (+161,6 %, base de 33,7 M$) et Haïti (-57,5 %, base de 19,6 M$), toutes deux des effets de petite base qui faisaient passer la pente d'une régression linéaire de -0,22 à -0,82 point par 1 000 km selon qu'on les inclut. Décision : abandon de la droite de régression au profit de médianes par tranche de distance (< 1 000, 1 000-2 000, 2 000-4 000, > 4 000 km) avec IC bootstrap et effectifs affichés. Résultat : médianes de -1,6, +2,1, -3,4 et -2,0 points, tous les IC incluant zéro ; aucun gradient de distance visible en 2022.
D28/D32. Carte d'exposition (Ukraine, 2022) : projection azimutale équidistante centrée sur Kiev (`+proj=aeqd`), seule projection qui rend le cercle de 1 000 km exact dans toutes les directions. Distances de contrôle recalculées avec les coordonnées CShapes : Serbie 977 km (exposée), Slovaquie 1 003 km, Bulgarie 1 022 km, Autriche 1 054 km, Estonie 1 066 km (non exposées) — le seuil de 1 000 km est arbitraire et tranche des cas à quelques dizaines de kilomètres d'écart. Conservée comme schéma explicatif de la règle d'exposition, sans valeur de preuve. La Biélorussie y figure comme « exposée » alors qu'elle a servi de base arrière à l'invasion (UCDP ne la code pas comme belligérante).

## 4. Erreurs rencontrées et corrections
- Codage `country_converter` : codes erronés ou ambigus pour le Yémen (680 au lieu de 678) et la Serbie, corrigés par croisement avec UCDP puis avec CShapes.
- Piège avec `not_found=None` : le paquet garde le nom d'origine au lieu de renvoyer une valeur vide, ce qui rendait la détection des pays non reconnus trompeuse.
- `gwno_loc` contient parfois plusieurs codes séparés par une virgule (159 lignes) : éclatement (`explode`) nécessaire avant toute jointure.
- Recherche du Vietnam par le motif texte « Viet » dans les noms CShapes : capte aussi « Soviet » (Russia (Soviet Union)), d'où deux résultats et une correction manquée dans un premier temps. Corrigé en fixant directement le code (816, Hanoï).
- Règle d'inclusion « aucune implication » (`clean_any`) initialement retenue comme règle principale : écartait à tort des pays exposés au conflit étudié uniquement à cause d'un soutien extérieur sans rapport (voir D12).

## 5. Limites identifiées pour l'article
- Les frontières et capitales de CShapes s'arrêtent au 31/12/2019 : appliquées telles quelles à 1992-2025 (pas de mouvement de capitale ni de changement de frontière après cette date, ex. Crimée).
- Peu de conflits interétatiques purs depuis 1992 (17), et H2/H3 non testées faute de puissance.
- Le conflit 16099 (États-Unis/Royaume-Uni contre les Houthis) est largement maritime, mal capté par une distance terrestre entre capitales.
- Incertitude propre aux estimations SIPRI, plus forte pour les pays en conflit interne (Angola, RDC) ou peu transparents.
- La distance entre capitales sous-estime l'exposition réelle des très grands pays (Russie, Chine, États-Unis) et de leurs voisins frontaliers (ex. Finlande vis-à-vis de Kiev).
- UCDP ne code pas la Biélorussie comme belligérante en 2022, bien qu'elle ait servi de base arrière à l'invasion.
- Tendance de hausse déjà perceptible à *k* = -1 dans certaines variantes : fragilise toute lecture causale stricte.
- Puissance statistique limitée : impossible d'exclure un effet modeste (2 à 4 points par an), seul un effet important (plus de 5 points) est exclu par les données.
- Non-indépendance des événements : un même conflit touche souvent plusieurs pays voisins la même année (ex. les huit mêmes pays d'Europe de l'Est pour les épisodes Ukraine 2014 et 2022) ; erreurs groupées par pays et vote par épisode pour en tenir compte partiellement, mais l'indépendance reste imparfaite.
- Seuil de 1 000 km sensible à quelques dizaines de kilomètres près (voir D28) : la carte en donne un exemple concret.

## 6. Étapes réalisées et pistes pour de futures études

**Étapes réalisées :**
- Panel pays-année reconstitué et validé sur 1992-2025 (167 pays).
- Mesures de distance, filtres d'intensité et fenêtres de contrôle (`clean_prim`) appliqués.
- Étude d'événements exécutée et figures générées (*k* = -3 à *k* = +3), avec sept variantes de robustesse.
- Article rédigé, présentant l'absence d'effet détecté dans l'analyse principale et les limites de l'exercice.

**Pistes pour prolonger l'étude (non réalisées ici) :**
- **Alliances contre distance** : l'article avance, à titre d'hypothèse non tranchée, que la hausse observée en 2014 et 2022 pourrait devoir autant aux logiques d'alliance (objectif OTAN des 2 % du PIB, sommet de Newport 2014) qu'à la seule proximité géographique. Ce n'est pas démontré ici : il faudrait une régression avec un terme d'interaction (distance × appartenance à une alliance) pour le tester proprement.
- **Distance aux frontières** plutôt qu'entre capitales, pour les grands pays (nécessiterait d'extrapoler les frontières de CShapes au-delà de 2019).
- **Ratio dépenses/PIB** en variable expliquée alternative (onglet « Share of GDP » de SIPRI), pour limiter les biais liés à l'inflation ou à la taille de l'économie.
- **Fenêtres complètes post-2022** : les chocs de 2023-2025 n'ont pas de fenêtre *k* = +2/+3 complète ; à revisiter quand SIPRI et UCDP couvriront 2026-2028.

## 7. Historique Git
- Un seul commit réalisé à ce jour, après l'obtention des résultats de robustesse (message : « État du projet après les résultats de robustesse »).
- Aucun commit n'a été fait avant l'analyse événementielle. La primauté des définitions figées (D9, D10, D12, D15, D16 notamment) sur les résultats ne repose donc pas sur l'historique Git, mais sur les dates portées dans ce document au moment de leur rédaction, avant l'exécution des cellules correspondantes.
- Prochain commit prévu : ajout des scripts finaux (`export.py`), du `README.md` et de ce fichier.
- Règle suivie tout au long du projet : ne jamais antidater un commit, ne jamais réécrire l'historique.

## 8. Sources externes citées dans l'article
- OTAN, Déclaration du Sommet du pays de Galles (2014) : https://www.nato.int/cps/en/natohq/events_112136.htm
- CRS R43698, « NATO's Wales Summit: Outcomes and Key Challenges » (2014) : quatre alliés à 2 % du PIB en 2013.
- SIPRI, communiqué du 27/04/2026 : https://www.sipri.org/media/press-release/2026/global-military-spending-surges-2025
- SIPRI Fact Sheet, avril 2026 : 2 887 Md$ de dépenses mondiales en 2025 (+2,9 % en termes réels, onzième année de hausse), Europe +14 % à 864 Md$.
- OTAN, rapport annuel du secrétaire général (Mark Rutte), 26 mars 2026 : les 32 membres ont atteint ou dépassé l'objectif de 2 % du PIB en 2025, une première depuis l'engagement de 2014.
