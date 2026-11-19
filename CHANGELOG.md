# Journal des versions

## 3.0.3
- Dans Grist, une colonne ou une liste de choix modifiée par le widget (nouvelle PMG créée dans « Gérer les PMG », colonnes ajoutées automatiquement) n'apparaissait qu'après environ 45 secondes : la mémoire des métadonnées des colonnes n'était pas vidée après la modification. Elle est maintenant vidée après chaque modification de structure.

## 3.0.2
- Affichage rétabli : des règles de style perdues lors de la fusion de la version 3.0.0 sont restaurées (barre « Actions » des fiches, boutons des barres d'outils, ordre des boutons de validation, barre de progression d'envoi des courriels, étape de confirmation de l'admission groupée, crayon d'édition sur écran tactile).

## 3.0.1
- Modèles de documents des pages Réservistes (PPE, contrats) : les variables à une seule accolade, par exemple `{N_ESR}`, `{Date_contrat}`, `{Identite_complete}`, `{Date_debut}` et `{Fin_PPE}`, sont maintenant remplacées comme celles à double accolade. Une variable inconnue reste telle quelle.

## 3.0.0
Version fusionnée de trois lignées de travail.
- Réglages configurables en back office (nom du service, états de signature, références d'actes…).
- Page « Gérer les PMG » (cartes par PMG), seuils de stock, formations administratives.
- Création de candidatures depuis un PDF Agorh@ et intégration des réponses du formulaire d'affectation.
- Barre d'actions dans les fiches candidat et réserviste.
- Règles VCA / AM (VMP au lieu de VEMI), filigranes de liste (VCA/AM, limite d'âge, Agorh@).
- Recherche de doublons par NID ou N° candidature.
- Champs employeur, N° chrono employeur et lettre d'information employeur.
- Données de démonstration et exemples neutralisés.
