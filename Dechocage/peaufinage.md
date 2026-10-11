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
  - rejouer le même patient depuis le débriefing.

## Ordre de livraison proposé

- [ ] **L1** Interface : toasts, anneau d'appui, récents, synonymes (livrés le 2026-10-09) ; reste le résumé du débriefing.
- [x] **L0** Ergonomie iPhone en paysage (2026-10-09) : barre de titre masquée en jeu, barre du scope avec ⏸, titre, chrono à barre et « ✓ n » en appui maintenu ; chiffres face aux courbes ; colonne d'icônes des catégories et 🔍 ; onglets Gestes / Dossier avec pastille de résultats ; silhouette remplacée par un résumé sous la vignette (dessin via 🧍 et à la pause).
- [ ] **L2** Silhouette du patient (livrée) et ambiance par contexte (à faire).
- [ ] **L3** Résultats différés.
- [ ] **L4** Équipe qui parle : moteur, puis messages ajoutés aux 8 trames et aux 5 complications.
- [x] **L5** Schémas d'examens SVG, branchés sur les scénarios existants (avec TDM et angioscanner).
- [ ] **L6** Frise comparée au chemin expert (retenue le 2026-10-11) : au débriefing, la chronologie du joueur face au chemin idéal (gestes `indispensable` de chaque étape du chemin par défaut, objectifs `delai`) sur une même échelle de temps patient ; repères des gestes posés sur la courbe des constantes (toucher un repère = son `why`) ; écarts en couleur (fait à temps, en retard, oublié, contre-indiqué). Calculé par le moteur depuis les scénarios, rien à écrire dans le contenu.
- [ ] **L7** Respirateur réglable (retenu le 2026-10-11) : après l'intubation, réglage du Vt (rapporté au poids idéal), de la PEP, de la FiO2 et de la FR, avec effet sur SpO2, EtCO2 et PA. Pièges jouables : auto-PEEP et collapsus de l'asthmatique si FR trop haute, hypoxémie du SDRA sans PEP, hypotension à l'intubation de l'hypovolémique. Les tuiles « Respirateur » actuelles (stratégies toutes faites) deviennent des préréglages.
- [ ] **L8** Gaz du sang dynamiques (retenu le 2026-10-11) : pH, PaCO2, PaO2 et lactates calculés depuis l'état de la partie (PaCO2 selon FR, intubation et réglages du respirateur ; lactates selon le temps passé en hypotension ; PaO2 selon SpO2 et FiO2) au lieu d'un texte figé ; le contrôle reflète la qualité de la réanimation. Le scénario garde la main sur le point de départ et les anomalies propres (acidocétose, hyperkaliémie).
- [ ] **L9** Effet selon l'état du patient (retenu le 2026-10-11) : l'effet hémodynamique d'un hypnotique ou d'une amine dépend de l'état (précharge-dépendance, tamponnade, choc) : induction modérée chez le normovolémique, collapsus chez l'hypovolémique ; kétamine mieux tolérée que le propofol dans le choc. S'appuie sur la précharge-dépendance existante (`hemodynamique`), par classe (`hypnotique`, `vasopresseur`).
- [ ] **L10** Variation de pression pulsée (retenue le 2026-10-11) : sous ventilation contrôlée, la courbe de PA invasive oscille avec la respiration tant que le patient est précharge-dépendant, puis s'aplatit après remplissage. Valeur de VPP affichable. Lié à L7 (FR et Vt du respirateur).
- [ ] **L11** Capnographie signifiante (retenue le 2026-10-11) : forme du tracé selon l'état : aileron de requin du bronchospasme, plateau normal, chute brutale (embolie, ACR, extubation accidentelle), remontée franche au RACS ; en RCP, EtCO2 selon la qualité du massage.
- [ ] **L12** Diurèse horaire (retenue le 2026-10-11) : constante affichée après pose d'une sonde urinaire, effondrée dans le choc, qui remonte avec la réanimation ; alerte en cas de rhabdomyolyse ou après produit de contraste.
- [ ] **L13** Ionogramme et glycémie dynamiques (retenus le 2026-10-11, prolongent L8) : kaliémie, glycémie et natrémie évoluent avec les traitements et leur délai : l'insuline fait baisser la kaliémie (acidocétose : hypokaliémie et troubles du rythme sans supplémentation), calcium, salbutamol et insuline-glucose dans l'hyperkaliémie.
