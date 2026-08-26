# Stratégie et dispositif qualité — Application mobile de paiement

## 1. Contexte et objectifs

L'application compte environ 200 000 utilisateurs et présente actuellement :
- aucun test automatisé ;
- une recette entièrement manuelle de 5 jours avant chaque livraison ;
- 3 incidents de production en 6 mois ;
- un incident critique de double débit ayant touché des centaines de clients ;
- une échéance réglementaire de 3 mois ;
- un objectif de passage d'une livraison trimestrielle à une livraison mensuelle ;
- une équipe composée de 8 développeurs et 2 testeurs manuels.

### Enjeux

La priorité n'est pas d'automatiser toute la couverture en trois mois. Il faut réduire rapidement le risque sur les parcours financiers critiques, sécuriser la version réglementaire et construire un socle permettant ensuite des livraisons mensuelles.

Les principes retenus sont :
1. **Risque métier avant couverture exhaustive**.
2. **Automatiser en priorité les tests répétitifs, stables et critiques**.
3. **Déplacer les contrôles vers les niveaux les moins coûteux** : unitaires puis API/intégration, avant UI.
4. **Conserver une validation humaine pour l'exploratoire, l'UX, les parcours complexes et les contrôles réglementaires nécessitant un jugement.**
5. **Aucune régression critique connue ne doit être acceptée en production.**

---

# Tâche 1 — Stratégie de test

## 2. Périmètre prioritaire

### Priorité P0 — Risque financier et réglementaire

À automatiser en premier :
- authentification et gestion de session ;
- création et validation d'une transaction ;
- débit/crédit ;
- prévention du double débit ;
- idempotence d'une transaction rejouée ;
- gestion des timeouts et retries ;
- annulation/rollback ;
- consultation du solde ;
- historique des transactions ;
- statuts transactionnels : pending, success, failed, cancelled ;
- frais et commissions ;
- limites de transaction ;
- contrôle des doublons ;
- cohérence entre transaction, solde et historique ;
- notifications liées au paiement ;
- contrôle des droits et des rôles ;
- règles introduites par la nouvelle version réglementaire.

Le **double débit** devient un axe de test dédié. Les scénarios doivent notamment vérifier :
- double clic utilisateur ;
- double appel API ;
- retry réseau ;
- timeout côté client ;
- retry côté serveur ;
- répétition avec le même identifiant/idempotency key ;
- réception répétée d'un même événement ;
- concurrence de deux demandes identiques ;
- reprise après indisponibilité d'un service externe.

### Priorité P1 — Parcours fonctionnels majeurs

À automatiser ensuite :
- inscription et activation ;
- récupération de compte ;
- consultation du profil ;
- consultation du solde ;
- historique ;
- recherche/filtrage des opérations ;
- notifications ;
- opérations courantes non financières critiques.

### Priorité P2 — Fonctionnalités secondaires

À traiter principalement en manuel pendant les trois mois :
- éléments visuels peu risqués ;
- cas rares ;
- fonctionnalités peu utilisées ;
- scénarios très instables ;
- compatibilités matérielles difficiles à maintenir automatiquement.

---

## 3. Répartition entre les niveaux de test

La stratégie cible une pyramide de tests.

| Niveau | Cible | Part indicative | Automatisation |
|---|---|---:|---|
| Tests unitaires | règles métier, calculs, validation, idempotence | 50–60 % | Très forte |
| Tests intégration/API | services, DB, paiement, transactions, contrats API | 25–35 % | Très forte |
| Tests UI/mobile | parcours critiques de bout en bout | 10–15 % | Sélective |
| Tests manuels exploratoires | UX, scénarios nouveaux, risques non modélisés | Complément | Manuelle |

Les pourcentages sont des objectifs de répartition et non une obligation de mesurer mécaniquement la couverture par nombre de tests.

### Pourquoi privilégier API et intégration ?

Les tests UI mobiles sont plus coûteux et plus fragiles. Pour une application de paiement, une grande partie du risque critique se situe dans les règles métier, les API, les transactions et les interactions avec les systèmes externes.

Un test API peut vérifier rapidement :
- le montant débité ;
- le solde avant/après ;
- le statut de transaction ;
- l'identifiant de transaction ;
- l'idempotence ;
- la réponse en cas de retry ;
- la cohérence avec la base.

Le test UI doit surtout confirmer que le parcours utilisateur fonctionne correctement de bout en bout.

---

## 4. Ce qui doit rester manuel

Pendant les trois mois, il serait contre-productif d'essayer d'automatiser tout le catalogue.

À conserver en manuel :
- tests exploratoires ;
- tests UX et ergonomie ;
- validation visuelle ;
- scénarios nouveaux ou fortement modifiés ;
- tests de compatibilité sur les appareils prioritaires ;
- tests nécessitant une intervention physique ou un équipement spécifique ;
- validation finale de certains changements réglementaires ;
- tests d'acceptation métier ;
- investigation d'anomalies complexes ;
- tests de scénarios exceptionnels difficiles à stabiliser.

Les deux testeurs manuels deviennent progressivement responsables de la stratégie de régression, de l'exploration et de la conception des scénarios automatisables, au lieu d'exécuter uniquement une recette répétitive de cinq jours.

---

## 5. Plan de mise en œuvre sur trois mois

### Semaines 1–2 — Stabiliser et prioriser

Objectifs :
- cartographier les parcours critiques ;
- construire une matrice de risques ;
- identifier les exigences réglementaires ;
- définir les critères de mise en production ;
- analyser les 3 incidents historiques ;
- reproduire et sécuriser en priorité le scénario de double débit ;
- choisir l'outillage ;
- préparer les environnements et données de test ;
- établir une première suite de smoke tests API.

Livrable principal : **Risk-Based Test Plan + matrice de traçabilité réglementaire.**

### Semaines 3–4 — Socle automatisé

Créer :
- structure du framework ;
- données de test contrôlées ;
- tests unitaires sur les règles financières ;
- tests API sur les transactions critiques ;
- premiers tests d'idempotence ;
- tests de non-régression des incidents connus ;
- exécution automatique dans la CI.

Objectif : disposer d'une première suite fiable exécutée à chaque changement.

### Semaines 5–8 — Couverture critique

Priorité :
- paiement ;
- débit/crédit ;
- solde ;
- historique ;
- frais ;
- limites ;
- erreurs et retries ;
- authentification ;
- autorisation ;
- intégrations externes ;
- exigences réglementaires.

Ajouter quelques parcours UI critiques, mais ne pas transformer la suite en énorme catalogue de tests UI.

### Semaines 9–10 — Durcissement

- tests de régression ;
- tests de concurrence ;
- tests de reprise après erreur ;
- tests de performance ciblés ;
- tests de sécurité applicative ;
- tests sur appareils représentatifs ;
- validation réglementaire ;
- correction et stabilisation des tests flaky.

### Semaines 11–12 — Release candidate

- gel des fonctionnalités non indispensables ;
- exécution complète de la régression automatisée ;
- campagne manuelle ciblée ;
- validation réglementaire ;
- tests de non-régression des incidents historiques ;
- validation des métriques ;
- préparation du plan de rollback ;
- Go/No-Go formel.

---

## 6. Organisation de l'équipe

### Les 8 développeurs

Ils doivent être responsables de la qualité au niveau du code :
- tests unitaires ;
- tests d'intégration ;
- correction des anomalies ;
- maintien des tests automatisés ;
- analyse des causes racines ;
- revues de code ;
- contribution aux tests API.

La qualité ne doit pas être entièrement déléguée aux deux testeurs.

### Les 2 testeurs

Répartition recommandée :
- Testeur 1 : stratégie, tests métier/réglementaires, API, régression.
- Testeur 2 : automatisation UI/API, exploratoire, compatibilité et validation finale.

Cette répartition reste flexible selon les compétences.

### Principe de fonctionnement

Toute fonctionnalité doit arriver chez les testeurs avec un premier niveau de tests développeurs déjà présent.

---

# Tâche 2 — Dispositif qualité conditionnant la mise en production

## 7. Critères de passage

Une version ne peut être mise en production que si les conditions suivantes sont réunies.

### Critères bloquants

- 100 % des tests critiques P0 passent ;
- aucune anomalie critique ouverte ;
- aucune anomalie majeure non acceptée formellement sur un parcours financier critique ;
- aucune régression connue sur les incidents historiques ;
- tous les tests réglementaires applicables sont passants ;
- tests d'idempotence et de prévention du double débit passants ;
- tests de cohérence transaction/solde/historique passants ;
- smoke tests passants ;
- tests de sécurité critiques passants ;
- rollback testé et opérationnel ;
- monitoring et alerting opérationnels ;
- données et comptes de test validés ;
- validation métier et QA obtenues ;
- Go/No-Go formel enregistré.

### Critères de qualité de la suite automatisée

Les tests critiques doivent être suffisamment stables pour être utilisés comme garde-fou de CI/CD.

Un test flaky ne doit pas être ignoré : il doit être corrigé, isolé avec justification temporaire ou supprimé s'il n'a plus de valeur.

---

## 8. Classification des anomalies

| Sévérité | Exemple | Règle |
|---|---|---|
| S1 Critique | double débit, perte d'argent, corruption de données, faille critique | Bloque immédiatement |
| S2 Majeure | fonctionnalité métier principale inutilisable | Bloque sauf dérogation formelle |
| S3 Mineure | fonctionnalité secondaire dégradée | Peut être acceptée selon risque |
| S4 Cosmétique | problème visuel sans impact métier | Non bloquant |

Une anomalie S1 doit déclencher :
1. blocage de la release ;
2. analyse de cause racine ;
3. correction ;
4. test de reproduction ;
5. test de non-régression ;
6. vérification des impacts sur les autres composants ;
7. validation avant reprise du processus de release.

---

## 9. Gestion des données de test

Les données de paiement sont sensibles. Il faut éviter d'utiliser des données réelles en environnement de test.

Principes :
- jeux de données synthétiques ;
- comptes de test dédiés ;
- montants contrôlés ;
- utilisateurs et rôles prédéfinis ;
- données réinitialisables ;
- génération automatisée des données lorsque possible ;
- séparation stricte test/préproduction/production ;
- aucune donnée bancaire réelle dans les logs ou rapports ;
- anonymisation/pseudonymisation si des données réalistes sont nécessaires ;
- nettoyage après campagne.

### Jeu minimal de données

Prévoir notamment :
- utilisateur actif ;
- utilisateur bloqué ;
- compte sans solde ;
- solde insuffisant ;
- compte avec limite atteinte ;
- transactions réussies ;
- transactions échouées ;
- transactions en attente ;
- transaction répétée ;
- transaction concurrente ;
- compte avec historique important ;
- cas de timeout ;
- cas de retry ;
- cas de remboursement/annulation.

---

## 10. Indicateurs de suivi

Les KPI doivent mesurer le risque et la capacité à livrer, pas seulement le nombre de tests.

### Qualité produit

- nombre d'incidents production ;
- nombre d'incidents S1/S2 ;
- taux de défauts échappés en production ;
- taux de réouverture des anomalies ;
- répartition des défauts par module ;
- taux de réussite des tests critiques.

### Automatisation

- nombre de tests automatisés ;
- couverture des parcours P0 ;
- taux de réussite de la suite ;
- durée de la suite ;
- taux de tests flaky ;
- fréquence d'exécution.

### Livraison

- durée de recette ;
- fréquence de livraison ;
- taux de releases réussies ;
- taux de rollback ;
- lead time entre correction et mise en production.

### Exemple de tableau de bord hebdomadaire

| KPI | Cible à 3 mois |
|---|---:|
| Parcours P0 automatisés | ≥ 90 % |
| Tests critiques passants avant release | 100 % |
| Anomalies S1 ouvertes à la release | 0 |
| Régressions liées aux incidents historiques | 0 |
| Tests réglementaires passants | 100 % |
| Tests flaky critiques | 0 |
| Rollback opérationnel | Oui |
| Données de test sensibles exposées | 0 |

Les seuils doivent être ajustés après mesure initiale, mais les exigences de zéro S1 et 100 % sur les contrôles réglementaires/critiques sont non négociables.

---

## 11. CI/CD et qualité continue

Le passage à une livraison mensuelle ne doit pas simplement raccourcir la recette manuelle.

Pipeline recommandé :

**Commit → Build → Tests unitaires → Tests intégration/API → Analyse qualité → Tests de sécurité → Smoke tests → Environnement de recette → Tests UI critiques → Validation QA/métier → Production**

À chaque pull request :
- compilation/build ;
- tests unitaires ;
- tests d'intégration ciblés ;
- analyse statique ;
- tests de sécurité de base.

À chaque release candidate :
- suite de régression complète ;
- tests réglementaires ;
- tests critiques UI/API ;
- tests de performance ciblés ;
- validation manuelle.

---

## 12. Stratégie spécifique contre le double débit

Compte tenu de l'incident historique, l'idempotence doit être traitée comme une exigence qualité de premier niveau.

Pour une même demande de paiement :
- une clé d'idempotence unique doit permettre d'identifier la demande ;
- plusieurs requêtes identiques ne doivent produire qu'un seul débit ;
- un retry après timeout ne doit pas créer une deuxième transaction ;
- une transaction déjà finalisée doit retourner son résultat connu plutôt que recréer une opération ;
- les traitements concurrents doivent être testés ;
- le système doit garantir la cohérence entre montant débité, solde et historique.

Des tests de concurrence et de répétition doivent être exécutés automatiquement dans la CI ou au minimum dans la campagne de validation avant chaque release concernée.

---

## 13. Passage de trimestriel à mensuel

Le changement de cadence doit être progressif.

### Mois 1
Objectif : sécuriser la base et réduire le risque.

- automatisation P0 ;
- pipeline CI ;
- tests des incidents historiques ;
- première réduction de la recette manuelle.

### Mois 2
Objectif : rendre la régression répétable.

- extension API/intégration ;
- quelques tests UI critiques ;
- réduction de la durée de recette ;
- premières releases mensuelles sous contrôle.

### Mois 3
Objectif : sécuriser la version réglementaire et industrialiser.

- régression automatisée stable ;
- validation réglementaire ;
- critères Go/No-Go ;
- rollback ;
- monitoring ;
- documentation.

Après la version réglementaire, l'équipe pourra augmenter progressivement la cadence et la couverture sans chercher à automatiser artificiellement 100 % des tests.

---

# 14. Décision finale

Avec trois mois disponibles, le mauvais choix serait de tenter une automatisation exhaustive de l'application.

Le choix recommandé est une **stratégie risk-based**, centrée sur les flux financiers et réglementaires. Les développeurs prennent en charge les tests unitaires et d'intégration ; les testeurs concentrent leur effort sur l'analyse de risques, les scénarios métier, l'automatisation des API/UI critiques, l'exploration et la validation réglementaire.

Le point le plus critique est le risque financier lié au double débit. Il doit être couvert par des tests d'idempotence, de retry, de concurrence et de cohérence transactionnelle.

La mise en production est conditionnée par des critères objectifs : **zéro S1, 100 % des contrôles réglementaires et P0 passants, absence de régression sur les incidents historiques, données de test maîtrisées, rollback et monitoring opérationnels**.

Cette approche permet de satisfaire l'échéance réglementaire de trois mois tout en posant les fondations nécessaires au passage durable d'une livraison trimestrielle à une livraison mensuelle.

