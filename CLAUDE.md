# ScrollMed

Application de révision en anesthésie-réanimation au format « reels » : on fait défiler des cartes courtes (recommandations, posologies, quiz…).

- `index.html` : toute l'app (HTML + CSS + JS, sans build ni dépendance). Contient aussi quelques cartes de départ codées en dur.
- `Recos/` : **le registre de cartes**. L'app liste ce dossier via l'API GitHub, sur la branche `main` par défaut, et charge chaque fichier `.json` à chaque lancement. Une carte poussée sur `main` est donc en ligne tout de suite.
- `Recos/cartes_complementaires_a_creer.md` : liste des sujets restant à traiter. Cocher (`- [x]`) les sujets couverts par un nouveau lot.

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
keys = collections.Counter()
for f in sorted(glob.glob("Recos/*.json")):
    cards = json.load(open(f, encoding="utf-8"))   # plante si le JSON est invalide
    assert isinstance(cards, list), f"{f} : doit être un tableau"
    for i, c in enumerate(cards):
        miss = REQ - c.keys()
        assert not miss, f"{f}[{i}] : champs manquants {miss}"
        assert isinstance(c["tags"], list), f"{f}[{i}] : tags doit être une liste"
        keys[(c["module"] + "::" + c["hook"]).lower().strip()] += 1
dups = [k for k, n in keys.items() if n > 1]
print(f"{sum(keys.values())} cartes, {len(keys)} uniques")
assert not dups, f"doublons module::hook : {dups}"
EOF
```

## Commits

- Messages en français, préfixés par la zone : `Recos : lot14, 12 cartes sur …` ou `ScrollMed : …` pour l'app.
- Résumer dans le corps du message ce que contient le lot (thème, nombre de cartes, format).
