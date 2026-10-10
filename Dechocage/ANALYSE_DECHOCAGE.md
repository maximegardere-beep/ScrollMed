# Analyse des Faille du Module Dechocage

**Date** : 2025-10-10  
**Auteur** : Vibe Code (avec validation utilisateur)  
**Objectif** : Identifier, hiérarchiser et corriger les failles techniques et médicales du module Dechocage pour améliorer la qualité pédagogique et la conformité aux recommandations.

---

## 📌 Méthodologie

1. **Recensement** : Analyse exhaustive des fichiers `actions.json`, `complications.json`, `terrains.json`, et des scénarios (`sc_*.json`).
2. **Hiérarchisation** : Classification en **Critique** (🔴), **Majeure** (🟠), **Mineure** (⚪) selon l'impact sur :
   - La **sécurité du patient simulé** (ex: CI non bloquées → risque d'ACR évitable).
   - La **qualité pédagogique** (ex: protocoles obsolètes → mauvaises pratiques apprises).
   - L'**expérience utilisateur** (ex: feedback retardé → frustration).
3. **Solutions proposées** : Correctifs **simplifiés et faisables** en VibeCoding, priorisés pour un impact maximal avec un effort minimal.

---

## 🎯 Hiérarchisation des Correctifs

### 🔴 **PRIORITÉ 1 : CORRECTIFS CRITIQUES** *(À corriger en premier - Impact élevé, effort faible)*

#### **1.1. Contre-indications non appliquées** *(Risque : Actions dangereuses autorisées)*
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** | **Sources** |
|-------|------------|----------------|------------|----------------------|------------|------------|
| **CI-001** | Allergie aux bêta-lactamines : `atb_c3g_amika` reste `indispensable` | `terrains.json` + `sc_choc_septique.json` | 🔴 Joueur peut déclencher une anaphylaxie | 1. Ajouter `"ci": [{"classe": "betalactamine", "complication": "anaphylaxie"}]` dans `allergie_betalactamines`.<br>2. Promouvoir `atb_aztreo_amika` en `indispensable` si terrain `allergie_betalactamines`. | ⭐ | SFAR/SFA 2011 |
| **CI-002** | Myasthénie : `succinylcholine` non CI | `terrains.json` | 🔴 Risque d'hyperkaliémie mortelle | Ajouter `"ci": [{"id": "isr_*_succi", "complication": "hyperkaliemie"}]` dans `myasthenie`. | ⭐ | RCP |
| **CI-003** | IRC dialysée : `succinylcholine` non CI | `terrains.json` | 🔴 Risque d'hyperkaliémie | Ajouter `"ci": [{"classe": "curare_depolarisant", "complication": "hyperkaliemie"}]` dans `irc_dialyse`. | ⭐ | UK Kidney Association 2020 |
| **CI-004** | BPCO : `o2_masque` sans limite de SpO2 | `terrains.json` | 🔴 Risque d'hypercapnie | Ajouter `"constantes": {"SpO2": -5}` + note : "Cible SpO2 88-92% (BTS 2017)". | ⭐ | BTS 2017 |
| **CI-005** | Coronarien : `nitres_iv` non CI en hypotension | `terrains.json` | 🔴 Risque d'hypotension aggravée | Ajouter `"ci": [{"id": "nitres_iv", "dans": ["*"]}, "why": "Contre-indiqué si PAS < 100 mmHg"}]`. | ⭐ | ESC 2023 |

**Actions requises** :
- [ ] Corriger `CI-001` à `CI-003` (3h max).
- [ ] Ajouter une **note explicative** dans le débriefing si une CI est ignorée.

---

#### **1.2. Protocoles Obsolètes ou Non Conformes** *(Risque : Mauvaise pratique apprise)*
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** | **Sources** |
|-------|------------|----------------|------------|----------------------|------------|------------|
| **PROT-001** | Choc septique : `remp_nacl` marqué `debattu` | `sc_choc_septique.json` | 🔴 SSC 2021 recommande les **cristalloïdes balancés** (RL) en 1ère intention | 1. Changer `remp_nacl` en `contre_indique` dans `sc_choc_septique`.<br>2. Ajouter une note : "NaCl 0.9% → acidose hyperchlorémique (SSC 2021)". | ⭐⭐ | SSC 2021 |
| **PROT-002** | Choc septique : `hydrocortisone` marqué `debattu` en e1 | `sc_choc_septique.json` | 🟠 Devrait être **recommandé** si noradrénaline > 0.25 µg/kg/min | 1. Changer en `recommande` si `noradre` est démarré depuis > 4h.<br>2. Ajouter un `declencheur` pour promouvoir `hydrocortisone`. | ⭐⭐ | SSC 2021 |
| **PROT-003** | EME : `fosphénytoïne` effet `PAS: -10` | `actions.json` | 🟠 Effet exagéré (hypotension rare à dose thérapeutique) | Réduire l'effet à `PAS: -3`. | ⭐ | ESETT 2019 |
| **PROT-004** | Anaphylaxie : `glucagon` marqué `inutile` | `complications.json` | 🔴 Devrait être **indispensable** si terrain `betabloquant` | 1. Ajouter `"promeut": [{"id": "glucagon", "dans": ["anaphylaxie"], "why": "Bêta-bloqué : glucagon IV si résistance à l'adrénaline"}]` dans `betabloquant`.<br>2. Changer `glucagon` en `indispensable` si terrain `betabloquant`. | ⭐⭐ | SFAR/SFA 2011 |

**Actions requises** :
- [ ] Corriger `PROT-001` et `PROT-004` (priorité absolue).
- [ ] Revoir `PROT-002` et `PROT-003` si temps disponible.

---

#### **1.3. Effets Physiologiques Incohérents** *(Risque : Simulation irréaliste)*
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** | **Sources** |
|-------|------------|----------------|------------|----------------------|------------|------------|
| **EFF-001** | `remp_cristal` et `remp_nacl` ont le même effet | `actions.json` | 🟠 RL et NaCl 0.9% n'ont pas les mêmes effets | 1. Séparer les effets :<br>   - `remp_cristal` : `PAS: +8`, `FC: -6`, `pH: +0.01` (équilibre acido-basique).<br>   - `remp_nacl` : `PAS: +8`, `FC: -6`, `pH: -0.02` (acidose hyperchlorémique). | ⭐⭐ | SSC 2021 |
| **EFF-002** | Pas de seuil de PAM pour déclencher des complications | Moteur | 🔴 Risque d'ischémie non détecté | Ajouter un **seuil global** : si `PAM < 65` pendant > 5 min → déclencher `ischemie` (si terrain `coronarien`). | ⭐⭐⭐ | ERC 2021 |
| **EFF-003** | `noradre` : pas de dose maximale | `actions.json` | ⚪ Manque de réalisme | Ajouter `"max": {"IV": 0.5, "VVC": 1.0}` (µg/kg/min). | ⭐ | SSC 2021 |

**Actions requises** :
- [ ] Corriger `EFF-001` (priorité moyenne).
- [ ] Implémenter `EFF-002` si temps (sinon reporter).

---

### 🟠 **PRIORITÉ 2 : CORRECTIFS MAJEURS** *(À corriger après les critiques - Impact élevé, effort modéré)*

#### **2.1. Logique des Actions et Priorités**
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|------------|----------------------|------------|
| **LOG-001** | Pas d'ordre de priorité dans les scénarios | `sc_*.json` | 🟠 Joueur ne sait pas par où commencer | Ajouter un champ `"priorite": [1, 2, 3]` dans les actions `indispensable`. Exemple :<br>```json
"atb_c3g_amika": {"note": "indispensable", "priorite": 2, ...},
"remp_cristal": {"note": "indispensable", "priorite": 1, ...},
"noradre": {"note": "indispensable", "priorite": 3, ...}
``` | ⭐⭐⭐ |
| **LOG-002** | `repetable: true` sans limite de doses | `actions.json` | 🟠 Risque de surdosage simulé | Ajouter `"max_dose": 3` pour les actions répétables (ex: `salbutamol_neb`). | ⭐⭐ |
| **LOG-003** | Actions `indispensable` non hiérarchisées | `sc_*.json` | 🟠 Confusion pour le joueur | 1. Afficher les actions `indispensable` **par ordre de priorité** dans l'interface.<br>2. Ajouter un **résumé des étapes clés** avant de commencer. | ⭐⭐ |

---

#### **2.2. Contenu Médical à Améliorer**
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|------------|----------------------|------------|
| **MED-001** | Pas de dose pour `adre_titree` | `actions.json` | 🟠 Manque de précision | Ajouter `"dose": "0.1-0.2 µg/kg/min (bolus répétés)"`. | ⭐ |
| **MED-002** | `exacyl` sans indication claire | `actions.json` | ⚪ Manque de contexte | Ajouter `"indication": "Hémorragie massive (PTM)"` dans la description. | ⭐ |
| **MED-003** | `isr_*` sans checklist d'intubation difficile | `actions.json` | 🟠 Oubli de la sécurité | Ajouter une **note obligatoire** : "Vérifier : aspiration préte, chariot d'ID, plan B annoncé". | ⭐⭐ |

---

#### **2.3. Gestion des Complications**
| **ID** | **Faille** | **Localisation** | **Impact** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|------------|----------------------|------------|
| **COMP-001** | `oap` : `furosemide` marqué `debattu` | `complications.json` | 🔴 Devrait être **contre-indiqué** en choc | Changer en `contre_indique` + note : "Aggrave l'hypovolémie". | ⭐ |
| **COMP-002** | `anaphylaxie` : `ephedrine` marqué `inutile` | `complications.json` | ⚪ Devrait être **contre_indique** | Changer en `contre_indique` + note : "Inefficace, risque d'aggravation". | ⭐ |
| **COMP-003** | Pas de feedback si CI ignorée | Moteur | 🟠 Joueur ne comprend pas son erreur | Ajouter un **message d'alerte** dans le débriefing :<br>"⚠️ Vous avez utilisé [action] alors que le terrain [terrain] l'interdit. Risque : [complication]." | ⭐⭐ |

---

### ⚪ **PRIORITÉ 3 : CORRECTIFS MINEURS** *(À corriger si temps disponible - Impact faible, effort variable)*

#### **3.1. Cohérence des Données**
| **ID** | **Faille** | **Localisation** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|----------------------|------------|
| **DATA-001** | Doublons d'actions ISR | `actions.json` | Fusionner les actions similaires (ex: `isr_etomidate_rocu` → `isr_etomidate` avec option `curare: "rocu"`). | ⭐⭐⭐ |
| **DATA-002** | `pente` manquant pour certaines actions | `actions.json` | Ajouter des `pente` pour les actions sans effet continu (ex: `adrenaline_ivse`). | ⭐⭐ |

---

#### **3.2. Expérience Utilisateur**
| **ID** | **Faille** | **Localisation** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|----------------------|------------|
| **UX-001** | Pas de calcul automatique de la PAM | Moteur | Ajouter `PAM = (PAS + 2*PAD)/3` dans l'affichage. | ⭐⭐ |
| **UX-002** | Pas de résumé des erreurs dans le débriefing | `peaufinage.md` | Ajouter une section **"Pièges évités / Déclenchés"** avec :<br>- Liste des CI respectées.<br>- Liste des complications déclenchées. | ⭐⭐ |
| **UX-003** | Temps de réponse non adapté | `sc_*.json` | Augmenter `chrono` pour les scénarios complexes (ex: `sc_eme` → 120s au lieu de 90s). | ⭐ |

---

#### **3.3. Améliorations Médicales**
| **ID** | **Faille** | **Localisation** | **Correctif Proposé** | **Effort** |
|-------|------------|----------------|----------------------|------------|
| **MED-004** | Pas de mention de la **normocapnie** dans `sc_eme` | `sc_eme.json` | Ajouter `"gds": "Cible : PaCO2 35-40 mmHg (normocapnie)"` dans les résultats. | ⭐ |
| **MED-005** | `tdm_ap` dans `sc_choc_septique` : pas de mention du **risque de transport** | `sc_choc_septique.json` | Ajouter une note : "⚠️ Transport risqué : stabiliser d'abord (PAM ≥ 65 mmHg)". | ⭐ |

---

## 📊 Synthèse des Priorités

| **Priorité** | **Nombre de Correctifs** | **Effort Total Estimé** | **Impact** | **Recommandation** |
|-------------|------------------------|------------------------|------------|------------------|
| 🔴 **Critique** | 8 | ~10h | ⭐⭐⭐⭐⭐ | **À faire en premier** |
| 🟠 **Majeure** | 9 | ~15h | ⭐⭐⭐⭐ | **À faire ensuite** |
| ⚪ **Mineure** | 10+ | ~20h | ⭐⭐⭐ | **Si temps disponible** |

---

## 🛠️ Plan d'Action Proposé

### **Phase 1 : Correctifs Critiques (1 semaine)**
**Objectif** : Éliminer les risques de **mauvaise pratique médicale** et d'**actions dangereuses autorisées**.

| **Tâche** | **Fichiers à Modifier** | **Livrable** | **Validation** |
|----------|----------------------|-------------|--------------|
| Corriger les CI manquantes (`CI-001` à `CI-005`) | `terrains.json`, `sc_choc_septique.json` | CI bloquées + alternatives promues | ✅ Tests unitaires |
| Mettre à jour les protocoles obsolètes (`PROT-001`, `PROT-004`) | `sc_choc_septique.json`, `complications.json` | Protocoles conformes à SSC 2021 | ✅ Relecture médicale |
| Corriger les effets physiologiques (`EFF-001`) | `actions.json` | Effets différenciés pour RL vs NaCl | ✅ Simulation réaliste |

---

### **Phase 2 : Correctifs Majeurs (2 semaines)**
**Objectif** : Améliorer la **qualité pédagogique** et la **logique des scénarios**.

| **Tâche** | **Fichiers à Modifier** | **Livrable** | **Validation** |
|----------|----------------------|-------------|--------------|
| Ajouter un ordre de priorité (`LOG-001`, `LOG-003`) | `sc_*.json`, Interface | Actions classées par priorité | ✅ Tests utilisateurs |
| Améliorer le contenu médical (`MED-001` à `MED-003`) | `actions.json` | Descriptions complètes | ✅ Relecture |
| Corriger la gestion des complications (`COMP-001` à `COMP-003`) | `complications.json` | Complications réalistes | ✅ Simulation |

---

### **Phase 3 : Correctifs Mineurs (Optionnel)**
**Objectif** : **Peaufiner** l'expérience et la cohérence.

| **Tâche** | **Fichiers à Modifier** | **Livrable** | **Validation** |
|----------|----------------------|-------------|--------------|
| Fusionner les doublons (`DATA-001`) | `actions.json` | Actions unifiées | ✅ Tests |
| Améliorer l'UX (`UX-001` à `UX-003`) | Moteur, Interface | Meilleure lisibilité | ✅ Feedback utilisateurs |
| Ajouter des détails médicaux (`MED-004`, `MED-005`) | `sc_*.json` | Scénarios plus complets | ✅ Relecture |

---

## 📝 Annexes

### **A. Liste des Fichiers Concernés**
```
Dechocage/
├── actions.json          # Correctifs : CI-001, PROT-001, EFF-001, LOG-002, MED-001, DATA-001
├── complications.json   # Correctifs : CI-001, PROT-004, COMP-001, COMP-002
├── terrains.json         # Correctifs : CI-001 à CI-005, PROT-004
├── sc_choc_septique.json # Correctifs : PROT-001, PROT-002, LOG-001
├── sc_eme.json           # Correctifs : PROT-003, MED-004
└── peaufinage.md         # Correctifs : UX-002
```

---

### **B. Sources Médicales à Consulter**
| **Source** | **Lien** | **Utilisation** |
|-----------|----------|----------------|
| Surviving Sepsis Campaign 2021 | [SSC 2021](https://www.sccm.org/SurvivingSepsisCampaign/Guidelines) | Choc septique, remplissage, amines |
| European Resuscitation Council 2021 | [ERC 2021](https://www.erc.edu/) | RCP, anaphylaxie, EME |
| SFAR 2017 (Intubation Difficile) | [SFAR 2017](https://www.sfar.org/) | Voies aériennes, CICO |
| SFAR/SFA 2011 (Anaphylaxie) | [SFAR/SFA 2011](https://www.sfar.org/) | Allergies, glucagon |
| UK Kidney Association 2020 | [UKKA 2020](https://www.renal.org/) | Hyperkaliémie, IRC |
| BTS 2017 (BPCO) | [BTS 2017](https://www.brit-thoracic.org.uk/) | Oxygène, BPCO |

---

### **C. Exemple de Correctif Appliqué**
**Faille** : `CI-001` (Allergie aux bêta-lactamines → `atb_c3g_amika` non CI).

**Avant** (`sc_choc_septique.json`) :
```json
"atb_c3g_amika": {
  "note": "indispensable",
  "alt": "atb",
  "why": "Choc septique : antibiotique dans l'heure (SSC 2021)."
}
```

**Après** (`sc_choc_septique.json` + `terrains.json`) :
```json
// Dans terrains.json
{
  "id": "allergie_betalactamines",
  "ci": [
    {
      "classe": "betalactamine",
      "complication": "anaphylaxie",
      "why": "Anaphylaxie à la ceftriaxone : toute bêta-lactamine est contre-indiquée."
    }
  ],
  "promeut": [
    {
      "id": "atb_aztreo_amika",
      "dans": ["choc_septique"],
      "why": "Allergie aux bêta-lactamines : aztréonam + amikacine en alternative."
    }
  ]
}

// Dans sc_choc_septique.json
"atb_c3g_amika": {
  "note": "contre_indique",
  "why": "Allergie aux bêta-lactamines : risque d'anaphylaxie."
},
"atb_aztreo_amika": {
  "note": "indispensable",
  "alt": "atb",
  "why": "Choc septique + allergie : aztréonam + amikacine (SPILF 2018)."
}
```

---

## 🔄 Suivi des Correctifs

| **ID** | **Statut** | **Date** | **Responsable** | **Commentaires** |
|--------|------------|----------|----------------|-----------------|
| CI-001 | ⏳ | - | - | À faire en priorité |
| CI-002 | ⏳ | - | - | À faire en priorité |
| CI-003 | ⏳ | - | - | À faire en priorité |
| PROT-001 | ⏳ | - | - | À faire en priorité |
| PROT-004 | ⏳ | - | - | À faire en priorité |

---

## 📌 Conclusion

Ce document propose une **feuille de route claire** pour corriger les failles du module Dechocage, avec :
1. **8 correctifs critiques** à appliquer en premier (impact maximal).
2. **9 correctifs majeurs** pour améliorer la qualité pédagogique.
3. **10+ correctifs mineurs** pour peaufiner l'expérience.

**Prochaine étape** : Valider ce document avec toi, puis commencer par les correctifs **🔴 Critiques** (Phase 1).

---

*Document généré par Vibe Code - À valider et compléter avec l'équipe médicale.*
