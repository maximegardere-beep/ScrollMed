# Patient procédural : guide de conception

Projet : générer des patients variables dans le mode Déchoc (anesthésie ou réanimation, pathologie(s) aiguë(s) ± associées, antécédents) pour travailler la gestion des contre-indications et multiplier les cas.

Document tenu à jour au fil des QCM de conception. Chaque décision est datée et numérotée ; une décision révisée est barrée, pas supprimée.

## Décisions

### Architecture
- **D1** (2026-10-08) : intégré **dans le mode Déchoc**, en parallèle des scénarios pré-rédigés (`sc_*.json`). Même moteur (constantes, scope, catalogue d'actions, débriefing) ; le patient procédural est un second type de partie.

- **D5** (2026-10-08) : lancement par un **bouton dédié** « Patient aléatoire » à côté de la liste des scénarios, avec filtres optionnels (contexte, difficulté).

### Contextes couverts
- **D6** (2026-10-08) : les trames peuvent couvrir les quatre contextes :
  - **déchoc / urgence** (patient qui arrive : choc, détresse, trauma) ;
  - **induction au bloc** (programmé ou urgent : drogues d'induction, ISR, intubation difficile prévue) ;
  - **complication peropératoire** (patient endormi : anaphylaxie, hyperthermie maligne, hémorragie, bronchospasme, toxicité des AL) ;
  - **dégradation en réa / SSPI** (patient hospitalisé qui s'aggrave).

### Génération du patient
- **D16** (2026-10-08) : **format unique**. Un scénario `sc_*.json` qui déclare une clé `terrains` est aussi jouable en patient aléatoire : il sert de trame. Les 5 scénarios existants deviennent des trames en ajoutant cette clé ; joués en mode fixe, ils restent inchangés.
- **D2** (2026-10-08) : **hybride**. Une trame rédigée (pathologie aiguë / situation) + un terrain tiré au hasard parmi les terrains déclarés compatibles avec cette trame.

- **D7** (2026-10-08) : nombre de comorbidités **selon la difficulté** choisie au lancement : facile 0-1, moyen 1-2, difficile 2-4 (effets additifs, cf. D10).

- **D17** (2026-10-08) : variabilité en plus du terrain :
  - **identité tirée** : sexe, âge, poids dans les fourchettes de la trame, compatibles avec le terrain (âgé, obèse) ;
  - **constantes bruitées** (± 5-10 % autour des valeurs de la trame) ;
  - **sévérité variable** : la trame définit plusieurs degrés de gravité (constantes, pentes), tirés selon la difficulté ;
  - **résultats variables** : valeurs biologiques tirées dans des plages, mise en rouge automatique hors normes.

### Trames prioritaires (nouveaux contextes)
- **D18** (2026-10-08) : à écrire en premier :
  - **induction en séquence rapide** (urgence chirurgicale, estomac plein ; terrains : allergie, IOT difficile, RA, insuffisance cardiaque) ;
  - **hémorragie peropératoire** (transfusion, acide tranexamique, fibrinogène ; terrains : anticoagulé, antiagrégé, coronarien) ;
  - **détresse respiratoire en SSPI** (curarisation résiduelle, laryngospasme, morphiniques ; terrains : SAOS, obèse, BPCO, myasthénie).
  - Reportée : anaphylaxie peropératoire comme trame (elle existera comme complication injectée, cf. D19).
- **D20** (2026-10-08) : compatibilité trame / terrain : **tous les terrains sauf ceux exclus** par la trame (ex. pas d'« estomac plein » en SSPI).
- **D21** (2026-10-08) : en plus du nombre de comorbidités (D7), la difficulté ne joue que sur la **sévérité de la trame**. Le rythme (temps réel, ralenti, fixé) reste le réglage temporel. Écartés : fiche annotée en facile, terrains pièges favorisés en difficile.
- **D22** (2026-10-08) : **antécédents « bruit de fond »** sans conséquence (appendicectomie, hypothyroïdie traitée, tabac sevré…), à dose modérée, pour le réalisme et le tri de l'information.

### Effets physiologiques du terrain
- **D8** (2026-10-08) : le terrain agit sur
  - les **constantes de base** (BPCO : SpO2 88-92 % ; HTA chronique : PAS de base élevée ; IRC : K⁺ et créatinine modifiés dans les résultats) ;
  - la **réponse aux agressions et aux traitements** (bêtabloqué : pas de tachycardie compensatrice ; sujet âgé : chute de PA plus forte à l'induction ; cardiopathie : remplissage mal toléré, OAP).
  - Écartés pour l'instant : cibles modifiées par le terrain (cible de PAM, cible SpO2 BPCO), posologies à choisir.

### Terrains de la v1
- **D24** (2026-10-08) : susceptibilité à l'hyperthermie maligne et myopathie **reportées** (leurs complications ne sont pas dans la v1, cf. D23).
- **D13** (2026-10-08) : familles retenues pour la v1 :
  - **cardiovasculaire** : coronarien, insuffisance cardiaque à FEVG basse, FA sous anticoagulant, rétrécissement aortique, bêtabloqué, HTA ;
  - **respiratoire / VAS** : BPCO, asthme, SAOS, intubation difficile connue, estomac plein ;
  - **allergies / pharmaco / neuro** : allergie aux bêtalactamines, aux curares, au latex ; ~~susceptibilité à l'hyperthermie maligne~~ ; myasthénie ; ~~myopathie~~ ; antiagrégants.
  - Révision (D24) : susceptibilité à l'hyperthermie maligne et myopathie reportées avec leurs complications.
  - Reportées : métabolique / rénal / foie (IRC dialysée, diabète, cirrhose).
- **D14** (2026-10-08) : terrains physiologiques retenus : **sujet âgé** (> 75 ans) et **obésité** (IMC > 40). Écartés : grossesse, enfant (normes de constantes par âge non gérées par le moteur).

### Découverte du terrain
- **D3** (2026-10-08) : **tout est affiché d'emblée** (fiche patient complète : ATCD, traitements, allergies). Le défi porte sur l'adaptation de la prise en charge, pas sur la recherche d'information.

### Contre-indications
- **D4** (2026-10-08) : une action contre-indiquée par le terrain entraîne
  - une **note dégradée** (ex. recommandé → contre-indiqué), expliquée au débriefing ;
  - un **effet physiologique** visible sur le scope (anaphylaxie, hyperkaliémie sous succinylcholine, bronchospasme sous bêtabloquant…).
  - Écartés pour l'instant : alternative rendue indispensable, alerte avant l'acte.
- **D19** (2026-10-08) : l'effet physiologique passe par une **complication injectée** : complications génériques réutilisables (anaphylaxie, bronchospasme, hyperthermie maligne, hyperkaliémie…) décrivant constantes, ECG et actions attendues (adrénaline, dantrolène…), insérées comme une étape dans la partie. Le joueur peut la traiter.
- **D23** (2026-10-08) : complications de la v1 :
  - **anaphylaxie** (bêtalactamine, curare, latex : adrénaline titrée, remplissage, tryptase ; glucagon si bêtabloqué) ;
  - **voies aériennes** : intubation impossible / CICO (IOT difficile connue ignorée), inhalation (estomac plein sans ISR), bronchospasme (asthme, bêtabloquant) ;
  - **cardiovasculaire** : collapsus à l'induction (RA serré, sujet âgé), OAP de surcharge (FEVG basse), ischémie myocardique (coronarien tachycarde ou anémié).
  - Reportée : pharmacogénétique (hyperthermie maligne, hyperkaliémie à la succinylcholine sur myopathie).

- **D25** (2026-10-08) : le terrain peut aussi **promouvoir une action jusqu'à `recommande`** (+2 si faite, rien si oubliée) : préoxygénation en VNI chez l'obèse, glucagon chez le bêtabloqué en anaphylaxie… Jamais jusqu'à `indispensable` (cohérent avec D4).
- **D26** (2026-10-08) : la modulation de la réponse (D8) passe **par classe d'action**, même mécanisme que les CI (D12) : le terrain multiplie l'effet des actions d'une classe (FEVG basse → remplissage PAS ×0,5 et SpO2 qui baisse ; sujet âgé → hypnotiques PAS ×1,5).
- **D27** (2026-10-08) : **barème inchangé** : les notes modifiées par le terrain suffisent ; score sur 20 comparable aux scénarios fixes.

### Données
- **D9** (2026-10-08) : **catalogue central** `Dechocage/terrains.json` : chaque comorbidité / traitement / allergie y décrit une fois pour toutes ses modificateurs de constantes, de réponse et ses CI. Les trames listent les terrains compatibles (ou exclus).
- **D12** (2026-10-08) : les CI ciblent des **classes pharmacologiques** : nouvelle clé `classe` sur les actions de `actions.json` (ex. `"classe": ["betalactamine"]`, `curare_depolarisant`, `ains`, `betabloquant`…). Un terrain cible une classe, ce qui couvre aussi les actions ajoutées plus tard.
- **D10** (2026-10-08) : comorbidités à **effets additifs** : chacune applique ses effets, sans règle d'interaction spécifique.

- **D28** (2026-10-08) : **traitements habituels tirés selon la comorbidité** : chaque comorbidité propose ses traitements possibles (FA → AVK, AOD ou aucun ; coronarien → aspirine ± clopidogrel, bêtabloquant). Le traitement tiré porte ses propres CI et modulations.

### Interface
- **D29** (2026-10-08) : **fiche complète avant le chrono**, puis **bandeau compact** (ATCD et allergies clés) toujours visible pendant la partie, dépliable d'un toucher.

### Suivi du joueur
- **D15** (2026-10-08) : **historique des scores par terrain** (repérer les comorbidités mal gérées, à la manière du radar de compétences). Écartés : code patient partageable, « rejouer ce patient », tirage ciblé sur les faiblesses.

### Débriefing
- **D11** (2026-10-08) : le débriefing montre le rôle du terrain par
  - une **annotation sur chaque action touchée** (« Contre-indiqué chez ce patient : allergie aux bêtalactamines », note d'origine barrée) ;
  - les **pièges évités** : bonne adaptation valorisée (ex. rocuronium plutôt que succinylcholine chez l'insuffisant rénal hyperkaliémique). Valorisation au débriefing, sans rendre l'alternative indispensable (cf. D4) ;
  - un **lien vers les cartes Recos** correspondantes (tags communs).
  - Écartée : section « Terrain » séparée.

## Ordre de livraison
- **D30** (2026-10-08) :
  1. **Moteur sur une trame existante** : `terrains.json`, clé `classe` sur les actions, tirage du patient, fiche + bandeau, notes et modulations par le terrain, complication injectée **anaphylaxie**, débriefing annoté. Testé sur `sc_choc_septique.json` (allergie aux bêtalactamines).
  2. **Anesthésie** : nouvelles actions (hypnotiques et curares d'induction, sugammadex, préoxygénation VNI…) et trame **ISR**.
  3. **Suite** : trames hémorragie peropératoire et détresse respiratoire en SSPI, autres complications de D23, conversion des autres scénarios en trames, historique par terrain.

## Questions ouvertes

_(aucune pour l'instant)_
