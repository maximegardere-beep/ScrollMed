# Peaufinage du mode Déchoc : décisions et plan

Pistes choisies par QCM le 2026-10-08. Cocher (`- [x]`) quand c'est livré.

## Décisions

- **P1** Interface :
  - toasts déplacés hors des boutons (sous le scope) ;
  - anneau de progression sur la tuile pendant l'appui maintenu, vibration à la validation ;
  - rangée « Récents » en tête des tuiles ;
  - recherche tolérante aux synonymes et abréviations (IOT, NAD, CGR, TXA…), via une clé `alias` dans `actions.json` ;
  - débriefing qui commence par un résumé : les 3 erreurs les plus coûteuses, les pièges évités ; étapes repliées en dessous.
- **P2** Réalisme :
  - **résultats différés** : délai de rendu en minutes patient (`delai` sur l'action du catalogue, surchargeable par le scénario). Le résultat apparaît avec une notification quand l'horloge l'atteint ;
  - **équipe qui parle** : bulles datées (IDE, chirurgien, réanimateur…) écrites dans les scénarios et les complications (`messages` d'une étape, déclenchés à l'entrée, après un délai ou sur une action). Le moteur reste sans donnée médicale ;
  - **ambiance par contexte** : couleur d'accent, icône et libellé du lieu (déchoc, bloc, salle de réveil), selon `terrains.contexte` ou une clé `lieu` du scénario.
- **P3** Illustrations :
  - **silhouette du patient en SVG**, générée par le moteur d'après l'état (pâleur si PAS basse, cyanose si SpO2 basse, marbrures, sonde d'intubation si capnographie, perfusions et pousse-seringues posés, massage pendant l'ACR) ;
  - **schémas d'examens en SVG** (ETT, E-FAST, radio de thorax…), choisis par le scénario dans une petite bibliothèque de schémas (`schema` sur un résultat), sans image externe.
- Écartés pour l'instant : événements imprévus, tests versés dans le dépôt, ECG 12 dérivations, images réelles, rejouer le même patient, frise des actions.

## Ordre de livraison proposé

- [ ] **L1** Interface : toasts, anneau d'appui, récents, synonymes, résumé du débriefing.
- [ ] **L2** Silhouette du patient et ambiance par contexte.
- [ ] **L3** Résultats différés.
- [ ] **L4** Équipe qui parle : moteur, puis messages ajoutés aux 8 trames et aux 5 complications.
- [ ] **L5** Schémas d'examens SVG, branchés sur les scénarios existants.
