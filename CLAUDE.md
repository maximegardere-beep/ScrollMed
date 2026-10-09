# ScrollMed

Application de révision en anesthésie-réanimation au format « reels » : on fait défiler des cartes courtes (recommandations, posologies, quiz…).

- `index.html` : toute l'app (HTML + CSS + JS, sans build ni dépendance). Contient aussi quelques cartes de départ codées en dur.
- `Recos/` : **le registre de cartes**. L'app liste ce dossier via l'API GitHub, sur la branche `main` par défaut, et charge chaque fichier `.json` à chaque lancement. Une carte poussée sur `main` est donc en ligne tout de suite.
- `Recos/cartes_complementaires_a_creer.md` : liste des sujets restant à traiter. Cocher (`- [x]`) les sujets couverts par un nouveau lot.
- `Dechocage/` : contenu du **mode Déchoc** (jeu de simulation de déchocage, onglet ✚ de la barre du bas) : catalogue d'actions, scénarios, terrains et complications du patient procédural, synchronisés comme `Recos/`. Voir la section « Mode Déchoc ».
- `Dechocage/scenarios_a_creer.md` : scénarios Déchoc restant à écrire, avec leur trame. Cocher (`- [x]`) ceux qui sont poussés.
- `Dechocage/patient_procedural.md` : conception du patient aléatoire (décisions, écarts à corriger). `Dechocage/peaufinage.md` : plan de peaufinage du mode Déchoc (interface, réalisme, illustrations), en attente.

## Règle n°1 : génération de lots de cartes → push direct, pas de PR

Quand on te demande de générer un lot de cartes (ou de corriger, fusionner ou retirer des cartes dans `Recos/`) :

1. **N'ouvre pas de pull request.**
2. Commite et pousse **directement sur `main`**, puisque c'est la branche que lit l'app. Si la session impose une branche de travail `claude/...`, pousse aussi le commit sur `main` (`git push origin HEAD:main`) après avoir intégré `origin/main` (`git pull --rebase origin main` sur ta branche si besoin).
3. Ce message vaut autorisation explicite de pousser sur `main` pour tout ce qui touche au contenu de `Recos/`.

Les changements de code dans `index.html` suivent le même chemin (push direct sur `main`), sauf si l'utilisateur demande expressément une PR.

## Format d'une carte

Chaque fichier `Recos/lotN.json` est un **tableau JSON** de cartes :

```json
{
  "module": "Ventilation & oxygénation",
  "hook": "Titre accrocheur, court, qui porte le message",
  "body": "Le message principal en 1 à 3 phrases.",
  "detail": "Nuance, mécanisme, piège ou exception (affiché en dépliant).",
  "source": "Société savante / étude + année",
  "grade": "Grade 1+",
  "tags": ["SDRA", "ventilation"]
}
```

- Obligatoires : `module`, `hook`, `body`, `source`, `tags`. Fortement recommandé : `detail`. Optionnel : `grade` (uniquement s'il est réel), `quiz`.
- Exception, cartes de culture générale (`Culture & histoire`, `Éthique & droit`) : `source` peut être omise quand la carte relève de la culture générale (étymologie, anecdote, réflexion éthique) et qu'aucune référence précise n'existe. Ne jamais inventer une source pour combler le champ ; si une référence réelle existe (loi, article, date historique), la mettre.
- `"quiz": true` : le `hook` sert de question et `body` / `detail` sont cachés jusqu'au clic (utilisé pour `Posologies` et `Idées reçues`).
- `Urgences & déchocage` : `body` en 2 à 5 gestes numérotés `①`, `②`… séparés par `\n`.
- `Idées reçues` : `hook` = l'idée reçue entre « », `body` commence par « Vrai : » ou « Faux : ».
- La source doit être datée : l'app affiche une pastille « +5 ans » sur les sources anciennes (sauf modules intemporels : `Culture & histoire`, `Éthique & droit`, `Grandes études`, `Physiologie`, `Idées reçues`).

### Modules existants (ne pas en créer sans raison)

Posologies · Échographie · Hémostase & transfusion · Sepsis & infectiologie · Réanimation médicale · Soins quotidiens en réa · Culture & histoire · Hémodynamique & cardiologie · Ventilation & oxygénation · Anesthésie peropératoire · Urgences & déchocage · Grandes études · Idées reçues · Physiologie · Éthique & droit · Voies aériennes · Neuro-réanimation · Arrêt cardiaque · ALR & douleur · Obstétrique · Évaluation préopératoire · Pédiatrie

Le nom du module doit être recopié **à l'identique** (accents, `&`) : c'est lui qui regroupe les cartes, colore le fond et alimente le radar de compétences. Un nouveau module apparaît seul sur le radar dès 4 cartes (sinon il tombe dans « Autres ») ; lui donner un libellé court dans `RADAR_LABELS` (`index.html`).

## Process de génération d'un lot

1. **Lire l'existant avant d'écrire** : charger toutes les cartes de `Recos/*.json` (et les cartes de départ de `index.html`) pour éviter les doublons.
2. **Pas de doublon** : ni même `module` + `hook` (clé de dédoublonnage de l'app, insensible à la casse), ni doublon de fond (même message avec un autre titre). En cas de recouvrement, enrichir ou corriger la carte existante plutôt qu'en ajouter une.
3. **Exactitude** : uniquement des recommandations et des doses vérifiables, conformes aux données actuelles (RFE SFAR/SRLF, ESAIC, SSC, ERC…). Ne jamais inventer de source, d'année ou de grade.
4. **Style** : français, phrases courtes, lisibles sur un écran d'iPhone. Le `hook` doit faire passer le message à lui seul.
5. **Fichier** : nouveau fichier `Recos/lotN.json`, N = dernier numéro + 1. Ne pas renommer les fichiers existants.
6. **Valider** avant de commiter (voir ci-dessous), puis commit + push direct sur `main`.

### Validation (à lancer avant chaque commit)

```bash
python3 - <<'EOF'
import json, glob, collections
REQ = {"module", "hook", "body", "source", "tags"}
SANS_SOURCE_OK = {"Culture & histoire", "Éthique & droit"}   # culture générale
keys = collections.Counter()
for f in sorted(glob.glob("Recos/*.json")):
    cards = json.load(open(f, encoding="utf-8"))   # plante si le JSON est invalide
    assert isinstance(cards, list), f"{f} : doit être un tableau"
    for i, c in enumerate(cards):
        miss = REQ - c.keys() - ({"source"} if c.get("module") in SANS_SOURCE_OK else set())
        assert not miss, f"{f}[{i}] : champs manquants {miss}"
        assert isinstance(c["tags"], list), f"{f}[{i}] : tags doit être une liste"
        keys[(c["module"] + "::" + c["hook"]).lower().strip()] += 1
dups = [k for k, n in keys.items() if n > 1]
print(f"{sum(keys.values())} cartes, {len(keys)} uniques")
assert not dups, f"doublons module::hook : {dups}"
EOF
```

## Mode Déchoc

Jeu de simulation : un patient tiré au hasard arrive au déchocage, ses constantes évoluent en temps réel et le joueur choisit ses actions dans un arbre de catégories, valide chaque étape, puis lit un débriefing (détail action par action, courbe des constantes). Le moteur (`index.html`) ne contient **aucune donnée médicale** : tout est dans `Dechocage/`.

Publication : le contenu de `Dechocage/` suit la règle n°1 (push direct sur `main`, pas de PR).

### Temps et constantes

- **Rythme** choisi par le joueur avant chaque station (`DC_MODES`, `index.html`) ; le scénario n'a rien à prévoir :
  - `reel` (temps réel, difficulté élevée) : chrono doublé (un `chrono` de 90 donne 3 min réelles), **1 s réelle = 5 s patient** ;
  - `ralenti` : chrono quadruplé, 1 s réelle = 2,5 s patient ;
  - `fixe` (temps fixé) : pas de chrono, rien ne bouge entre deux gestes ; la dégradation (`pente`) du temps patient consommé par les actions de l'étape s'applique d'un coup à la validation.
  Dans tous les cas, une étape représente `chrono` × 10 s patient. Le temps pour récupérer un ACR passe de 90 s réelles (temps réel) à 135 s (ralenti) et 180 s (temps fixé).
- Chaque action consomme `duree` minutes patient : l'horloge avance d'autant et la dégradation s'applique d'un coup.
- `pente` : variation par minute patient, par constante (`FC`, `PAS`, `PAD`, `SpO2`, `FR`, `T`, `GCS`, `EtCO2`). Une variation de PAS entraîne la PAD de moitié si la PAD n'est pas précisée. Une fois le chrono de l'étape écoulé, les pentes sont doublées (sauf en temps fixé).
- `seuils` du scénario (défaut `{ "PAS": 50, "SpO2": 70 }`) : passer sous un seuil déclenche un ACR.

### Scope

- Courbes en balayage : ECG et pléthysmographie toujours ; PA invasive après une action `"moniteur": ["PA"]` (sinon PNI toutes les 3 min patient, ou à la demande en touchant la case PA) ; capnographie après une action `"moniteur": ["CO2"]` (intubation). `"moniteur": ["MCE"]` (massage) ajoute l'artefact de compressions et la capno de RCP. Ces valeurs se mettent dans le catalogue.
- Téléphone en paysage : chaque case de chiffres est en face de sa courbe (FC / ECG, SpO2 / pléthysmographie, PA, EtCO2 / capnographie ; sans PA invasive, la PNI garde une bande vide), FR, T et GCS en petites cases dessous (`dcRows()`, `index.html`). En portrait, ils restent dans la barre du bas.
- `ecg` : rythme affiché, parmi `sinus`, `fa`, `qrs_larges`, `st_plus`, `tv`, `fv`, `asystolie`, `aesp`. Sur le scénario (défaut `sinus`), sur une étape (à l'entrée), sur une action d'étape (ex. le calcium affine les QRS) ou sur `acr` (sinon déduit du texte de `rythme` : FV, TV, asystolie, sinon AESP). Tachycardie et bradycardie découlent de la FC.
- `EtCO2` : constante optionnelle (38 par défaut), abaissée par le bas débit.
- `alarmes` du scénario, optionnel : surcharge des seuils `[priorité moyenne, haute]`, défaut `{ "FC": { "bas": [50, 40], "haut": [120, 150] }, "PAS": { "bas": [90, 70] }, "SpO2": { "bas": [90, 85] } }`. L'alarme PA porte sur la valeur affichée (PNI ou invasive).
- Sons (coupés par défaut) : bip de pouls plus grave quand la SpO2 baisse, alarmes moyenne et haute façon IEC 60601-1-8, silence 2 min.
- ACR : alarme nommée d'après le tracé (ASYSTOLIE, FIBRILLATION VENTRICULAIRE, TACHYCARDIE VENTRICULAIRE, PAS DE POULS pour l'AESP), qui sonne même si un silence était en cours. Le bouton « Silence alarme » acquitte et lance un métronome de massage à 110/min (son + pulsation à l'écran), jusqu'au RACS.

### Catalogue : `Dechocage/actions.json`

Tableau d'actions, partagé par tous les scénarios :

```json
{ "id": "noradre", "label": "Noradrénaline IVSE", "path": ["Traitement", "Amines"], "duree": 3 }
```

- `path` : chemin dans l'arbre de tuiles (1er niveau : Airway / ventilation · Voies & monitorage · Bilan · Imagerie & écho · Traitement · Orientation · Diagnostic · RCP). Ajouter une action ou une sous-catégorie = ajouter une ligne, sans toucher au code.
- `examen: true` : l'action révèle un résultat (« Pas d'anomalie notable » par défaut) et peut être refaite pour un contrôle.
- `repetable: true` : geste refaisable (bolus de remplissage, CGR, adrénaline…).
- `type: "diagnostic"` : hypothèse diagnostique, sans durée ; un seul diagnostic actif à la fois.
- `voie` : voies fournies par un geste (`vvp` et `io` → `["IV"]`, `ktc` → `["IV", "VVC"]`). `requiert` : voie nécessaire (`"IV"` pour tout médicament ou soluté intraveineux). Sans elle, la tuile est verrouillée (🔒) et l'action ne se lance pas.
- `titration` : pousse-seringue titré automatiquement (noradrénaline). Une fois lancé, il tient la PAM à `cible_pam` : la dose monte quand la PA baisse (`gain` = mmHg de PAS par unité) et redescend vers `debut` quand la PA dépasse la cible. Elle est plafonnée par `max` selon la meilleure voie posée (ex. `{ "IV": 2, "VVC": 8 }` : plafond bas sur VVP, levé par la VVC). Au plafond, la PA rechute et le scope affiche « MAX ». Dose affichée dans le scope, dose maximale au débriefing.
- `transfert` : l'action fait quitter le déchoc (scanner, bloc, radiologie interventionnelle…). Avant de la lancer, une fenêtre évalue la stabilité du patient et propose « Partir maintenant » ou « Pas maintenant » (redemander plus tard) ; le chrono est arrêté pendant la décision. Format : `{ "vers": "au scanner", "min": { "PAM": 65, "SpO2": 90, "GCS": 9 }, "max": { "FC": 140 } }` (le critère GCS est levé si le patient est intubé ; une noradrénaline au plafond rend instable).
- Les actions se réalisent par **appui maintenu** (≈ 0,5 s) sur la tuile, ce qui évite les touchers accidentels. La tuile affiche la durée patient.
- La catégorie `RCP` n'apparaît que pendant un ACR.
- `classe` : classes pharmacologiques ou de geste (`betalactamine`, `remplissage`, `hypnotique`, `curare`, `curare_depolarisant`, `vasopresseur`, `adrenaline`, `betabloquant`, `antithrombotique`, `anticoagulant`, `aminoside`…), visées par les terrains du patient procédural. Une nouvelle action d'une classe existante est couverte d'office.
- Ne jamais renommer un `id` d'action (les scénarios s'y réfèrent) ni un `id` de scénario (l'historique des joueurs s'y réfère). Idem pour les `id` de terrains et de complications.

### Scénario : `Dechocage/sc_<nom>.json`

```json
{
  "id": "choc_septique_pna", "titre": "…", "source": "SSC 2021 ; …", "tags": ["sepsis"],
  "patient": "Femme, 68 ans, 70 kg",
  "constantes": { "FC": 124, "PAS": 82, "PAD": 44, "SpO2": 94, "FR": 26, "T": 39.2, "GCS": 14 },
  "seuils": { "PAS": 50, "SpO2": 70 },
  "resultats": { "lactates": "Lactates 4,6 mmol/L." },
  "etapes": [{
    "id": "e1", "titre": "Accueil", "vignette": "…", "chrono": 120,
    "pente": { "PAS": -1.2, "FC": 0.8 },
    "constantes": { },
    "diagnostic": ["dx_choc_septique"],
    "actions": {
      "atb_c3g_amika": { "note": "indispensable", "alt": "atb", "why": "…" },
      "noradre": { "note": "recommande", "why": "…", "effet": { "PAS": 18 }, "pente": { "PAS": 0.2 } },
      "isr_propofol": { "note": "contre_indique", "letal": "acr", "why": "…" },
      "lactates": { "note": "recommande", "refaire": true, "why": "…" }
    },
    "resultats": { "lactates": "Contrôle : 3,8 mmol/L." },
    "letal_si_manque": { "actions": ["calcium"], "issue": "acr", "why": "…" },
    "acr": { "rythme": "AESP", "requis": ["mce", "adre_acr"], "why": "…", "apres_racs": { "FC": 100, "PAS": 90 } },
    "suite": [ { "si_manque": [["atb_c3g_amika", "atb_carba"]], "vers": "e1_aggrav" }, { "vers": "e2" } ]
  }]
}
```

- **Notes** : `indispensable` (+3 si fait, −3 si oublié), `recommande` (+2), `debattu` (0), `inutile` (−1), `contre_indique` (−5). Une action absente de l'étape vaut `inutile`, sauf si une autre étape l'attend (elle est alors comptée comme anticipée). Diagnostic : le joueur peut en poser plusieurs ; +2 si au moins un fait partie de la liste `diagnostic` de l'étape (lister toutes les réponses acceptables : choc cardiogénique, SCA, OAP…), sans pénalité pour les autres. ACR récupéré : −5. Décès : 0/20.
- `why` est obligatoire : c'est le texte du débriefing. Il doit être exact et sourcé, comme une carte.
- `alt` : actions interchangeables (plusieurs antibiotiques acceptables). Une seule suffit pour satisfaire l'indispensable, et seule la première faite rapporte des points.
- `effet` : variation immédiate des constantes. `pente` sur une action : remplace la pente de l'étape pour ces constantes, jusqu'à la fin de la partie.
- `refaire: true` : l'action doit être refaite dans cette étape (contrôle), un passage antérieur ne compte pas. Sur une action non `repetable` déjà faite, la tuile est verrouillée : le passage antérieur compte. Un geste qu'une étape peut exiger de nouveau (cardioversion, appel à l'aide, arrêt du produit) doit donc être `repetable`.
- `resultats` : texte révélé par un examen. Celui de l'étape l'emporte sur celui du scénario. Entourer chaque valeur anormale de `**…**` : elle s'affiche en rouge (ex. `"**K⁺ 7,9 mmol/L** · Na 140 mmol/L"`).
- `schemas` : schéma SVG dessiné par le moteur sous un résultat, mêmes clés que `resultats` et **au même niveau** (scénario, étape, terrain, traitement) : `{ "type": "ett", …paramètres }`, `legende` optionnelle. Le schéma suit le texte retenu : si un terrain remplace le texte sans schéma, rien n'est dessiné. Il ne doit rien montrer que le texte ne dise. Types : `ett`, `efast`, `echo_pleuro`, `echo_veineuse`, `echo_renale`, `rp`, `rx_bassin`, `tdm` (bibliothèque `DC_SCHEMAS`, `index.html`). Paramètres (absents = normal ; côtés `"d"` / `"g"`, seuls ou en liste) :
  - `ett` (apicale 4 cavités + VCI sous-costale) : `vg` (`normal`, `hyperkinetique`, `petit`, `dilate`), `akinesie` (`anterieure`), `vd` (`dilate`), `septum` (`paradoxal`), `vci` (`normale`, `collabee`, `dilatee`, `false` = non dessinée), `pericarde` (bool), `ra` (bool : vue 5 cavités, RA calcifié).
  - `efast` (6 fenêtres) : `epanchement` (`morison`, `splenorenal`, `douglas`, `pericarde`, `plevre_d`, `plevre_g`), `pneumothorax` (côtés). `echo_pleuro` : `lignes_b`, `epanchement`, `glissement_absent` (côtés).
  - `echo_veineuse` (avec / sans compression) : `thrombose` (bool), `cote`, `niveau` (`femorale`, `poplitee`). `echo_renale` : `dilatation`, `calcul` (côtés), `petits_reins` (bool), `vessie` (`vide`, `normale`, `globe`).
  - `rp` : `oap` (bool), `foyer` (`lsd`, `lid`, `lsg`, `lig`), `epanchement`, `pneumothorax`, `coupole` (côtés) ; sonde d'intubation et KTC dessinés s'ils étaient posés au moment de l'examen. `rx_bassin` : `disjonction` (`symphyse`, `si_d`, `si_g`).
  - `tdm` (coupe axiale, droite du patient à gauche) : `coupe` `thorax` (`thrombus` : `ap_d`, `ap_g` ; `vd_dilate`), `abdomen` (`hydronephrose`, `infiltration`, `calcul` : côtés), `bassin` (`fracture` : `true` ou `sacro_iliaque_d/g`, `aile_iliaque_d/g`, `sacrum` ; `extravasation`, `hematome` (bool), `cote`) ou `cerveau` ; `coupes: [{…}, {…}]` pour plusieurs coupes.
- `titration` du scénario, optionnel : surcharge des réglages d'un pousse-seringue du catalogue (ex. `{ "noradre": { "max": { "IV": 1, "VVC": 1 } } }` dans le choc hémorragique, où la noradrénaline ne doit pas masquer le saignement).
- `constantes` d'une étape : valeurs imposées à l'entrée (utile pour une étape d'aggravation). Une constante absente du scénario (GCS d'un patient endormi) s'affiche « — ».
- `moniteur` et `voies` du scénario : monitorage et voies déjà en place au début (patient au bloc : `"moniteur": ["CO2"]`, `"voies": ["IV"]`).
- `si_instable` sur une action d'étape (`acr` ou `deces`) : conséquence d'un départ en `transfert` alors que le patient est instable (ex. TDM dans un choc hémorragique non contrôlé). Sans ce champ, partir instable n'a pas de conséquence (le geste est le traitement : embolisation, bloc, coronarographie). Le débriefing signale « parti instable ».
- `transfert` sur une étape (même format que dans le catalogue, plus `si_instable`) : l'étape se termine par un départ ; le bouton de validation devient « 🚑 Partir … » et ouvre la fenêtre de stabilité. Avec `si_instable`, chaque départ instable relance la conséquence : le joueur doit stabiliser avant de repartir.
- `letal` sur une action : `acr` (phase RCP rattrapable) ou `deces` (fin immédiate). `letal_si_manque` : omission létale vérifiée à la validation de l'étape. Par défaut, un geste fait plus tôt compte ; `"etape": ["cardioversion", "choc_ext"]` (ou `true`) exige que ces gestes soient faits pendant l'étape (une cardioversion antérieure ne réduit pas une nouvelle TV). Les gestes visés doivent alors être `repetable`.
- `acr.ecg_racs` : rythme affiché après le RACS (ex. `st_plus` après le choc d'une TV). Sans lui, le rythme d'avant l'ACR revient, sauf un rythme d'arrêt ou une TV (`tv`, `fv`, `asystolie`, `aesp`), remplacé par le rythme de l'étape ou du scénario.
- `acr.requis` : actions à faire pendant la RCP (90 s réelles en temps réel) pour obtenir un RACS ; une liste imbriquée = alternatives. Chaque entrée doit être de la catégorie `RCP` ou `repetable`. Défaut : `["mce", "adre_acr"]`.
- `suite` : la première règle qui correspond l'emporte. `si_manque` : au moins une entrée non faite. `si_fait` : toutes faites. La dernière règle est sans condition. `"vers": "fin"` termine la partie.
- Prévoir pour chaque étape critique une branche d'aggravation (`e1_aggrav`) plutôt que des embranchements multiples.
- Équilibrage : vérifier qu'une prise en charge complète garde le patient au-dessus des seuils, et que l'inaction le fait passer en ACR avant la fin du chrono doublé.

### Silhouette du patient

Encart « 🧍 Patient » au-dessus des résultats, dessiné par le moteur d'après l'état de la partie seulement (rien à écrire dans le contenu) : pâleur (PAS < 90, < 70), cyanose (SpO2 < 90, < 80), marbrures (PAS < 80), fièvre (T ≥ 38,5), sonde et respirateur (action `"moniteur": ["CO2"]`, ou scénario déjà intubé), VVP, IO (`io`), KTC (voie `VVC`), KTA (`"moniteur": ["PA"]`), un pousse-seringue par `titration` en cours, massage animé pendant l'ACR. Repliable, état mémorisé pendant la partie.

### Patient procédural (bouton « 🎲 Patient aléatoire »)

Une **trame** (scénario qui déclare `terrains`) + un **terrain** tiré au hasard : comorbidités, traitements habituels, allergies, antécédents sans conséquence. Conception et décisions : `Dechocage/patient_procedural.md`.

- Trame : `"terrains": { "contexte": "induction", "exclus": ["estomac_plein"], "identite": { "sexe": ["F"], "age": [55, 88], "poids": [50, 95] }, "gravite": { "difficile": { "pente": 1.3, "constantes": { "PAS": -8 } } } }`. Fixer `sexe` tant que les vignettes sont genrées. Le scénario reste jouable tel quel en mode fixe.
  - `contexte` : `dechoc` (défaut), `induction`, `perop` ou `rea` (réa / SSPI) ; filtre proposé au lancement.
  - `gravite` : par niveau, multiplicateur des pentes (`pente`) et décalage des constantes de départ (`constantes`). Défaut : pentes ×0,85 en facile, ×1,2 en difficile.
- Résultats variables : `{3,8~5,4}` dans un texte de résultat est tiré dans la plage, avec les décimales des bornes, une fois par partie ; en mode fixe, valeur médiane. Garder les `**…**` si toute la plage est anormale.
- Tirage : 0-1 comorbidité (facile), 1-2 (moyen), 2-4 (difficile), effets additifs ; un traitement tiré par comorbidité ; 0 à 2 antécédents « bruit » ; constantes de la trame ± 8 %, puis décalées par le terrain. Fiche complète avant le chrono, bandeau dépliable ensuite.
- **`Dechocage/terrains.json`** : `{ "terrains": [...], "traitements": [...], "bruit": [...] }`.

```json
{
  "id": "allergie_betalactamines", "label": "Allergie grave aux bêtalactamines", "rubrique": "allergies",
  "fiche": "Texte de la fiche patient.",
  "ci": [ { "classe": "betalactamine", "complication": "anaphylaxie", "why": "…" } ],
  "promeut": [ { "id": "atb_aztreo_amika", "dans": ["e1"], "why": "…" } ],
  "module": [ { "classe": "remplissage", "effet": { "PAS": 0.5 }, "plus": { "SpO2": -2 }, "why": "…" } ],
  "constantes": { "SpO2": -5 }, "pente": { "PAS": -0.2 }, "bornes": { "FC": [null, 100] },
  "ecg": "fa", "resultats": { "ett": "**FEVG 30 %** …" }, "identite": { "age": [80, 92] },
  "traitements": [["betabloquant", "aspirine"], ["aspirine"]], "exclut": ["autre_terrain"],
  "tags": ["anaphylaxie"]
}
```
  - `rubrique` : `allergies`, `atcd` (défaut) ou `vie` (mode de vie).
  - `ci` : l'action (par `id` ou `classe`) est notée `contre_indique` avec ce `why`, quelle que soit sa note dans la trame, et ne compte pas pour `alt`, `suite` ni `letal_si_manque`, sauf avec `"compte": true` (le geste a bien eu lieu : intubation réussie malgré l'anaphylaxie au curare). Avec `complication`, elle déclenche cette complication. Une action attendue par la trame mais contre-indiquée n'est plus due ; ses alternatives (`alt`) restent dues ; ne pas la faire s'affiche « piège évité ».
  - `promeut` : note portée à `recommande` (jamais `indispensable`), sans pénalité si oubliée. `dans` : limite la règle à une complication (`anaphylaxie`), à un contexte (`induction`, `perop`, `rea`, `dechoc`), à une trame (`isr_occlusion`) ou à une étape d'une trame (`isr_occlusion/e2`) ; aussi possible sur `ci` et `module`.
  - `module` : multiplie l'`effet` des actions visées (`effet`), ajoute un effet propre (`plus`). Avec `complication` et `apres` (nombre) : la complication survient à la N-ième action visée (OAP au 3e remplissage chez l'insuffisant cardiaque).
  - `seuil` : `[{ "PAS": { "min": 70 }, "complication": "ischemie", "why": "…" }]` : la complication survient quand une constante franchit la limite en cours de partie (pas si le patient arrive déjà au-delà).
  - `omission` : `[{ "id": "salle_sans_latex", "dans": ["induction", "perop"], "complication": "anaphylaxie", "why": "…" }]` : geste non fait à la validation d'une étape visée → complication, puis retour à l'étape. Pas de pénalité de points : la complication est la sanction.
  - Chaque déclencheur (seuil, cumul, omission) agit une fois par partie ; il est signalé au débriefing (⚡ ou ⚠️).
  - `constantes` : décalage des constantes de départ et des constantes imposées par une étape. `pente` : s'ajoute à celle de l'étape. `bornes` : `[min, max]` (`null` = pas de borne), ex. FC plafonnée sous bêtabloquant ; levées pendant une TV ou une FV.
  - `resultats` : l'emportent sur ceux de l'étape et du scénario. Clés `action`, `trame/action` ou `trame/étape/action` (la plus précise gagne) : un résultat générique ne doit pas effacer un signe diagnostique de la trame (ECG de l'EP, de l'hyperkaliémie, du STEMI) ; écrire alors un texte combiné propre à la trame. `ecg` : rythme de fond (remplace `sinus`).
  - `identite.age` : tranche d'âge du terrain ; un patient sans ce terrain est tiré hors de la tranche. `identite.poids` : remplace la fourchette de poids de la trame (obésité).
  - `traitements` : variantes (listes d'id de `traitements`), une tirée. Un traitement a `id`, `fiche` et les mêmes clés d'effet (`ci`, `promeut`, `module`, `constantes`, `bornes`, `resultats`, `tags`) ; partagé entre comorbidités, il n'est appliqué qu'une fois.
  - `bruit` : `{ "fiche", "traitement"?, "sexe"? }`, sans effet.
  - `tags` : cartes Recos proposées « Pour réviser » au débriefing (tags communs).
- **`Dechocage/complications.json`** : tableau d'étapes au format habituel (`vignette`, `chrono`, `constantes`, `pente`, `actions`, `diagnostic`, `letal_si_manque`, `acr`), plus `id`, `titre`, `source`, `tags`, **sans `suite`** : à la validation, retour à l'étape interrompue (chrono relancé, pentes des gestes de la complication oubliées). Mettre `refaire: true` sur les gestes qui doivent être faits pendant la complication (sinon un remplissage antérieur compte).
- Les parties procédurales sont enregistrées avec `proc`, `niveau` et les id de terrains (historique par terrain sur l'accueil Déchoc, les plus mal gérés d'abord) ; elles ne comptent pas dans le record du scénario.

### Validation du mode Déchoc (à lancer avant chaque commit touchant `Dechocage/`)

```bash
python3 - <<'EOF'
import json, glob, os, re
NOTES = {"indispensable", "recommande", "debattu", "inutile", "contre_indique"}
VITALS = {"FC", "PAS", "PAD", "SpO2", "FR", "T", "GCS", "EtCO2"}
ECG = {"sinus", "fa", "qrs_larges", "st_plus", "tv", "fv", "asystolie", "aesp"}
acts = json.load(open("Dechocage/actions.json", encoding="utf-8"))
ids = [a["id"] for a in acts]
assert len(ids) == len(set(ids)), "id d'action en double"
A = {a["id"]: a for a in acts}
CLASSES = {c for a in acts for c in a.get("classe", [])}
for a in acts:
    assert a.get("label") and isinstance(a.get("path"), list) and a["path"], f"action {a['id']} : label/path"
    assert set(a.get("moniteur", [])) <= {"PA", "CO2", "MCE"}, f"action {a['id']} : moniteur inconnu"
    assert set(a.get("voie", [])) <= {"IV", "VVC"} and a.get("requiert") in (None, "IV", "VVC"), f"action {a['id']} : voie / requiert"
    assert isinstance(a.get("classe", []), list), f"action {a['id']} : classe doit être une liste"
    if "titration" in a:
        assert {"debut", "gain", "max"} <= a["titration"].keys(), f"action {a['id']} : titration incomplète"
    if "transfert" in a:
        assert set(a["transfert"].get("min", {})) | set(a["transfert"].get("max", {})) <= VITALS | {"PAM"}, f"action {a['id']} : critère de transfert inconnu"
def flat(entries):
    for e in entries:
        yield from (e if isinstance(e, list) else [e])
def check_res(w, res, prefixe=False):
    for k, t in res.items():
        # terrain : "action", "trame/action" ou "trame/étape/action"
        parts = k.split("/") if prefixe else [k]
        assert parts[-1] in A and (len(parts) == 1 or parts[0] in {sc["id"] for sc in SCS}), f"{w} : résultat pour une action ou une trame inconnue {k}"
        assert t.count("**") % 2 == 0, f"{w} : ** non fermé dans « {t} »"
        for m in re.findall(r"\{[^}]*\}", t):
            assert re.fullmatch(r"\{\d+(,\d+)?~\d+(,\d+)?\}", m), f"{w} : plage invalide {m} (format {{3,8~5,4}})"
SCHEMAS = {"ett", "efast", "echo_pleuro", "echo_veineuse", "echo_renale", "rp", "rx_bassin", "tdm"}
def check_schemas(w, o):
    # schéma d'examen : même clé qu'un résultat du même niveau (le schéma illustre ce texte)
    for k, s in o.get("schemas", {}).items():
        assert k in o.get("resultats", {}), f"{w} : schéma {k} sans résultat au même niveau"
        assert isinstance(s, dict) and s.get("type") in SCHEMAS, f"{w} : type de schéma inconnu pour {k}"
def check_step(w, st, steps, compl=False):
    assert st.get("vignette"), f"{w} : vignette manquante"
    assert st.get("ecg", "sinus") in ECG and st.get("acr", {}).get("ecg", "sinus") in ECG and st.get("acr", {}).get("ecg_racs", "sinus") in ECG, f"{w} : ecg inconnu"
    for k in list(st.get("pente", {})) + list(st.get("constantes", {})):
        assert k in VITALS, f"{w} : constante inconnue {k}"
    for aid, spec in st.get("actions", {}).items():
        assert aid in A, f"{w} : action inconnue {aid}"
        assert A[aid].get("type") != "diagnostic", f"{w} : {aid} est un diagnostic"
        assert spec.get("note") in NOTES, f"{w} : note invalide pour {aid}"
        assert spec.get("why"), f"{w} : justification manquante pour {aid}"
        assert spec.get("letal") in (None, "acr", "deces"), f"{w} : letal invalide pour {aid}"
        assert spec.get("si_instable") in (None, "acr", "deces"), f"{w} : si_instable invalide pour {aid}"
        assert "si_instable" not in spec or "transfert" in A[aid], f"{w} : si_instable sur {aid}, qui n'a pas de transfert"
        assert spec.get("ecg", "sinus") in ECG, f"{w} : ecg inconnu ({aid})"
        for k in list(spec.get("effet", {})) + list(spec.get("pente", {})):
            assert k in VITALS, f"{w} : constante inconnue {k} ({aid})"
    check_res(w, st.get("resultats", {}))
    check_schemas(w, st)
    for dx in st.get("diagnostic", []):
        assert A.get(dx, {}).get("type") == "diagnostic", f"{w} : {dx} n'est pas un diagnostic"
    tr = st.get("transfert")
    if tr:
        assert set(tr.get("min", {})) | set(tr.get("max", {})) <= VITALS | {"PAM"} and tr.get("si_instable") in (None, "acr", "deces"), f"{w} : transfert invalide"
    lsm = st.get("letal_si_manque")
    if lsm:
        assert lsm.get("issue") in ("acr", "deces"), f"{w} : issue de letal_si_manque"
        assert lsm.get("etape") in (None, True) or set(lsm["etape"]) <= set(flat(lsm["actions"])), f"{w} : letal_si_manque.etape : true ou liste de gestes de la règle"
        etape = list(flat(lsm["actions"])) if lsm.get("etape") is True else lsm.get("etape") or []
        for aid in etape: assert A[aid].get("repetable") or A[aid]["path"][0] == "RCP", f"{w} : {aid} exigé dans l'étape (letal_si_manque.etape) doit être repetable"
        for aid in flat(lsm["actions"]): assert aid in A, f"{w} : action inconnue {aid}"
    for e in st.get("acr", {}).get("requis", []):
        alts = e if isinstance(e, list) else [e]
        for aid in alts: assert aid in A, f"{w} : action inconnue {aid} (acr.requis)"
        # refaisable pendant la RCP : catégorie RCP ou action repetable
        assert any(A[a].get("repetable") or A[a]["path"][0] == "RCP" for a in alts), f"{w} : {alts} (acr.requis) doit être RCP ou repetable"
    if compl:
        assert "suite" not in st, f"{w} : une complication n'a pas de suite (retour à l'étape interrompue)"
        return
    suite = st.get("suite", [])
    assert suite and not ("si_manque" in suite[-1] or "si_fait" in suite[-1]), f"{w} : la dernière règle de suite doit être sans condition"
    for r in suite:
        assert r["vers"] == "fin" or r["vers"] in steps, f"{w} : cible inconnue {r['vers']}"
        for aid in flat(r.get("si_manque", []) + r.get("si_fait", [])): assert aid in A, f"{w} : action inconnue {aid}"
SCS = [json.load(open(f, encoding="utf-8")) for f in glob.glob("Dechocage/sc_*.json")]
# complications injectées par le terrain : une étape, sans suite
COMPL = {}
if os.path.exists("Dechocage/complications.json"):
    for c in json.load(open("Dechocage/complications.json", encoding="utf-8")):
        w = f"complications.json [{c.get('id')}]"
        assert c.get("id") and c.get("titre") and c.get("source"), f"{w} : id / titre / source"
        assert c["id"] not in COMPL, f"{w} : id en double"
        COMPL[c["id"]] = c
        check_step(w, c, set(), compl=True)
# terrains : comorbidités, traitements, antécédents sans conséquence
TERR = set()
DANS = {sc["id"] for sc in SCS} | {f"{sc['id']}/{e['id']}" for sc in SCS for e in sc["etapes"]} | {"dechoc", "induction", "perop", "rea"}
if os.path.exists("Dechocage/terrains.json"):
    T = json.load(open("Dechocage/terrains.json", encoding="utf-8"))
    TR = {x["id"]: x for x in T.get("traitements", [])}
    assert len(TR) == len(T.get("traitements", [])), "terrains.json : id de traitement en double"
    def check_src(w, s):
        for kind in ("ci", "promeut", "module", "omission", "seuil"):
            for r in s.get(kind, []):
                if kind == "seuil":
                    assert r.get("complication") in COMPL and r.get("why") and set(r) - {"complication", "why", "dans"} <= VITALS, f"{w} : seuil invalide"
                    for k in set(r) & VITALS: assert set(r[k]) <= {"min", "max"}, f"{w} : seuil {k} : min / max"
                    continue
                if kind == "omission":
                    assert r.get("id") in A and r.get("complication") in COMPL and r.get("why") and r.get("dans"), f"{w} : omission : id, dans, complication, why"
                    continue
                assert ("id" in r) != ("classe" in r), f"{w} : {kind} vise soit un id, soit une classe"
                assert r.get("id", "") in A or r.get("classe") in CLASSES, f"{w} : {kind} vise une action ou une classe inconnue ({r.get('id') or r.get('classe')})"
                assert kind == "module" or r.get("why"), f"{w} : {kind} sans justification"
                assert set(r.get("dans", [])) <= COMPL.keys() | DANS, f"{w} : {kind}.dans : ni complication, ni trame, ni « trame/étape »"
                if kind in ("ci", "module") and "complication" in r:
                    assert kind == "ci" or isinstance(r.get("apres"), int), f"{w} : module avec complication : apres (nombre) requis"
                    assert r["complication"] in COMPL, f"{w} : complication inconnue {r['complication']}"
                if kind == "module":
                    assert set(r.get("effet", {})) | set(r.get("plus", {})) <= VITALS, f"{w} : constante inconnue (module)"
        assert set(s.get("constantes", {})) | set(s.get("pente", {})) | set(s.get("bornes", {})) <= VITALS, f"{w} : constante inconnue"
        assert s.get("ecg", "sinus") in ECG, f"{w} : ecg inconnu"
        check_res(w, s.get("resultats", {}), prefixe=True)
        check_schemas(w, s)
    for t in T["terrains"]:
        w = f"terrains.json [{t.get('id')}]"
        assert t.get("id") and t.get("label") and t.get("fiche"), f"{w} : id / label / fiche"
        assert t["id"] not in TERR, f"{w} : id en double"
        TERR.add(t["id"])
        assert t.get("rubrique", "atcd") in {"atcd", "allergies", "vie"}, f"{w} : rubrique inconnue"
        assert set(t.get("identite", {})) <= {"age", "poids"}, f"{w} : identite : age / poids"
        for var in t.get("traitements", []):
            for x in var: assert x in TR, f"{w} : traitement inconnu {x}"
        check_src(w, t)
    for x in TR.values():
        assert x.get("fiche"), f"terrains.json [{x['id']}] : fiche manquante"
        check_src(f"terrains.json [{x['id']}]", x)
    for b in T.get("bruit", []):
        assert b.get("fiche") and b.get("sexe") in (None, "F", "M"), f"terrains.json : antécédent « bruit » invalide {b}"
n = 0
for f in sorted(glob.glob("Dechocage/sc_*.json")):
    sc = json.load(open(f, encoding="utf-8")); n += 1
    for k in ("id", "titre", "source", "tags", "patient", "constantes", "etapes"):
        assert k in sc, f"{f} : champ {k} manquant"
    assert set(sc["constantes"]) <= VITALS, f"{f} : constante inconnue"
    assert sc.get("ecg", "sinus") in ECG, f"{f} : ecg inconnu"
    assert set(sc.get("alarmes", {})) <= {"FC", "PAS", "SpO2"}, f"{f} : alarme inconnue"
    assert set(sc.get("moniteur", [])) <= {"PA", "CO2"} and set(sc.get("voies", [])) <= {"IV", "VVC"}, f"{f} : moniteur / voies"
    for k in sc.get("titration", {}): assert "titration" in A.get(k, {}), f"{f} : {k} n'a pas de titration au catalogue"
    if "terrains" in sc:
        assert set(sc["terrains"].get("exclus", [])) <= TERR, f"{f} : terrain exclu inconnu"
        assert set(sc["terrains"].get("identite", {})) <= {"sexe", "age", "poids"}, f"{f} : identite : sexe / age / poids"
        assert sc["terrains"].get("contexte", "dechoc") in {"dechoc", "induction", "perop", "rea"}, f"{f} : contexte inconnu"
        for niv, g in sc["terrains"].get("gravite", {}).items():
            assert niv in {"facile", "moyen", "difficile"} and set(g) <= {"pente", "constantes"} and set(g.get("constantes", {})) <= VITALS, f"{f} : gravite invalide"
    check_res(f, sc.get("resultats", {}))
    check_schemas(f, sc)
    steps = {e["id"] for e in sc["etapes"]}
    assert not steps & COMPL.keys(), f"{f} : une étape porte l'id d'une complication"
    for st in sc["etapes"]:
        check_step(f"{f} [{st['id']}]", st, steps)
print(f"{len(acts)} actions, {n} scénarios, {len(TERR)} terrains, {len(COMPL)} complications : OK")
EOF
```

## Commits

- Messages en français, préfixés par la zone : `Recos : lot14, 12 cartes sur …`, `Déchoc : scénario …` ou `ScrollMed : …` pour l'app.
- Résumer dans le corps du message ce que contient le lot (thème, nombre de cartes, format).
