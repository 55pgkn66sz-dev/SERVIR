# Candidatures créées depuis un PDF Agorh@

Documentation technique de la fonction « Depuis un PDF (Agorh@) » du widget SERVIR (`SERVIR.html`, page **Candidats**).

## Principe

1. Un PDF Agorh@ est déposé dans **Candidats › Créer une candidature › Depuis un PDF (Agorh@)**.
2. Le texte est extrait dans le navigateur (pdf.js) : **un PDF = un candidat**.
3. Un aperçu signale les erreurs et les doublons ; l'utilisateur confirme.
4. Une ligne est créée par candidat (colonne `Source` = `Agorh@`) et un courriel s'ouvre dans la messagerie avec le lien du formulaire Grist d'affectation souhaitée.
5. Tant que le candidat n'a pas répondu, sa ligne porte un **filigrane bleu « intégré depuis Agorh@ »**.
6. Les réponses du formulaire Grist sont lues dans leur table : affectation de référence → reportée dans la ligne (le filigrane disparaît) ; autre valeur → la ligne est supprimée.

Tout s'exécute côté navigateur, dans le widget : aucun serveur à installer.

## Installation et configuration

### Dans Grist
- Le widget reste un widget personnalisé sur la table des candidats. Deux colonnes sont **créées automatiquement** au premier chargement si elles manquent (droit de modification de la structure requis) :
  - `Source` (Texte) : vaut `Agorh@` pour les candidatures créées depuis un PDF ;
  - `Réponse Agorh` (Date) : date d'intégration de la réponse. Une ligne `Source = Agorh@` **sans** date est « en attente » (filigrane bleu).
- Créer le **formulaire Grist** (widget Formulaire) sur une table de réponses du **même document**. Colonnes attendues :
  | Colonne | Rôle |
  |---|---|
  | `Affectation` | Choix du candidat (obligatoire) |
  | `N° candidature` et/ou `NID` et/ou `Email` | Clé de rapprochement avec le candidat (au moins une) |
  | `NOM`, `Prénom` | Rapprochement de dernier recours |
  | `Traité` (Booléen, facultatif) | Posée à vrai par le widget une fois la réponse traitée |
- Les choix du formulaire (GGD77, GGD78, GGD91, GGD95, SAJ, CRR + gendarmeries spécialisées) sont gérés dans Grist : liste évolutive, sans modification du widget.

### Dans le back office (page Réservations ; rubrique **Courriels**)
- **Candidatures Agorh@ : formulaire d'affectation**
  - *Lien du formulaire Grist* (`https://…`), inséré dans le courriel par `{{LIEN_FORMULAIRE}}`.
  - *Affectations de référence* : `GGD77, GGD78, GGD91, GGD95, SAJ, CRR` par défaut. **Toute autre réponse supprime la candidature.**
  - *Table des réponses* : vide = détection automatique (nom contenant `agorh` + `rep`/`form`/`affect`, ou `affectation` + `souhait`/`reponse`).
- **Modèles de courriels › Candidatures · Agorh@** : objet et corps du courriel, au même endroit que les autres modèles. Variables : `{{CIVILITE}}`, `{{NOM}}`, `{{PRENOM}}`, `{{IDENTITE}}`, `{{N_CANDIDATURE}}`, `{{NID}}`, `{{LIEN_FORMULAIRE}}`.
- Les réglages sont enregistrés avec ceux des réservations (bouton d'enregistrement du back office).

### Dépendances
- `pdf.js 3.11.174` chargé à la demande depuis `cdn.jsdelivr.net` (script et worker). Le réseau doit autoriser ce domaine ; sinon le message « la bibliothèque de lecture des PDF n'a pas pu être chargée » s'affiche.

## Fonctionnement détaillé

### Extraction (`agPdfLines`, `agParseLines`, `agBuild`)
- Les éléments de texte sont regroupés par ordonnée (tolérance 3 pt) et triés par abscisse ; un écart horizontal > 12 pt devient une tabulation.
- Trois formes sont reconnues : `Libellé : valeur`, `Libellé<tabulation>valeur`, et libellé seul suivi de sa valeur à la ligne.
- Les libellés sont comparés sans accents ni ponctuation (`AG_FIELDS`) : Nom, Prénom, Civilité, Date de naissance, Lieu de naissance, Courriel/E-mail, Téléphone, Adresse, Code postal, Ville, N° candidature, NID, N° de sécurité sociale, Origine, Profession, PMG. **Pour ajouter un libellé, compléter l'expression régulière correspondante.**
- Un seul candidat par PDF (sur une ou plusieurs pages) : si un libellé est répété, la première valeur est conservée.
- Les valeurs sont converties comme à l'import .csv (`csvParseValue` : dates JJ/MM/AAAA, civilité, listes de choix). `Origine` accepte « Civil », « VCA »… (`agOrigin`).
- Champs obligatoires : Nom, Prénom, Courriel valide, et NID **ou** N° candidature. Une valeur invalide hors champs clés est ignorée avec avertissement.

### Doublons (`agMarkDups`)
- Même règle que la page Candidats (`dupKeys`) : même NID, même N° candidature, ou à défaut mêmes nom, prénom et date de naissance. Comparaison avec les fiches existantes.
- Un doublon n'est créé que si la case « Créer quand même cette candidature » est cochée ; il est alors marqué `Doublon`.

### Courriel (`RSV.agorhInvite`)
- Ouvre un message `mailto:` (copie et « répondre à » repris des réglages des réservations ; destinataires supplémentaires du modèle `rsv:x:agorh` pris en compte).
- Un seul candidat créé : le courriel s'ouvre directement. Plusieurs : un bouton « Écrire » par candidat (un navigateur n'ouvre pas plusieurs messageries à la suite).
- Sans lien de formulaire valide ou sans adresse valide, un message explicite s'affiche.

### Intégration des réponses (`integrateAgorh`)
- Déclenchée **après chaque chargement de la liste** (dès que les réglages du back office sont lus), **toutes les 5 minutes** tant que la page est ouverte, et par le bouton **« Réponses Agorh@ »**. Un widget n'a pas de tâche de fond : les réponses sont donc intégrées à l'ouverture de la page, pas en temps réel.
- Seules les candidatures « en attente » (`Source = Agorh@`, sans `Réponse Agorh`) sont concernées.
- Rapprochement réponse → candidat : N° candidature, puis NID, puis courriel, puis nom + prénom ; une clé n'est retenue que si **un seul** candidat en attente correspond.
- Comparaison de l'affectation sans casse, espaces ni ponctuation (`GGD 78` = `GGD78`).
- Référence : `Affectation` ← valeur de référence canonique, `Réponse Agorh` ← date du jour. Hors référence : **suppression de la ligne** (`deleteRow`). Un bandeau récapitule les noms intégrés et supprimés.
- Idempotence : une candidature traitée n'est plus « en attente » ; la colonne `Traité` de la table des réponses (si présente) est posée à vrai. **Sans colonne `Traité`**, une réponse ancienne pourrait être réappliquée à une candidature recréée avec la même clé : ajouter la colonne.

### Filigrane
- Ligne en attente : fond bleu et mention « intégré depuis Agorh@ » à droite (`.row-agorh`, `.agorh-wm`). Entrée ajoutée à la légende « i » de la liste.

## Cas limites

| Cas | Comportement |
|---|---|
| PDF scanné ou sans texte | « PDF illisible : aucun texte trouvé… » |
| PDF corrompu | « PDF illisible : <détail pdf.js> » |
| Fichier non PDF | « Le fichier n'est pas un PDF » |
| Aucun libellé reconnu | « PDF illisible : aucune information de candidature reconnue… » |
| Candidat sans courriel, sans NID/N° candidature | Ligne « Erreur » dans l'aperçu, non créée |
| Doublon (fiche existante) | Signalé, créé seulement après confirmation |
| Colonnes `Source` / `Réponse Agorh` absentes | Erreur explicite (créées au prochain chargement) |
| Lien du formulaire non renseigné | Candidature créée, courriel refusé avec message |
| Réponse sans candidat en attente correspondant | Ignorée (comptée dans le message) |
| Clé ambiguë (plusieurs candidats correspondent) | Ignorée : rien n'est supprimé par erreur |
| Plusieurs réponses du même candidat | La plus ancienne (identifiant de ligne) est appliquée, les suivantes ne correspondent plus à un candidat en attente |
| Table des réponses introuvable | Message au clic sur « Réponses Agorh@ » ; silence en automatique |
| Réglages du back office pas encore chargés | Intégration automatique différée (jusqu'à 15 tentatives d'1,2 s) ; jamais de suppression avec des réglages par défaut |

## Tests à prévoir (recette)

1. **Extraction** : PDF Agorh@ réel (une ou plusieurs pages) ; ajuster `AG_FIELDS` aux libellés réels.
2. **Création** : candidat valide ; doublon (NID, N° candidature, nom + DDN) refusé puis confirmé ; candidat sans courriel .
3. **Courriel** : un candidat (ouverture directe) et plusieurs (boutons) ; lien absent ; variables du modèle.
4. **Filigrane** : présent après création, absent après réponse, présent dans la légende.
5. **Réponses** : GGD77/78/91/95, SAJ, CRR (avec variantes de casse et d'espaces) ; valeur hors liste (suppression) ; réponse sans candidat ; clé ambiguë ; second passage (aucun effet) ; liste de référence modifiée dans le back office.
6. **Droits Grist** : document sans droit de modification de la structure (colonnes à créer à la main) ; formulaire sans colonne `Traité`.
7. **Réseau** : domaine `cdn.jsdelivr.net` bloqué (message d'erreur lisible).

Tests déjà passés en mode démonstration (navigateur, PDF généré) : extraction, aperçu (prêt / doublon / erreur), création, filigrane, intégration (1 réponse reportée, 1 suppression, 1 en attente), second passage sans effet, PDF scanné / corrompu / non PDF / sans libellé, courriel avec variables.

## Limites connues

- Les libellés du PDF réel d'Agorh@ n'étaient pas disponibles : les expressions de `AG_FIELDS` sont génériques et à valider sur un vrai fichier.
- Pas d'OCR : un PDF scanné est refusé.
- L'intégration n'a lieu que page ouverte (chargement, minuteur de 5 min, bouton).
- Le formulaire doit écrire dans une table du **même document Grist** que le widget.
