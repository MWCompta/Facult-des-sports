# Contexte projet — Outil covoiturage L1 STAPS (Nancy/Épinal)

À coller/attacher en premier message d'une nouvelle conversation. Voir note importante en bas.

## Qui / quoi
Valentin, auto-entrepreneur Nancy (APA, formation EHPAD, baby gym), conseiller municipal, développeur web. Projet : outil de covoiturage pour ~400-515 étudiant·es L1 STAPS, 20 groupes fixes à l'année (16 Nancy + 4 Épinal). Destinations : Nancy {48.6671246, 6.1565395}, Épinal {48.1843027, 6.4548672}.

Préférences Valentin : direct, concis, zéro flatterie, honnêteté sur l'incertitude, meilleur rapport qualité-prix. Écrit en français.

## Fichiers du projet
- **carte_groupes_nancy.html** — fichier principal, autonome (HTML+CSS+JS), ~3000 lignes. À RÉATTACHER pour toute modification (ce résumé ne contient pas son code).
- **tests_covoiturage.html** — 21 tests automatiques (unitaires : distance, éligibilité ramassage, échappement, détection refus covoiturage, grandes lignes algo). NE COUVRE PAS : minimum conducteurs, mode "tout le monde", mode "mauvais temps", destination temporaire, recréation groupes par proximité, mise en page PDF.
- **audit_covoiturage.md** — daté 5 sept 2026, partiellement obsolète (échappement HTML depuis fait, import réel de 515 réponses depuis testé).
- **Passation_covoiturage_STAPS.docx** — document de passation créé le 9 sept (voir contenu ci-dessous).

## Structure des données importées (colonnes A-Q, positions fixes — voir "Point ouvert" plus bas)
C=Nom, D=Prénom, E=Groupe, F/G/H=rue/CP/ville (adresse principale), I="adresse principale ?" (bascule vers J/K/L si Non), J/K/L=adresse secondaire, M=mode déplacement habituel, N=places dispo, O=stabilité abonnement transport, P=déplacement secours ("plan B"), Q=souhait covoiturage. 9 modes de transport avec profils CO2/coût (voiture = ADEME + barème fiscal, solide ; autres modes = estimations moins vérifiées).

## Fonctionnalités actuelles
- **Import + géocodage** : service Géoplateforme (remplace l'ancienne API Adresse gouv., décommissionnée fin janv. 2026), limite de débit 8/s + 2 tentatives auto + bouton retry échecs seuls, détection adresses non géolocalisées + doublons probables.
- **Algorithme covoiturage** : ramassage détour max 15 min, consolidation Clarke & Wright, plusieurs essais randomisés, repli à vol d'oiseau si routage en panne. Optimise le CO2 (temps/coût affichés, ne pilotent pas).
- **Recréation groupes par proximité** : k-means à taille contrainte. `MIN_CAR_PER_GROUP = 3` (constante ajustable ; Valentin a choisi 3 plutôt que 5). Une première passe priorise les personnes en capacité de conduire (mode principal OU secours, `hasCarCapability`) pour atteindre ce minimum par groupe avant de compléter normalement. Vérification de faisabilité en amont (avertissement chiffré dans la boîte de confirmation si insuffisant). Rapport final : liste par groupe (nb personnes + nb conducteurs, ⚠️ si sous le minimum).
- **Carte** : 6 fonds Esri gratuits sans clé (OpenStreetMap couleur, Relief, Clair, Sombre, Satellite, Rues) — CARTO écarté (clé imposée fin août 2026) ; filtres Nancy/Épinal ; destination temporaire par groupe/bassin.
- **Modes spéciaux** : "tout le monde" (élargit à tous modes), "mauvais temps" (bascule piétons vers plan B).
- **Export** : PDF/PNG individuel + groupé, bilan CO2/coût/distance, repli PNG auto si PDF échoue, "exporter tous les groupes tracés".
- **Historique interne** (bouton 📋) : réorganisé le 9 sept en 8 thèmes, sections repliables fermées par défaut, chronologie conservée par thème. Thèmes : Données/import/géocodage, Algorithme de covoiturage, Groupes/destinations/proximité, Carte et fonds de carte, Export PDF/PNG, Interface et accessibilité, Tests et fiabilité, Décisions de cadrage. Structure JS : `HISTORY_THEME_ORDER` (array labels) + `PROJECT_HISTORY` (array `{theme, text}`), 85 entrées. Convention : ne jamais réécrire une entrée passée, sauf correction explicite d'une contradiction trompeuse ; toute modification substantielle ajoute une entrée avec thème.
- Échappement HTML systématique (`esc()`) sur données importées. Cases cochées mémorisées globalement (`memberState`), pas par groupe.

## Décisions de cadrage actées (ne pas rouvrir sauf demande explicite)
Dépendance services tiers gratuits acceptée comme risque · trafic/heure non pris en compte · rendu mobile et anciens navigateurs non prioritaires · diffusion feuilles récap via relais L3 · collecte adresses = instantané début d'année, pas de re-collecte · dispositif = simple mise en relation, sans responsabilité organisateur en cas d'accident (visibilité réelle auprès des étudiants NON confirmée) · géocodage ~1 min pour 400 personnes jugé négligeable.

## Méthode de test systématique
`node --check` sur JS extrait, comptage `<div>`/`</div>` équilibré, simulation jsdom (mocks Leaflet/Papa/XLSX/html2canvas/fetch), tests e2e injectés via `scriptCode +=` (IDs test ≥ 9000).

## Dernière session (9 sept 2026) — dans l'ordre
1. Rapport `regroupByGeography` simplifié (liste par groupe, plus d'avertissement fourchette 28-31).
2. `MIN_CAR_PER_GROUP` confirmé à 3 (implémenté, testé cas limite + cas impossible).
3. Historique réorganisé par thème (voir ci-dessus) ; 3 entrées anciennes scindées sans changer le texte (vérifié par code).
4. Point d'étape oral : forces = outil complet et testé à chaque étape ; faiblesses = fichier volumineux (~3000 lignes), dépendance services tiers gratuits.
5. Critique approfondie demandée (passation + présentation professeurs) — voir "Points ouverts" ci-dessous.
6. Paragraphe RGPD du formulaire relu par Claude (pas un avis juridique) — gaps identifiés, voir ci-dessous.
7. Document de passation créé (3 pages) : présentation générale, structure des données, fonctionnalités, dépendances et risques (avec précédents concrets), limites connues, état des tests, RGPD, 5 recommandations.

## Points ouverts / actions non lancées
- **Test réel complet jamais fait** : tout a été vérifié par simulation jsdom, jamais dans un vrai navigateur avec les vraies données de bout en bout (import → géocodage → export PDF des 20 groupes). Recommandation n°1 avant toute passation/présentation.
- **Parsing positionnel fragile — CHANTIER LANCÉ (go donné le 9 sept)** : import basculé sur une détection des colonnes par mot-clé dans l'en-tête (`detectColumnMapping`), avec repli automatique sur les positions fixes historiques (+ avertissement affiché) si l'en-tête ne permet pas d'identifier tous les champs indispensables. 6 tests dédiés ajoutés (27/27 passent). Limite assumée, non levée : les mots-clés ont été choisis sur des intitulés plausibles, pas sur l'en-tête réel du formulaire (jamais fourni à Claude) — **à vérifier sur un vrai export avant la prochaine collecte réelle**, quitte à ajuster les mots-clés d'un champ si le repli se déclenche à tort.
- **Tests automatisés obsolètes** : 21 tests ne couvrent pas les fonctionnalités récentes (voir liste plus haut). Pas encore étendus.
- **Pas de sauvegarde/persistance** : refresh de page = tout perdu. Valentin a dit ne pas s'en inquiéter pour l'instant, à revoir seulement si crashs ou lenteur constatés.
- **RGPD** : paragraphe de consentement existant analysé par Claude, gaps identifiés — sous-déclare les données réelles collectées (dit "prénom" + "ville de résidence", réalité = nom complet + adresse précise nécessaire au géocodage) ; ambiguïté sur si "coordonnées" inclut l'adresse partagée aux groupes ; absence d'identité du responsable de traitement et des droits RGPD (accès/suppression/réclamation CNIL — obligation article 13) ; durée de conservation "tant que nécessaire" sans durée/point de révision. Valentin pensait "passable" avec "quelques erreurs" ; Claude a été plus critique. Recommandation : faire relire par le service conformité RGPD de l'université avant présentation aux professeurs — hors de portée de Claude (pas juriste).
- **Visibilité clause de non-responsabilité** : actée en interne, pas confirmé qu'elle est visible des étudiant·es eux-mêmes (formulaire ou fiche remise aux L3).

## Note sur le mode "réponses orales"
Une partie de cette conversation s'est déroulée avec Valentin en voiture, demandant des réponses courtes façon oral (pas de listes/markdown), "jusqu'à nouvel ordre". Ce mode n'a pas été explicitement annulé mais les derniers échanges (critique, RGPD, document) sont redevenus écrits/structurés car il ne dictait plus. Statut à clarifier si besoin en reprise.

---
**Important pour la reprise** : ce fichier résume les décisions et l'état du projet, mais ne contient pas le code de `carte_groupes_nancy.html`. Pour continuer à le modifier, réattache le fichier lui-même (téléchargé depuis la conversation précédente) en plus de ce résumé. Une partie de ces informations est aussi déjà en mémoire longue durée côté Claude et pourrait être disponible automatiquement dans une nouvelle conversation — ce fichier sert de copie de secours fiable et autonome.
