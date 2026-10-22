# Schéma DPT : révision B (21/10/2026)

Toutes les modifications ont été faites dans `half-bridge-inverter-dpt.kicad_sch`. Le PCB n'a pas été touché : il se met à jour avec **F8**. La feuille passe en **A3** pour faire de la place, et les nouveaux blocs sont reliés par **labels globaux**.

## 1. Erreurs corrigées dans le schéma existant

| # | Problème | Correction | Pourquoi c'est important |
|---|---|---|---|
| 1 | Le **−** des 5 condensateurs de bus n'était relié qu'à la bobine (J3) et à C2. Il n'allait pas à la source de Q2. | Nouveau net **DC−** : condensateurs, entrée bus J4, boucle de courant J5. | Le bus n'était pas refermé : aucun courant ne pouvait circuler dans la demi-branche. |
| 2 | **C2** (100 nF) était **en série** entre DC− et la source de Q2. | C2 devient un condensateur HF **entre VDC et DC−** (1210, 630 V). | En série, il bloquait le courant continu. En parallèle, il fournit le courant pendant les quelques ns de la commutation. |
| 3 | La bobine (J3) était entre SW et DC−. | **J3 : SW ↔ VDC**, en parallèle de Q1. | Pour tester Q2 (bas), la bobine doit être en roue libre à travers la diode de corps de Q1. |
| 4 | La **source puissance de Q2** était sur le GND logique (BNC, STM32). | Net **SRC_L** → boucle de mesure J5 → **DC−**. Le GND logique est maintenant isolé de la puissance. | Sinon, l'isolation des drivers ne sert à rien, et la masse du STM32 recevrait du bruit de commutation à quelques dizaines de V/ns. |
| 5 | Les deux drivers partageaient les **mêmes VDD / VEE**. | Nets séparés **VDD_H / VEE_H** (référencés à COM_H) et **VDD_L / VEE_L** (référencés à COM_L). | Le driver haut flotte avec SW (0 → 200 V). Il lui faut sa propre alimentation isolée. |
| 6 | **DESAT** (broche 2) était reliée à COM. | Protection DESAT ajoutée (voir 2.4). | Reliée à COM, la protection contre les courts-circuits était désactivée. |
| 7 | Les **LED** D1 à D4 étaient montées à l'envers : anode côté broche, cathode vers VCC. | LED retournées : VCC → R → anode, cathode → RDY/FLT. Résistances de 2 kΩ réduites à **1 kΩ**. | RDY et FLT sont des sorties à drain ouvert, elles tirent vers le bas. Dans l'ancien sens, les LED n'auraient jamais pu s'allumer. |
| 8 | Pull-up de RST/EN de 10 kΩ, alors que la broche a 50 kΩ de pull-down interne (soit 2,75 V). | **4,7 kΩ** (R3, R6), ce qui donne 3,0 V. | Meilleure marge sur le niveau haut (la datasheet recommande 5 kΩ). |
| 9 | Condensateurs de découplage de **100 µF en 0805** (C3, C4, C5, C10). | **10 µF 50 V X7R en 1206**, plus 100 nF au plus près des broches. | 100 µF n'existe pas en 0805 avec la tenue en tension nécessaire. La datasheet demande au moins 10 µF sur VDD et sur VEE. |
| 10 | Résistances de grille en **traversant axial** (12,7 mm). | **1206 CMS.** | Les pattes longues ajoutent de l'inductance dans la boucle de grille. |
| 11 | Broche 9 (GND) du symbole UCC21750 déclarée en `power_out`. | Passée en `power_in` dans le symbole embarqué. | Deux `power_out` sur le même net donnent une erreur ERC. **À corriger aussi dans `Created_Symboles`.** |
| 12 | Références invalides. | 4_MOSFET3/4 → **Q1/Q2**, U3 → **J3**, B32776… → **C11 à C15** (le numéro de pièce est dans le champ `MPN`). | |

## 2. Ajouts

### 2.1 Alimentations
- **J6** : entrée 12 V (bornier 5,08 mm).
- **U4 AMS1117-3.3** : 12 V → 3,3 V pour le côté primaire des drivers. Le courant total est d'environ 15 mA, soit environ 0,13 W dissipé. C'est **3,3 V** pour que les niveaux logiques soient les mêmes que ceux du STM32.
- **PS1 et PS2, Murata MGJ2D121805BSC** : 12 V → +18 / −5 V, isolation 5,2 kV, CMTI > 200 kV/µs. Un module par driver.
- **Précharge** : R9/R11 de 2,2 kΩ (8,2 mA sur +18 V) et R10/R12 de 1 kΩ (5 mA sur −5 V). Le module a besoin d'au moins **10 % de sa charge nominale**. À vide, sa sortie peut monter jusqu'à environ 1,25 × 18 V = 22,5 V, ce qui dépasse le V_GS max de 22,6 V. En DPT, le driver consomme seulement environ 4 mA, donc la précharge est indispensable.
- **C21/C24** : 4,7 µF en entrée de chaque module, la valeur du filtre recommandé par Murata.

### 2.2 Interface STM32 (J7, connecteur 1×8)
`GND, FLT_H, RDY_H, EN_H, FLT_L, RDY_L, EN_L, GND`
- Pull-ups de **4,7 kΩ** sur FLT et RDY (R13 à R16).
- **R17/R18 de 10 kΩ** en pull-down sur IN+ : si le câble BNC est débranché, la grille reste bloquée.
- Le STM32 peut **couper les drivers** en tirant EN à 0, et **lire un défaut** sur FLT, à câbler en interruption.

### 2.3 Bus et mesure
- **J4** : entrée bus 0 à 200 V, avec ton empreinte M4 `Connector_2_P8`.
- **J5** : boucle de fil dans la source de Q2, pour passer une sonde **Rogowski**. Plus tard, tu pourras la remplacer par un shunt coaxial.
- **C29 à C31** : 3 × 100 nF 630 V en plus de C2. Ils forment la deuxième famille de condensateurs, à placer **au ras de Q1/Q2**.
- **TP1 à TP4** : points de test pour SW, DC−, GATE_L et COM_L. Ils servent à mesurer V_DS avec la sonde HT, et V_GS entre la grille et la source Kelvin.

### 2.4 Protection DESAT (pour chaque driver)
`DESAT → 1 kΩ → diode US1M (1000 V) → drain`, avec un condensateur de masquage de 47 pF C0G entre DESAT et COM.
- **Temps de masquage** : 47 pF × 9,15 V / 500 µA ≈ 0,86 µs, plus 200 ns internes, soit environ 1,06 µs.
- **Seuil** : 9,15 V. Avec 10 A dans 32 mΩ, V_DS ≈ 0,3 à 0,5 V, donc pas de déclenchement en fonctionnement normal. En cas de court-circuit, la protection coupe en environ 1 µs.
- Si tu observes des déclenchements intempestifs à la mise sous tension, augmente C27/C28 à 68 ou 100 pF.

## 3. À faire de ton côté

1. **Corrige l'empreinte du MOSFET** (`NTH4L032N065M3S_MOSFET:NTH4LN032N065M3S`). D'après le dessin CASE 340CW de la datasheet, les entraxes sont **5,08 / 2,79 / 2,54 mm**. Ton empreinte utilise 5,08 / 2,54 / 2,54 mm. Dans l'éditeur d'empreintes : **pad 3 → X = 2,79 mm, pad 4 → X = 5,33 mm**, puis déplace aussi les textes « S1 » et « G ».
2. **Corrige ton symbole** `Created_Symboles:UCC21750DW` : broche 9 en `Power input`. Si tu ne le fais pas, la prochaine mise à jour depuis la bibliothèque réintroduira l'erreur.
3. **Lance l'ERC.** Avertissements attendus : « symbole différent de la bibliothèque » pour Device:D, Conn_01x02/08, TestPoint et AMS1117. C'est normal : j'ai dessiné ces symboles à la main avec les mêmes numéros de broches que les officiels. Fais **Outils → Mettre à jour les symboles depuis la bibliothèque** pour récupérer les dessins officiels.
4. **Range la mise en page.** Les nouveaux composants sont en A3, à droite et en bas. Quelques labels (FLT_H, EN_H, PWM_H…) sont posés directement sur des broches, dans des zones serrées : décale-les si tu veux.
5. **F8** pour mettre à jour le PCB.
6. **Vérifie le dessin mécanique du MGJ2D121805BSC** (page 21 de la datasheet). L'empreinte KiCad est celle de la famille MGJ2 SIP7 (broches 1, 2, 5, 6, 7 au pas de 2,54 mm), mais le boîtier de la version « B » peut être légèrement plus grand. Note aussi que cette variante est indiquée « preliminary » chez Murata. En repli : **MGJ2D121505SC** (+15 / −5 V, même brochage). R_DS(on) sera un peu plus élevé.

## 4. Conseils de placement pour le PCB

- **Boucle de puissance** : C29 à C31 et C2 → Q1 → Q2 → DC−. Elle doit être la plus petite possible, avec VDC et DC− sur deux couches internes adjacentes et superposées.
- **Zone d'isolation** : garde une bande sans cuivre sous chaque UCC21750 et sous chaque MGJ2, entre le côté primaire (GND) et le côté secondaire (COM_x).
- **PS1, C10, C3 et les condensateurs DESAT** sont sur le potentiel de **SW** : place-les du côté de Q1, loin du GND logique.
- **Découplage** : les 100 nF (C22, C23, C25, C26, C19, C20) se placent **au ras des broches** des drivers, pas à côté des modules.
