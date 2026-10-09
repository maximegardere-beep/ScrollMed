# Peaufinage du mode Déchoc : décisions et plan

Pistes choisies par QCM le 2026-10-08. **Statut : en attente, à lancer plus tard** (sur demande de l'utilisateur). Cocher (`- [x]`) quand c'est livré.

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
- Non retenus pour l'instant (réserve d'idées) :
  - événements imprévus (famille, panne de voie, poche bouchée) ;
  - tests Playwright versés dans le dépôt (`tests/`) ;
  - ECG 12 dérivations dessiné à partir du code de rythme ;
  - images d'examens réelles sous licence libre ;
  - rejouer le même patient depuis le débriefing ; frise des actions sur la courbe.

## Ordre de livraison proposé

- [ ] **L1** Interface : toasts, anneau d'appui, récents, synonymes (livrés le 2026-10-09) ; reste le résumé du débriefing.
- [x] **L0** Ergonomie iPhone en paysage (2026-10-09) : barre de titre masquée en jeu, barre du scope avec ⏸, titre, chrono à barre et « ✓ n » en appui maintenu ; chiffres face aux courbes ; colonne d'icônes des catégories et 🔍 ; onglets Gestes / Dossier avec pastille de résultats ; silhouette remplacée par un résumé sous la vignette (dessin via 🧍 et à la pause).
- [ ] **L2** Silhouette du patient (livrée) et ambiance par contexte (à faire).
- [ ] **L3** Résultats différés.
- [ ] **L4** Équipe qui parle : moteur, puis messages ajoutés aux 8 trames et aux 5 complications.
- [x] **L5** Schémas d'examens SVG, branchés sur les scénarios existants (avec TDM et angioscanner).
