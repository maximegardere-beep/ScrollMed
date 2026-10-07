# ScrollMed

Application de révision en anesthésie-réanimation au format « reels » : on fait défiler des cartes courtes (recommandations, posologies, quiz…).

- `index.html` : toute l'app (HTML + CSS + JS, sans build ni dépendance). Contient aussi quelques cartes de départ codées en dur.
- `Recos/` : **le registre de cartes**. L'app liste ce dossier via l'API GitHub, sur la branche `main` par défaut, et charge chaque fichier `.json` à chaque lancement. Une carte poussée sur `main` est donc en ligne tout de suite.
- `Recos/cartes_complementaires_a_creer.md` : liste des sujets restant à traiter. Cocher (`- [x]`) les sujets couverts par un nouveau lot.
- `Dechocage/` : contenu du **mode Déchoc** (jeu de simulation de déchocage, onglet ✚ de la barre du bas) : catalogue d'actions et scénarios, synchronisés comme `Recos/`. Voir la section « Mode Déchoc ».

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

- Chrono par étape (`chrono`, 90 par défaut), **doublé par le moteur** (`DC_CHRONO_MULT`) : un `chrono` de 90 donne 3 min réelles. **1 s réelle = 5 s patient**, donc une étape dure le même temps patient qu'avant (`chrono` × 10 s), avec plus de temps pour naviguer.
- Chaque action consomme `duree` minutes patient : l'horloge avance d'autant et la dégradation s'applique d'un coup.
- `pente` : variation par minute patient, par constante (`FC`, `PAS`, `PAD`, `SpO2`, `FR`, `T`, `GCS`, `EtCO2`). Une variation de PAS entraîne la PAD de moitié si la PAD n'est pas précisée. Une fois le chrono de l'étape écoulé, les pentes sont doublées.
- `seuils` du scénario (défaut `{ "PAS": 50, "SpO2": 70 }`) : passer sous un seuil déclenche un ACR.

### Scope

- Courbes en balayage : ECG et pléthysmographie toujours ; PA invasive après une action `"moniteur": ["PA"]` (sinon PNI toutes les 3 min patient, ou à la demande en touchant la case PA) ; capnographie après une action `"moniteur": ["CO2"]` (intubation). `"moniteur": ["MCE"]` (massage) ajoute l'artefact de compressions et la capno de RCP. Ces valeurs se mettent dans le catalogue.
- `ecg` : rythme affiché, parmi `sinus`, `fa`, `qrs_larges`, `st_plus`, `tv`, `fv`, `asystolie`, `aesp`. Sur le scénario (défaut `sinus`), sur une étape (à l'entrée), sur une action d'étape (ex. le calcium affine les QRS) ou sur `acr` (sinon déduit du texte de `rythme` : FV, TV, asystolie, sinon AESP). Tachycardie et bradycardie découlent de la FC.
- `EtCO2` : constante optionnelle (38 par défaut), abaissée par le bas débit.
- `alarmes` du scénario, optionnel : surcharge des seuils `[priorité moyenne, haute]`, défaut `{ "FC": { "bas": [50, 40], "haut": [120, 150] }, "PAS": { "bas": [90, 70] }, "SpO2": { "bas": [90, 85] } }`. L'alarme PA porte sur la valeur affichée (PNI ou invasive).
- Sons (coupés par défaut) : bip de pouls plus grave quand la SpO2 baisse, alarmes moyenne et haute façon IEC 60601-1-8, silence 2 min.

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
- Les actions se réalisent par **appui maintenu** (≈ 0,5 s) sur la tuile, ce qui évite les touchers accidentels. La tuile affiche la durée patient.
- La catégorie `RCP` n'apparaît que pendant un ACR.
- Ne jamais renommer un `id` d'action (les scénarios s'y réfèrent) ni un `id` de scénario (l'historique des joueurs s'y réfère).

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

- **Notes** : `indispensable` (+3 si fait, −3 si oublié), `recommande` (+2), `debattu` (0), `inutile` (−1), `contre_indique` (−5). Une action absente de l'étape vaut `inutile`, sauf si une autre étape l'attend (elle est alors comptée comme anticipée). Diagnostic juste à la validation : +2. ACR récupéré : −5. Décès : 0/20.
- `why` est obligatoire : c'est le texte du débriefing. Il doit être exact et sourcé, comme une carte.
- `alt` : actions interchangeables (plusieurs antibiotiques acceptables). Une seule suffit pour satisfaire l'indispensable.
- `effet` : variation immédiate des constantes. `pente` sur une action : remplace la pente de l'étape pour ces constantes, jusqu'à la fin de la partie.
- `refaire: true` : l'action doit être refaite dans cette étape (contrôle), un passage antérieur ne compte pas.
- `resultats` : texte révélé par un examen. Celui de l'étape l'emporte sur celui du scénario. Entourer chaque valeur anormale de `**…**` : elle s'affiche en rouge (ex. `"**K⁺ 7,9 mmol/L** · Na 140 mmol/L"`).
- `titration` du scénario, optionnel : surcharge des réglages d'un pousse-seringue du catalogue (ex. `{ "noradre": { "max": { "IV": 1, "VVC": 1 } } }` dans le choc hémorragique, où la noradrénaline ne doit pas masquer le saignement).
- `constantes` d'une étape : valeurs imposées à l'entrée (utile pour une étape d'aggravation).
- `letal` sur une action : `acr` (phase RCP rattrapable) ou `deces` (fin immédiate). `letal_si_manque` : omission létale vérifiée à la validation de l'étape.
- `acr.requis` : actions à faire pendant la RCP (90 s réelles) pour obtenir un RACS ; une liste imbriquée = alternatives. Chaque entrée doit être de la catégorie `RCP` ou `repetable`. Défaut : `["mce", "adre_acr"]`.
- `suite` : la première règle qui correspond l'emporte. `si_manque` : au moins une entrée non faite. `si_fait` : toutes faites. La dernière règle est sans condition. `"vers": "fin"` termine la partie.
- Prévoir pour chaque étape critique une branche d'aggravation (`e1_aggrav`) plutôt que des embranchements multiples.
- Équilibrage : vérifier qu'une prise en charge complète garde le patient au-dessus des seuils, et que l'inaction le fait passer en ACR avant la fin du chrono doublé.

### Validation du mode Déchoc (à lancer avant chaque commit touchant `Dechocage/`)

```bash
python3 - <<'EOF'
import json, glob
NOTES = {"indispensable", "recommande", "debattu", "inutile", "contre_indique"}
VITALS = {"FC", "PAS", "PAD", "SpO2", "FR", "T", "GCS", "EtCO2"}
ECG = {"sinus", "fa", "qrs_larges", "st_plus", "tv", "fv", "asystolie", "aesp"}
acts = json.load(open("Dechocage/actions.json", encoding="utf-8"))
ids = [a["id"] for a in acts]
assert len(ids) == len(set(ids)), "id d'action en double"
A = {a["id"]: a for a in acts}
for a in acts:
    assert a.get("label") and isinstance(a.get("path"), list) and a["path"], f"action {a['id']} : label/path"
    assert set(a.get("moniteur", [])) <= {"PA", "CO2", "MCE"}, f"action {a['id']} : moniteur inconnu"
    assert set(a.get("voie", [])) <= {"IV", "VVC"} and a.get("requiert") in (None, "IV", "VVC"), f"action {a['id']} : voie / requiert"
    if "titration" in a:
        assert {"debut", "gain", "max"} <= a["titration"].keys(), f"action {a['id']} : titration incomplète"
def flat(entries):
    for e in entries:
        yield from (e if isinstance(e, list) else [e])
n = 0
for f in sorted(glob.glob("Dechocage/sc_*.json")):
    sc = json.load(open(f, encoding="utf-8")); n += 1
    for k in ("id", "titre", "source", "tags", "patient", "constantes", "etapes"):
        assert k in sc, f"{f} : champ {k} manquant"
    assert set(sc["constantes"]) <= VITALS, f"{f} : constante inconnue"
    assert sc.get("ecg", "sinus") in ECG, f"{f} : ecg inconnu"
    assert set(sc.get("alarmes", {})) <= {"FC", "PAS", "SpO2"}, f"{f} : alarme inconnue"
    for k in sc.get("titration", {}): assert "titration" in A.get(k, {}), f"{f} : {k} n'a pas de titration au catalogue"
    for st in sc["etapes"]:
        for t in [*sc.get("resultats", {}).values(), *st.get("resultats", {}).values()]:
            assert t.count("**") % 2 == 0, f"{f} : ** non fermé dans « {t} »"
    steps = {e["id"] for e in sc["etapes"]}
    for k in sc.get("resultats", {}):
        assert k in A, f"{f} : résultat pour une action inconnue {k}"
    for st in sc["etapes"]:
        w = f"{f} [{st['id']}]"
        assert st.get("vignette"), f"{w} : vignette manquante"
        assert st.get("ecg", "sinus") in ECG and st.get("acr", {}).get("ecg", "sinus") in ECG, f"{w} : ecg inconnu"
        for k in list(st.get("pente", {})) + list(st.get("constantes", {})):
            assert k in VITALS, f"{w} : constante inconnue {k}"
        for aid, spec in st.get("actions", {}).items():
            assert aid in A, f"{w} : action inconnue {aid}"
            assert A[aid].get("type") != "diagnostic", f"{w} : {aid} est un diagnostic"
            assert spec.get("note") in NOTES, f"{w} : note invalide pour {aid}"
            assert spec.get("why"), f"{w} : justification manquante pour {aid}"
            assert spec.get("letal") in (None, "acr", "deces"), f"{w} : letal invalide pour {aid}"
            assert spec.get("ecg", "sinus") in ECG, f"{w} : ecg inconnu ({aid})"
            for k in list(spec.get("effet", {})) + list(spec.get("pente", {})):
                assert k in VITALS, f"{w} : constante inconnue {k} ({aid})"
        for k in st.get("resultats", {}):
            assert k in A, f"{w} : résultat pour une action inconnue {k}"
        for dx in st.get("diagnostic", []):
            assert A.get(dx, {}).get("type") == "diagnostic", f"{w} : {dx} n'est pas un diagnostic"
        lsm = st.get("letal_si_manque")
        if lsm:
            assert lsm.get("issue") in ("acr", "deces"), f"{w} : issue de letal_si_manque"
            for aid in flat(lsm["actions"]): assert aid in A, f"{w} : action inconnue {aid}"
        for e in st.get("acr", {}).get("requis", []):
            alts = e if isinstance(e, list) else [e]
            for aid in alts: assert aid in A, f"{w} : action inconnue {aid} (acr.requis)"
            # refaisable pendant la RCP : catégorie RCP ou action repetable
            assert any(A[a].get("repetable") or A[a]["path"][0] == "RCP" for a in alts), f"{w} : {alts} (acr.requis) doit être RCP ou repetable"
        suite = st.get("suite", [])
        assert suite and not ("si_manque" in suite[-1] or "si_fait" in suite[-1]), f"{w} : la dernière règle de suite doit être sans condition"
        for r in suite:
            assert r["vers"] == "fin" or r["vers"] in steps, f"{w} : cible inconnue {r['vers']}"
            for aid in flat(r.get("si_manque", []) + r.get("si_fait", [])): assert aid in A, f"{w} : action inconnue {aid}"
print(f"{len(acts)} actions, {n} scénarios : OK")
EOF
```

## Commits

- Messages en français, préfixés par la zone : `Recos : lot14, 12 cartes sur …`, `Déchoc : scénario …` ou `ScrollMed : …` pour l'app.
- Résumer dans le corps du message ce que contient le lot (thème, nombre de cartes, format).
