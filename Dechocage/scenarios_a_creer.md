# Scénarios Déchoc à créer

Choisis par QCM. Cocher (`- [x]`) quand le scénario est poussé sur `main`.
Les actions et diagnostics nécessaires sont **déjà dans `actions.json`** (sauf mention).
Avant de pousser : validation de `CLAUDE.md` et simulation d'une prise en charge complète, d'une inaction et des erreurs clés.

- [x] **A. Embolie pulmonaire à haut risque** (`sc_ep_grave.json`, fait)

- [ ] **B. Asthme aigu grave** (GINA 2024 ; ERC 2021, asthme)
  - Femme, 26 ans, 58 kg, asthme mal contrôlé. FC 132, PA 128/76, SpO2 87 %, FR 34, GCS 15.
  - e1 Accueil : O2 (cible 93-95 %), salbutamol nébulisé répété, corticoïde systémique dans l'heure (indispensables) ; ipratropium, MgSO4 2 g IV si non-réponse (recommandés) ; salbutamol IV débattu ; sédatifs (midazolam, morphine) et bêtabloquant contre-indiqués ; intubation inutile à ce stade. Si bronchodilatateur ou corticoïde manquant → `e1_aggrav`.
  - e2 Épuisement (somnolence, thorax silencieux, bradypnée, PaCO2 78) : intubation indispensable (kétamine ou étomidate ; propofol débattu, hypotension), `vm_asthme` (FR basse, expiration longue, hypercapnie permissive) indispensable, remplissage avant l'induction. Si `vm_asthme` manquant → `e3_collapsus`.
  - e3 Collapsus après intubation (auto-PEEP) : `deconnexion` indispensable, remplissage, écho pleurale (pas de PNO) ; exsufflation sans PNO contre-indiquée. ACR : déconnecter, comprimer le thorax, chercher un PNO compressif (ERC 2021).
  - Diagnostic : `dx_aag`.

- [ ] **C. Plaie thoracique : pneumothorax compressif puis tamponnade** (ATLS 10e éd. 2018 ; ERC 2021, arrêt traumatique ; CRASH-2)
  - Homme, 27 ans, 75 kg, plaie par arme blanche parasternale gauche. FC 134, PA 86/60, SpO2 83 %, FR 36.
  - e1 PNO compressif (diagnostic clinique) : `exsufflation` indispensable (alt `drain_thorax`), O2, VVP ; E-FAST recommandé sans retarder ; radio débattue ; intubation contre-indiquée avant décompression (létale ACR). Létal si pas de décompression ; ACR : massage + exsufflation.
  - e2 Choc persistant, PA pincée, jugulaires turgescentes : E-FAST à refaire (épanchement péricardique, collapsus de l'OD) ; bloc (sternotomie) indispensable, péricardiocentèse en attente, remplissage et CGR recommandés ; intubation contre-indiquée (létale). ACR : `thoracotomie_sauvetage`.
  - Diagnostics : `dx_pnx` puis `dx_tamponnade` (et `dx_choc_obstructif`).

- [ ] **D. Œdème pulmonaire hypertensif** (ESC 2021, insuffisance cardiaque aiguë)
  - Femme, 79 ans, 68 kg, HTA. PA 214/118, FC 118, SpO2 81 %, FR 38.
  - e1 : VNI indispensable, nitrés IV à forte dose (classe IIb si PAS > 110), furosémide (classe I), ECG ; morphine contre-indiquée (classe III), remplissage et bêtabloquant contre-indiqués, intubation inutile si VNI. Sans VNI → `e1_aggrav` (épuisement, intubation indispensable).
  - e2 Réévaluation H+1 : recherche du facteur déclenchant (ECG, troponine, ETT : HVG, dysfonction diastolique), relais des nitrés IVSE, orientation USIC.
  - Diagnostic : `dx_oap`.

- [ ] **F. Acidocétose diabétique** (consensus ADA-EASD-JBDS 2024, crises hyperglycémiques de l'adulte)
  - Femme, 21 ans, 54 kg, diabète de type 1, gastro-entérite, insuline arrêtée. FC 126, PA 94/56, FR 30 (Kussmaul), GCS 14. Glycémie 4,9 g/L, pH 7,08, HCO3⁻ 6, cétonémie 6,1, K⁺ 5,7.
  - e1 : remplissage (NaCl 0,9 % ou soluté balancé), kaliémie avant l'insuline, `insuline_ivse` 0,1 UI/kg/h sans bolus ; pas de KCl si K⁺ > 5 ; bicarbonate inutile si pH ≥ 7,0 ; βHCG ; intubation contre-indiquée (perte de la compensation respiratoire, létale). Sans remplissage ou insuline → `e1_aggrav` (collapsus).
  - e2 Contrôle H+2 : glycémie, cétonémie, iono à refaire ; K⁺ 3,8 → `kcl_iv` indispensable ; G10 quand la glycémie passe sous 2,5 g/L. Sans KCl → `e2_aggrav` (hypokaliémie, ESV, MgSO4 ; FV si oubli).
  - Diagnostic : `dx_acidocetose`.

- [ ] **I. Traumatisme crânien grave avec engagement** (SFAR 2016, TC grave à la phase précoce ; CRASH 2004 ; CRASH-3 2019)
  - Homme, 34 ans, 80 kg, chute de scooter sans casque. GCS 6, mydriase droite, SpO2 88 %, FC 58, PA 160/88.
  - e1 : intubation (kétamine ou étomidate ; propofol débattu), osmothérapie (signes d'engagement), PAS > 110 mmHg (noradrénaline), normocapnie, collier, tête à 30° ; NaCl 0,9 % plutôt que solutés hypotoniques ; corticoïdes contre-indiqués (CRASH) ; acide tranexamique débattu (CRASH-3 : bénéfice dans les TC légers à modérés). Sans intubation ou osmothérapie → `e1_aggrav` (Cushing, mydriase bilatérale ; décès si pas d'osmothérapie).
  - e2 Scanner : TDM (hématome extradural de 32 mm, déviation de 9 mm), neurochirurgie indispensable, sédation IVSE, contrôle des gaz du sang.
  - Diagnostic : `dx_tc_grave`.

- [ ] **J. État de mal épileptique** (SRLF-SFMU 2018 ; ESETT 2019 ; RAMPART 2012)
  - Homme, 52 ans, 75 kg, éthylisme, épilepsie post-traumatique, traitement arrêté. Crise généralisée depuis 10 min. SpO2 87 %, FC 132, PA 172/96, GCS 3.
  - e1 : benzodiazépine (clonazépam IV, ou midazolam IM sans voie veineuse), O2, glycémie capillaire, thiamine (éthylisme) ; pas d'intubation d'emblée. Sans benzodiazépine → `e1_aggrav`.
  - e2 Crise persistante à 5 min : 2e dose de benzodiazépine, puis 2e ligne (fosphénytoïne, lévétiracétam ou phénobarbital ; valproate débattu chez l'éthylique).
  - e3 EME réfractaire (H+40 min) : intubation (propofol, kétamine ou étomidate relayé), sédation IVSE, EEG (état de mal non convulsif), TDM cérébral une fois intubé, réanimation ; PL débattue.
  - Diagnostic : `dx_eme`.
