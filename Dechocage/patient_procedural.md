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

### Implémentation de l'étape 1
- **D31** (2026-10-08) : l'allergie aux bêtalactamines de la v1 est une allergie **à toutes les bêtalactamines testées** (anaphylaxie à la ceftriaxone, tests positifs à l'amoxicilline, à la ceftriaxone et à l'imipénème). Une allergie isolée à la pénicilline n'aurait pas contre-indiqué le méropénème, ni forcément la C3G.
- **D32** (2026-10-08) : dans la trame choc septique, l'aztréonam + amikacine est une alternative `debattu` de l'antibiothérapie (hors allergie, la C3G reste le premier choix) ; le terrain allergique la promeut à `recommande`.
- **D33** (2026-10-08) : une action contre-indiquée par le terrain ne satisfait ni les alternatives, ni la `suite`, ni `letal_si_manque` ; elle n'est plus attendue, ses alternatives le restent. Ne pas la faire, l'alternative faite, s'affiche « piège évité ».
- **D34** (2026-10-08) : la promotion par le terrain peut être limitée à certaines étapes ou complications (`dans`) : le glucagon n'est promu que pendant l'anaphylaxie du bêtabloqué.
- **D35** (2026-10-08) : les parties procédurales n'entrent pas dans le record du scénario (difficulté différente) ; elles sont enregistrées avec niveau et terrains.

### Implémentation de l'étape 2 (anesthésie, trame ISR)
- **D36** (2026-10-08) : l'induction reste modélisée par des **combinaisons hypnotique + curare** (6 tuiles : étomidate, kétamine ou propofol, avec succinylcholine ou rocuronium) plutôt que par des drogues séparées. Les scénarios existants restent compatibles : les 3 nouvelles combinaisons y reprennent la note de celle qui a le même hypnotique. Exception : en hyperkaliémie, les combinaisons avec succinylcholine deviennent contre-indiquées.
- **D37** (2026-10-08) : une contre-indication peut garder le geste comme fait (`"compte": true`) : une intubation au rocuronium chez l'allergique réussit malgré l'anaphylaxie, alors qu'une ISR chez l'intubation difficile connue échoue (complication CICO).
- **D38** (2026-10-08) : nouveaux terrains : **allergie au rocuronium** (tests négatifs à la succinylcholine et au cisatracurium : il faut lire le courrier d'allergologie), **intubation difficile connue** (ISR → CICO ; fibroscopie vigile promue), **rétrécissement aortique serré** (chute de PA des hypnotiques ×1,8). L'insuffisance cardiaque à FEVG altérée amplifie aussi l'hypotension d'induction (×1,5).
- **D39** (2026-10-08) : nouvelle complication **CICO** (SFAR 2017, DAS 2015) : appel à l'aide, dispositif supraglottique, cricothyroïdotomie ; sans oxygénation rétablie, ACR.
- **D40** (2026-10-08) : `dans` accepte un id de trame ou « trame/étape », les id d'étape (`e1`, `e2`) se répétant d'une trame à l'autre.
- **D41** (2026-10-08) : RA serré + propofol : collapsus profond, rattrapable ou ACR selon la PA de départ tirée. Pas d'ACR forcé.

### Implémentation de l'étape 3
- **D42** (2026-10-08) : résultats variables écrits `{3,8~5,4}` dans le texte du résultat ; la mise en rouge reste explicite (`**…**`) plutôt qu'automatique : l'auteur choisit une plage entièrement anormale ou normale. En mode fixe, valeur médiane (D16).
- **D43** (2026-10-08) : gravité par défaut : pentes ×0,85 en facile, ×1 en moyen, ×1,2 en difficile ; la trame peut préciser pentes et constantes par niveau (`gravite`).
- **D44** (2026-10-08) : contextes `dechoc`, `induction`, `perop`, `rea` déclarés par la trame ; filtre en puces sur l'écran de difficulté, avec le nombre de trames.
- **D45** (2026-10-08) : historique par terrain sur l'accueil Déchoc (moyenne, nombre de parties, survie), les terrains les plus mal gérés en tête. Les patients aléatoires ne comptent plus dans « Patients déjà rencontrés ».

- **D46** (2026-10-08) : trois déclencheurs de complication en plus de la contre-indication, une fois chacun par partie :
  - `omission` : geste non fait à la validation d'une étape d'un contexte donné (salle sans latex en induction ou en peropératoire → anaphylaxie) ;
  - `seuil` : constante qui franchit une limite en cours de partie (PAS < 70 chez le coronarien ou le stenté → ischémie) ;
  - `apres` sur une modulation : N-ième action d'une classe (3e remplissage chez l'insuffisant cardiaque → OAP).
  Pas de pénalité de points : la complication est la sanction, signalée au débriefing.
- **D47** (2026-10-08) : terrain **estomac plein reporté** : toutes les trames actuelles le supposent déjà (déchoc, ISR) ou l'excluent (SSPI). Il prendra sens avec une trame de chirurgie programmée.
- **D48** (2026-10-08) : pas de complication séparée pour l'**inhalation** (déjà une branche de la trame ISR : induction trop tardive) ni pour le **collapsus d'induction** (produit par la modulation des hypnotiques : RA serré, grand âge, FEVG altérée, IEC). La myasthénie et le SAOS prennent leur sens dans la trame SSPI (sugammadex et PPC promus).
- **D49** (2026-10-08) : `identite.poids` d'un terrain remplace la fourchette de poids de la trame (obésité : 110 à 145 kg). `dans` accepte aussi un contexte (`induction`, `perop`, `rea`, `dechoc`).

- **D50** (2026-10-08) : un scénario peut démarrer avec monitorage et voies en place (`moniteur`, `voies`) : patient déjà intubé au bloc. Une constante absente (GCS sous anesthésie générale) s'affiche « — ».
- **D51** (2026-10-08) : trame SSPI centrée sur la **curarisation résiduelle** (SFAR 2018) ; le laryngospasme et la dépression morphinique pure restent pour une autre trame. Myasthénie : néostigmine contre-indiquée (sous pyridostigmine) ; SAOS : PPC promue en SSPI ; allergie au rocuronium exclue (le rocuronium a été utilisé). L'omission « salle sans latex » ne joue qu'en induction (D46 révisé : en peropératoire, la salle est déjà installée).

- **D52** (2026-10-08) : les 4 scénarios restants deviennent des trames, sexe masculin fixé (vignettes genrées, cancer de la prostate dans l'EP). Exclusions : STEMI sans coronarien connu, stent récent ni RA serré ; EP sans FA anticoagulée ; hyperkaliémie sans HTA (ramipril déjà dans la vignette) ; hémorragie peropératoire sans FA anticoagulée (anticoagulant arrêté avant une chirurgie programmée). Le plafond de FC du bêtabloquant est levé en TV ou en FV. Sous AVK, le CCP + vitamine K est promu dans les trames hémorragiques (HAS 2008).

### Nouvelles trames et terrains (2026-10-09/10)
- **D53** (2026-10-09) : 11 trames choisies par QCM (asthme aigu grave, plaie thoracique, acidocétose, TC grave, EME, BAV complet, induction programmée, toxicité des AL, intubation difficile imprévue, hématome cervical, dépression morphinique). Livrées : plaie thoracique, acidocétose, EME, BAV complet, hématome cervical, dépression morphinique (14 trames au total). Les 5 autres sont en brouillon dans `Dechocage/reserve/` (D60).
- **D54** (2026-10-09) : 4 terrains ajoutés : **estomac plein** (révise D47 : CI de l'induction classique en contexte `induction` → complication **inhalation**, qui révise D48), **IRC dialysée** (succinylcholine CI, morphine accumulée, OAP au 3e remplissage), **diabète** (gastroparésie : ISR étomidate / kétamine promues en induction), **cirrhose / éthylisme** (thiamine promue, coagulopathie, propranolol). Sans `identite.age` (le moteur tirerait trop jeunes les patients sans le terrain). `estomac_plein` exclu des trames hors induction.
- **D55** (2026-10-09) : nouvelles actions `ind_classique` (classe `induction_classique`), `videolaryngoscope` (tentative d'intubation, pose la capno, sans classe `intubation`), `bougie`, `reveil_patient`, `aspiration_pharyngee`, `arret_injection_al`, `intralipide`, `ouverture_cicatrice`, `naloxone_ivse`, `ees_endocavitaire`, `bav_cardio_pacemaker`, `stimulation_eveil`, `naloxone_bolus`, et 5 diagnostics. Les ISR portent la classe `isr`.
- **D56** (2026-10-09) : rythmes ECG `bav3` (P dissociées, échappement large) et `entraine` (spicule + QRS large), persistants après RACS.
- **D57** (2026-10-09) : l'éthylisme de l'EME vient du terrain cirrhose (vignette neutre) ; l'acidocétose garde une DT1 de 28-50 ans et exclut le diabète de type 2 ; le BAV exclut tous les terrains qui peuvent tirer un bêtabloquant (D58).
- **D60** (2026-10-10) : 5 trames mises en réserve à la demande de l'utilisateur (quota) : asthme aigu grave, TC grave, induction programmée, toxicité des AL, intubation difficile imprévue.

### Partie continue, hémodynamique, débriefing (2026-10-10, moteur)
- **D61** : modèle **hybride** choisi par QCM : prise en charge continue, étapes réservées aux vrais tournants cliniques, fin par une orientation puis bilan. Moteur : `fin` au catalogue (`reanimation`, `bloc`, `radio_interv`, `embolectomie`, `neurochir` ; pas `uro_drainage` ni `epuration`, gestes après lesquels le patient continue, ni `coro`, qui alerte la salle), `transfert` sur `reanimation`. Une orientation termine la partie dès qu'elle est faite ; restent dus les indispensables de l'étape et du chemin par défaut de `suite` (anti-départ précoce tant que les trames gardent leurs étapes « H+30 »). `"fin": false` dans l'acidocétose (e1, collapsus : la leçon sur le potassium vient après) et le STEMI (la réa vient après la coronarographie).
- **D62** : étapes-tournants par `declencheur` (temps, constantes, gestes faits ou manquants) ; l'étape quittée reste due jusqu'à la fin. `effet` par défaut au catalogue pour un geste fait hors de son étape (aucun encore renseigné : à faire avec la conversion des trames).
- **D63** : hémodynamique : `volume` au catalogue (cristalloïdes et HEA 500 mL, albumine 20 % 100 mL, CGR 280 mL, PFC 250 mL), bilan entrées, `hemodynamique` du scénario (réserve de précharge, seuil d'OAP) modulé par le terrain, résultats conditionnels (`si`), actions `lever_jambes` et `vti`. Une complication ne survient qu'une fois par partie (OAP du volume et OAP du 3e bolus de l'insuffisant cardiaque ne se cumulent pas).
- **D64** : débriefing : points forts en tête (indispensables faits avec l'heure, diagnostics justes, pièges évités, ACR récupérés, gestes recommandés, orientation stable), puis au plus 3 axes par priorité, chronologie, bilan entrées ; détail action par action replié. `delai` sur l'action d'étape, sans effet sur la note.
- **D65** : reste la conversion des 14 trames (unité 4) : supprimer les étapes « H+30 : et ensuite ? », passer les aggravations en `declencheur`, renseigner `hemodynamique`, ETT conditionnelles, `delai` (ATB, transfusion…), `effet` par défaut des remplissages ; puis `hemodynamique` des terrains FEVG altérée et IRC dialysée à la place de leur OAP au 3e bolus.

- **D66** (unité 4, conversion des 14 trames) : étapes « H+30 / réévaluation / contrôle » fondues dans l'étape d'accueil (sepsis, choc hémorragique, EP, hyperkaliémie, BAV, acidocétose « Contrôle à H+2 », hématome « Reprise au bloc ») ; aggravations en `declencheur` (temps, constantes, geste manquant) dans les 14 trames ; `hemodynamique` des états de choc et des trames où le remplissage compte (EP réserve 0 / OAP 1,5 L, STEMI 0 / 1 L, sepsis 2 / 4,5 L, bassin 4 / 6 L…), ETT conditionnelles ; `delai` : ATB ≤ 60 min et réa ≤ 6 h (SSC 2021), acide tranexamique ≤ 3 h (CRASH-2), ECG ≤ 10 min (ESC 2023). Hyperkaliémie : terrains à bêtabloquant exclus (D58) ; STEMI : plus de `si_instable` sur le départ en salle (la salle est le traitement), fin par la validation « Partir en salle ». Moteur : un tournant applique d'abord l'évolution en attente (temps fixé) ; une orientation qui n'a pas terminé la partie reste refaisable ; partir sans un geste du `letal_si_manque` de l'étape a la même issue que valider sans lui ; une promotion de terrain ne relève plus une action que la trame juge contre-indiquée.

## Écarts relevés le 2026-10-10 (à corriger)
- [ ] **D58. Bêtabloquants cumulés** : coronarien + cirrhose tirent bisoprolol et propranolol, décalages de FC additionnés (−45/min). Le moteur devrait regrouper les traitements d'une même classe. L'hyperkaliémie (FC 42) n'exclut pas ces terrains : FC possible vers 17.
- [ ] **D59. CI de terrain et `suite`** : une action attendue par la trame mais contre-indiquée par le terrain, non faite, déclenche quand même `suite.si_manque` (KCl chez la dialysée dans l'acidocétose).
- [ ] Antécédents « bruit » sans âge (hystérectomie en 2005 chez une femme de 28 ans).
- [ ] Un geste fait pendant une complication n'a que l'effet prévu par la complication (EES pendant une ischémie).
- [ ] La promotion de la VNI chez le SAOS en contexte `rea` s'applique aussi à l'hématome cervical (VNI « recommandée » en pleine compression) : la limiter à `sspi_curarisation`.
- [ ] Choc septique : partir en réa sans drainage ne coûte qu'un oubli (−3) ; le décès du choc réfractaire n'est pas appliqué (ajouter `uro_drainage` à un `letal_si_manque` ?).
- [ ] Complication `oap` : VNI indispensable, mal adaptée à l'EP (VNI débattue).
- [ ] Les ETT fixes des terrains FEVG altérée et RA serré l'emportent sur les ETT conditionnelles (acidocétose, plaie thoracique après péricardiocentèse) : les écrire en listes conditionnelles.
- [ ] Orientations manquantes au catalogue : « Poursuite de l'intervention » (ISR), « Sortie de SSPI », « Surveillance continue / SSPI prolongée » (dépression morphinique).
- [ ] Hématome cervical, étape « Œdème laryngé » en temps réel : l'induction (10 min) ne se termine presque jamais avant l'ACR.
- [ ] Sources à vérifier par l'utilisateur (conversion) : SFAR 2012 remplissage périopératoire (ISR), guideline européen 2023 (choc hémorragique), ESAIC 2022 (hémorragie peropératoire), UK Kidney Association 2020 (hyperkaliémie), seuil « bicarbonates < 10 » attribué à JBDS (critère ADA), chiffres ajoutés sans source (TAPSE 10 mm, VCI 25 mm, lactates 1,6, glycémie 1,10 g/L).
- [ ] Sources à vérifier par l'utilisateur, listées dans les rapports des trames (ADA 2024 seuil de K⁺, DAS 2022 grades, SRLF-SFMU 2018 choix de l'hypnotique, RCP Isuprel, andexanet ESC 2023…).

## Ordre de livraison
- **D30** (2026-10-08) :
  1. ✅ **Moteur sur une trame existante** : `terrains.json`, clé `classe` sur les actions, tirage du patient, fiche + bandeau, notes et modulations par le terrain, complication injectée **anaphylaxie**, débriefing annoté. Testé sur `sc_choc_septique.json` (allergie aux bêtalactamines).
  2. ✅ **Anesthésie** (livrée le 2026-10-08) : actions d'induction et de voies aériennes difficiles, trame **ISR** (`sc_isr_occlusion.json`), terrains et complication CICO (D36-D41).
  3. ✅ **Suite** (livrée le 2026-10-08) : trames hémorragie peropératoire et SSPI, terrains et complications manquants, conversion des scénarios, gravité et résultats variables, filtre par contexte, historique par terrain (D42-D52). Voir « Écarts à corriger ».

## Plan de l'étape 1 : moteur sur une trame existante

- [x] **Étape 1 livrée** (2026-10-08). Format de référence : `CLAUDE.md`, section « Patient procédural ». Choix faits pendant l'implémentation : voir D31 à D35.

Objectif : jouer `sc_choc_septique.json` en « Patient aléatoire » avec un terrain tiré, dont l'allergie aux bêtalactamines qui déclenche une anaphylaxie si on l'ignore. Tout le reste du mode Déchoc reste identique.

### 1. Données

**`actions.json`** (ne renommer aucun `id`) :
- clé `classe` (liste) sur les actions concernées par les terrains de la v1 : `betalactamine` (C3G + amikacine, pipéracilline-tazobactam, méropénème), `remplissage`, `hypnotique`, `curare`, `betabloquant`, `vasopresseur`, `anticoagulant`, `antiagregant`, `ains`…
- nouvelles actions :
  - `atb_aztreo_amika` « Aztréonam + amikacine » (alternative des PNA graves chez l'allergique ; à vérifier dans SPILF 2018 au moment de la rédaction) ;
  - `adre_titree` « Adrénaline IV titrée (bolus) », `repetable` (anaphylaxie : bolus selon le grade) ;
  - `glucagon` « Glucagon IV » (anaphylaxie réfractaire chez le bêtabloqué) ;
  - `tryptase` « Tryptase sérique », `examen`, dans `Bilan` ;
  - `arret_produit` « Arrêt du produit suspect ».

**`terrains.json`** (nouveau) : objet `{ "terrains": [...], "bruit": [...] }`.
```json
{
  "id": "allergie_betalactamines",
  "label": "Allergie aux bêtalactamines",
  "fiche": "Choc anaphylactique à l'amoxicilline en 2019 (bilan allergologique)",
  "rubrique": "allergies",
  "famille": "allergie",
  "ci": [ { "classe": "betalactamine", "complication": "anaphylaxie", "why": "…" } ],
  "promeut": [],
  "module": [],
  "constantes": {},
  "pente": {},
  "identite": {},
  "traitements": [],
  "tags": ["anaphylaxie", "allergie"]
}
```
- `ci` : la note devient `contre_indique` (D4), la `complication` est injectée (D19).
- `promeut` : `{ "classe" | "id", "why" }` ; la note monte à `recommande` sans jamais dépasser (D25), y compris pour une action absente de l'étape.
- `module` : `{ "classe", "effet": { "PAS": 0.5 }, "plus": { "SpO2": -2 } }` : multiplie l'`effet` des actions de la classe, ajoute un effet propre (D26).
- `constantes` : décalage des constantes de départ (BPCO : `SpO2 -6`) ; `pente` : pente ajoutée à celle de l'étape (réserve réduite).
- `identite` : contraintes sur l'identité (sujet âgé : `age: [76, 92]` ; obésité : `imc: [40, 50]`).
- `traitements` : variantes tirées (D28), chacune avec `fiche` et ses propres `ci` / `promeut` / `module`. Ex. FA : AOD, AVK ou aucun.
- `bruit` : antécédents sans effet (D22), `{ "fiche", "rubrique" }`.

Terrains écrits à l'étape 1 (sous-ensemble de D13-D14, pour tester tous les mécanismes) : allergie aux bêtalactamines, bêtabloqué (dont coronarien sous bêtabloquant), insuffisance cardiaque à FEVG basse, BPCO, sujet âgé, FA anticoagulée, plus une dizaine d'antécédents « bruit ».

**`complications.json`** (nouveau) : tableau d'étapes au format habituel (`vignette`, `chrono`, `constantes`, `ecg`, `pente`, `actions` avec `note` et `why`, `acr`, `diagnostic`), plus `id` et `titre`. Pas de `suite` : à la validation, retour à l'étape interrompue. Étape 1 : `anaphylaxie` seule (grade III : collapsus, tachycardie, bronchospasme, érythème ; adrénaline titrée, arrêt du produit, remplissage, O2 indispensables ou recommandés ; tryptase recommandée ; glucagon promu par le terrain bêtabloqué ; corticoïdes débattus). Sources : RFE SFAR-SFA sur l'anaphylaxie périopératoire et recommandations EAACI / WAO, années à vérifier à la rédaction.

**Trame `sc_choc_septique.json`** : ajouter
```json
"terrains": {
  "exclus": [],
  "identite": { "sexe": ["F"], "age": [55, 85], "poids": [50, 95] }
}
```
- `sexe` fixé tant que les vignettes sont genrées (« Adressée par le SAMU ») ; `age` et `poids` tirés, puis contraints par le terrain.
- Ajouter `atb_aztreo_amika` aux étapes e1 et e1_aggrav (note `recommande` ; le `why` cite le cas de l'allergique) et aux listes `suite.si_manque` d'antibiotiques.

### 2. Moteur (`index.html`)

- **Synchro** (`syncDechocFromGithub`) : charger `terrains.json` et `complications.json` à côté de `actions.json` ; `DC_CONTENT` gagne `terrains`, `bruit`, `complications`. Contenu absent : le bouton « Patient aléatoire » est masqué, rien d'autre ne change.
- **Tirage** (`dcGenPatient(trame, difficulte)`, nouveau) : nombre de comorbidités selon D7 ; terrains tirés sans remise hors `exclus` et sans conflit d'identité ; un traitement tiré par comorbidité ; 0 à 2 antécédents « bruit » ; identité ; constantes de la trame ± 5-10 % puis décalages du terrain, bornées par `DC_LIMITS`. Résultat `G.terrain = { items, identite, fiche }`.
- **Lancement** : `dcStart(scId, mode, proc)` ; bouton « 🎲 Patient aléatoire » sur l'accueil, avec choix de la difficulté (facile, moyen, difficile) puis du rythme existant. Tirage parmi les scénarios qui ont une clé `terrains`.
- **Fiche patient** (D29) : écran avant le chrono (identité, ATCD, traitements, allergies), puis bandeau compact sous le scope, déplié d'un toucher. Le chrono ne part qu'à la fermeture de la fiche.
- **Notation** (`dcNoteFor`) : surcouche terrain. Une CI force `contre_indique` avec son `why` ; un `promeut` force au moins `recommande`. La spec d'origine est conservée pour le débriefing.
- **Effets** (`dcDoAction`) : multiplier `spec.effet` selon `module`, ajouter `plus` ; pente de terrain ajoutée dans `dcSlopes()`.
- **Complication** : une action contre-indiquée entre dans l'étape de `complications.json` (`dcEnterStep` sur une étape clonée, avec mémoire de l'étape interrompue) ; à la validation, retour à l'étape interrompue, chrono relancé. Une action qui a déclenché une complication ne satisfait ni `alt`, ni `suite`, ni `letal_si_manque` (l'antibiotique allergisant ne compte pas comme antibiothérapie).
- **Débriefing** : annotation sur l'action touchée (« Contre-indiqué chez ce patient : allergie aux bêtalactamines », note d'origine barrée) ; ligne « piège évité » quand une action promue est faite ou qu'une action contre-indiquée est évitée alors qu'elle était attendue par la trame ; liens vers les cartes Recos dont les tags recoupent ceux du terrain (D11).
- **Historique** : `STATE.dechoc.runs` enregistre `proc: true`, la difficulté et les id de terrains (l'écran d'historique par terrain viendra à l'étape 3).

### 3. Validation et tests

- Étendre le script de validation de `CLAUDE.md` : `classe` sur les actions ; dans `terrains.json`, `ci.complication` existant dans `complications.json`, classes et id référencés existants, constantes connues ; complications au format d'étape ; trame avec `terrains.exclus` connus.
- Tests Playwright (Chromium préinstallé) : tirage de 200 patients par difficulté sans incohérence (nombre de comorbidités, identité, constantes bornées) ; partie choc septique avec allergie : C3G donnée → anaphylaxie → adrénaline titrée → retour à e1 → aztréonam → fin, débriefing annoté ; même trame sans terrain identique au mode fixe (non-régression) ; inaction pendant l'anaphylaxie → ACR.
- Mettre à jour `CLAUDE.md` (section Mode Déchoc : terrains, complications, clé `terrains` d'une trame, `classe`) et cocher l'étape 1 ici.

### 4. Hors étape 1
Sévérité variable et résultats variables (D17), autres complications (D23), trames ISR, hémorragie peropératoire, SSPI (D18), actions d'anesthésie, conversion des 4 autres scénarios, écran d'historique par terrain (D15).

## Écarts à corriger (relevés le 2026-10-08)

Cocher quand c'est fait. Les écarts déjà prévus à l'étape 3 y sont regroupés ; les autres sont ajoutés au plan.

- [x] **3a. Notation** : quand le joueur fait plusieurs actions d'un même groupe d'alternatives, chacune marque ses points (deux inductions « indispensables » dans le STEMI : score > 20). Une seule doit compter. *(hors plan initial)*
- [x] **3b. Moteur**
  - [x] gravité variable selon la difficulté (D17, D21)
  - [x] résultats biologiques variables (D17)
  - [x] filtre par contexte au lancement (D5) *(hors plan initial)*
  - [x] écran d'historique par terrain (D15)
- [x] **3c. Terrains manquants** (D13, D14) : HTA, asthme, SAOS, obésité, allergie au latex, myasthénie, double antiagrégation. Estomac plein reporté (D47).
- [x] **3d. Complications manquantes** (D23) : bronchospasme, OAP de surcharge, ischémie myocardique. Inhalation et collapsus d'induction traités autrement (D48).
- [x] **3e. Trames** (D18) : hémorragie peropératoire (`sc_hemorragie_perop.json`), détresse respiratoire en SSPI par curarisation résiduelle (`sc_sspi_curarisation.json`)
- [x] **3f. Conversion en trames** (D16) : choc hémorragique, EP grave, hyperkaliémie, STEMI (8 trames au total)
- [ ] **Sources à vérifier par l'utilisateur** *(hors plan initial)* : aztréonam + amikacine (SPILF 2018), glucagon chez le bêtabloqué (SFAR/SFA 2011), intubation vigile si intubation et ventilation au masque difficiles (SFAR 2017), étude IRIS (Sellick)

## Questions ouvertes

_(aucune pour l'instant)_
