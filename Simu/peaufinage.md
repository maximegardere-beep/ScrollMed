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

---

# Plan v2 : Simu, équipe, physiologie dynamique, polytraumatisé (2026-11-05)

Choix faits par QCM le 2026-11-05. **Statut : en attente de lancement**, lot par lot, dans l'ordre ci-dessous. Retenus : L7 (respirateur, réglable sur ordinateur seulement), L8 et L13 (gaz du sang et ionogramme dynamiques), L11 (capnographie). Les autres pistes de L1 à L13 restent en attente.

## Décisions

- **V1 Renommage** : le mode s'appelle **« Simu »** dans toute l'interface (onglet ✚, titres, toasts, accueil, réglages), et le dossier `Simu/` devient `Simu/` (CLAUDE.md, validateurs, `scenarios_a_creer.md`, `reserve/`). Les clés internes ne changent pas (`STATE.dechoc`, préfixes `dc*`, `id` de scénarios, d'actions, de terrains), pour garder l'historique des joueurs.
- **V2 Univers** : la page de lancement s'ouvre sur 3 univers, **🚑 Déchocage**, **💉 Anesthésie** et **🛏️ Réa**, qui filtrent à la fois le patient aléatoire et la liste des scénarios fixes.
  - Anesthésie = induction + peropératoire + **SSPI**.
  - Réa : vide pour l'instant (« bientôt »), à remplir par de nouvelles trames (V7).
  - Univers présélectionné d'après le domaine du feed (`STATE.domaine`), puis mémorisé (`DC_PREFS.univers`).
- **V3 L'équipe pose des questions** sur un oubli, avec une probabilité qui dépend du niveau :
  - gestes visés : un **socle** propre à chaque univers (au déchoc : scope, VVP, O2, glycémie ; au bloc : monitorage, check-list…), plus les **indispensables** de l'étape ;
  - un geste fait après une question de l'équipe vaut **0 point**, ni bonus ni malus ; une question ignorée laisse l'oubli compter normalement ;
  - au débriefing : « fait sur rappel de l'IDE ».
- **V4 Capnographie signifiante** (L11).
- **V5 Gaz du sang et ionogramme dynamiques** (L8 + L13). Le scénario fixe les valeurs de départ, puis le moteur les fait évoluer. Les textes fixes actuels restent en repli.
- **V6 Respirateur réglable** (L7), sur **navigateur d'ordinateur seulement**. Le téléphone garde les tuiles actuelles, qui deviennent des préréglages.
- **V7 Polytraumatisé procédural** :
  - architecture : **lésions combinables** (catalogue de lésions composées par le moteur) ;
  - prise en charge préhospitalière **tirée sans erreur** : des gestes déjà faits ou non, jamais mal faits ;
  - après la stabilisation, une **décision qui termine la partie** : TDM corps entier (étape jouée, puis orientation), bloc de damage control ou radiologie interventionnelle (fin).

## Lots

- [x] **S0 Simu et univers** (V1, V2, livré le 2026-11-05)
  - `git mv Dechocage Simu`. Le code lit `Simu/` et se replie sur `Dechocage/` s'il est absent, le temps que les apps en cache se mettent à jour. CLAUDE.md, validateurs et textes d'aide passent à « Simu ».
  - Contextes : la valeur `rea` actuelle (2 trames de SSPI) devient `sspi`, et `rea` est libéré pour la vraie réanimation. Aucune règle `dans` de `terrains.json` ne vise `rea` aujourd'hui, donc pas de migration de contenu au-delà des 2 trames.
  - `DC_UNIVERS = { dechoc: [dechoc], anesth: [induction, perop, sspi], rea: [rea] }`. Le sous-filtre par contexte est retiré.
  - Les scénarios fixes sans `terrains` sont rangés via une clé `univers` du scénario (défaut `dechoc`).
- [x] **S1 Questions de l'équipe** (V3, livré le 2026-11-05 ; socle et phrases dans `actions.json`, une question de socle n'est posée que si l'étape note le geste `recommande`)
  - Contenu, dans `actions.json` :
    - `socle: ["dechoc"]` : univers où le geste fait partie du socle ;
    - `rappel` : la phrase de l'IDE, par ex. « Je le scope ? », « On lui pose une voie ? », « Je fais une glycémie ? ».
  - Un geste du socle n'est demandé que si l'étape le note `recommande` (pas d'oxygène réclamé dans l'acidocétose) ; les notes des trames ne changent pas. Rien à réécrire dans les scénarios.
  - Déclenchement :
    - au-delà de 50 % du chrono de l'étape (en temps fixé, du temps patient de l'étape) ;
    - probabilité de **70 % en facile, 35 % en moyen, 0 % en difficile** (scénario fixe : moyen) ;
    - au plus 1 question par geste et par partie, 2 par étape, aucune pendant un ACR.
  - Interface : bulle datée de l'IDE sous le scope, avec « Oui, vas-y » (lance le geste, 0 point) et « Pas maintenant ». Cette bulle sert de base à L4 (équipe qui parle).
- [ ] **S5 Polytraumatisé procédural** (V7)
  - `Simu/lesions.json`, une lésion par entrée :
    - `id`, `label`, `region` (crâne, rachis, thorax, abdomen, bassin, membres), `exclut` ;
    - fragment de vignette, `constantes` et `pente` additives ;
    - `resultats` et `schemas` par examen (E-FAST, RP, radio du bassin, TDM), fusionnés par le moteur : paramètres des schémas réunis, texte des régions saines « sans anomalie » ;
    - `actions` dues (format d'étape : note, `alt`, `why`), `letal_si_manque`, contre-indications, `tags`.
  - Lésions de la v1 :
    - PNO compressif, hémothorax massif, tamponnade, volet costal et contusion ;
    - rupture splénique, lacération hépatique ;
    - fracture instable du bassin, fracture du fémur ;
    - plaie de membre avec saignement artériel ;
    - TC grave avec hématome extradural (repris de `reserve/sc_tc_grave.json`), lésion médullaire cervicale avec choc neurogénique.
  - Trame `sc_polytrauma.json` :
    - `lesions` : 1-2 en facile, 2-3 en moyen, 3-4 en difficile, au plus une par région ;
    - terrain tiré comme d'habitude, par ex. anticoagulé ;
    - `prehosp` : gestes SMUR tirés avec une probabilité (VVP, ceinture pelvienne, garrot, IOT, acide tranexamique, collier), déjà faits à l'arrivée, sans points pour le joueur, et rappelés dans la **transmission SMUR** (fiche avant le chrono) ;
    - e1 ABCDE et stabilisation, puis décision avec la fenêtre de transfert existante : TDM corps entier (étape e2 jouée, lésions révélées, puis orientation), bloc de damage control ou radiologie interventionnelle (fin).
  - Débriefing par lésion : gestes dus, faits, oubliés.
  - Validateur étendu à `lesions.json`.
- [ ] **S2 Capnographie signifiante** (V4, L11) : forme du tracé selon l'état, sans contenu à écrire.
  - Aileron de requin : complication bronchospasme, trame asthme, ou action de classe `bronchospasme`.
  - Plateau normal.
  - Chute brutale : ACR, EP, débit cardiaque effondré.
  - Remontée franche au RACS.
  - Pendant la RCP : EtCO2 entre 10 et 20 selon la qualité du massage.
  - Courbe plate si la sonde est retirée (tuile future « extubation accidentelle », V7 Réa).
- [ ] **S3 Gaz du sang et ionogramme dynamiques** (V5)
  - Données :
    - `bio` du scénario : valeurs de départ pH, PaCO2, HCO3⁻, PaO2, lactates, K⁺, Na⁺, glycémie ;
    - `bio` sur les actions : effet et délai, par ex. insuline −0,6 mmol/L de K⁺ en 30 min, ou KCl, bicarbonate, salbutamol, G10 ;
    - normes et format d'affichage dans `Simu/biologie.json`, pour que le moteur reste sans donnée médicale.
  - Moteur :
    - PaCO2 selon la ventilation minute (FR spontanée ou respirateur, S4) ;
    - HCO3⁻ selon les traitements ;
    - pH par Henderson-Hasselbalch ;
    - lactates qui montent pendant le temps passé sous PAM 65 et redescendent après correction ;
    - PaO2 déduite de la SpO2 et de la FiO2.
  - Les examens `gds`, `iono`, `glycemie` et `lactates` génèrent leur texte, valeurs anormales en `**…**`. Un résultat écrit du scénario s'ajoute pour ce qui n'est pas calculé (trou anionique, cétonémie).
  - Conditions `si` (résultats, déclencheurs) étendues aux variables `bio`, par ex. une aggravation hypokaliémique à `K < 3`. L'acidocétose et l'hyperkaliémie sont converties en premier.
- [ ] **S4 Respirateur réglable** (V6, L7)
  - Sur ordinateur, panneau « Respirateur » après l'intubation : Vt (affiché en mL/kg de poids idéal), PEP, FiO2, FR, I:E.
  - Il affiche Pplat, Pcrête, auto-PEEP et ventilation minute.
  - Il faut une taille au patient : `identite.taille` sur la trame, tirée selon le sexe.
  - `ventilation` du scénario : compliance, résistance, shunt, PEP utile.
  - Effets :
    - SpO2 selon FiO2, PEP et shunt ;
    - PaCO2 selon la ventilation minute (S3) ;
    - PAS qui chute avec la PEP et l'auto-PEEP chez le patient précharge-dépendant ;
    - collapsus de l'asthmatique quand la FR est trop haute.
  - Les tuiles actuelles (`vm_protectrice`, `vm_asthme`, `hypervent`) deviennent des préréglages, seule commande sur téléphone.
  - Débriefing : cibles du scénario (Vt 6 à 8 mL/kg, Pplat < 30, FR basse dans l'asthme).
  - L10 (variation de pression pulsée) pourra s'appuyer dessus plus tard.
- [ ] **S6 Trames de réa** (remplissent l'univers Réa, sujets à choisir) : extubation accidentelle, choc septique en cours de séjour, SDRA qui s'aggrave (s'appuie sur S4), hémorragie digestive du cirrhotique, désadaptation et PNO sous ventilation.

Ordre (revu le 2026-11-05) : S1 et S0 (livrés), puis S5 (polytraumatisé, avancé avant la physiologie), puis S2 et S3, puis S4 (dépend de S3 pour la PaCO2), puis S6.
