# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Nature du dépôt

Pas de code : rendu écrit individuel du TP « Architecture & Pilotage technique » (SupDeVinci, M2, Bloc 1). Tout est rédigé en **français**, en Markdown, puis converti en PDF.

- `0_TP_A/TP_A.md` / `.pdf` : Partie A, 16 questions de cours (terminée).
- `1_Velocite/TP_B_1.md` / `.pdf` : Partie B, Situation 1 Vélocité (terminée, sert de **modèle** pour les suivantes).
- Situations 2 à 5 (GestiMed dans `2_GestiMed/TP_B_2.md`, puis Beaulieu, Théo, Nomad) : un dossier et un fichier par situation, même structure que `TP_B_1.md`.

## Sources de référence

URL de base (`<base>`) : @linkM2.txt

- Énoncé complet (dossiers des 5 cas, extraits de code, comptes rendus, chiffres) : `<base>tp.html`
- Cours : `<base>chapitre-1/cours.html`

Les pages sont longues (~46 Ko) : WebFetch les résume et perd des chiffres. Pour un cas, récupérer le texte brut (`curl -s <url>`) et extraire la section de la situation avant de rédiger.

## Règles de notation qui guident la rédaction

- Chaque réponse doit utiliser les **chiffres, noms, contraintes et extraits du dossier** ; une réponse générique vaut zéro. Test : la réponse ne doit pas pouvoir être réutilisée pour une autre entreprise.
- Ne jamais inventer de faits : écrire « information manquante → hypothèse X » et marquer les incertitudes par l'emoji **🤔** (légendé dans l'introduction des hypothèses de chaque situation).
- Chaque situation suit l'ordre de l'énoncé : **Ce que j'ai compris** (≈5 lignes, chiffres clés) → **Ce qui manque : hypothèses** (tableau Extrait du dossier / Hypothèse / Ce qui change si fausse) → **Livrable 1…n** dans l'ordre → **Ce que je refuse de faire, et pourquoi**.
- **Usage de l'IA** : chaque fichier commence, juste après le titre, par une section « Usage de l'IA ». Elle explique en quelques phrases complètes à quoi l'IA a servi (par exemple extraction des chiffres du dossier, proposition de structure ou relecture), ce que l'auteur a vérifié ou corrigé lui-même, et ce qui relève de son propre jugement.
- Règles métier : colonne **Source** = Dossier (cité) / Déduite (lue dans le discours) / **🤔** (à valider). Ne jamais présenter une extrapolation comme un fait du dossier.
- Schémas obligatoirement légendés et lisibles sans l'auteur ; les schémas de modules indiquent qui dépend de qui et si le lien est un **appel** ou un **événement**.

## Gabarits du cours à réutiliser

| Élément | Format |
|---|---|
| Scénario de qualité | Qui / Quoi / Quand / Mesure observable (chiffrée) |
| User story | « En tant que…, je veux…, afin de… » |
| Utility tree | Utilité → attribut → scénario, notes (importance métier, difficulté) en H/M/L ; commencer par les H/H puis H/L |
| Compromis | « on gagne X, on perd Y » + justification |
| Calcul de charge | utilisateurs → requêtes → req/s moyen et au pic, puis conclusion |
| V1/V2/V3 | décisions prises en V1 à cause de la V3 (« portes ouvertes ») |
| Pattern | problème · solution · quand ne pas l'utiliser (8 patterns du cours : Factory, Adapter, Facade, Decorator, Strategy, Observer, State, Repository) |
| Sous-domaines | cœur / support / générique (acheter le générique) |

Ne pas introduire de méthodes absentes du cours (ATAM, ADR, C4) sauf demande explicite.

## Style

- Tableaux Markdown pour règles, contraintes, user stories, compromis, hypothèses ; schémas en blocs ASCII ou Mermaid.
- Ton pédagogue et professionnel : rester concis, mais écrire des phrases complètes reliées par des connecteurs logiques (car, donc, en effet, par conséquent, mais, or…) qui explicitent les causes et les conséquences.
- Le document doit être lisible par un humain qui ne connaît pas le contexte : pas de style télégraphique, pas d'abréviations non définies, pas de sous-entendus ; chaque décision est justifiée explicitement. Dans les tableaux, les cellules restent des phrases compréhensibles seules.
- Les « réponses au client » tiennent en ~10 lignes (citation `>`).
- Diagrammes Lucid possibles via le MCP `lucid` (déjà utilisé pour l'utility tree de Vélocité).

## Commandes

Conversion PDF via le skill `/md-to-pdf` (package `md-to-pdf` installé globalement via pnpm) :

```powershell
$pkg = (pnpm ls -g md-to-pdf --parseable | Select-Object -Last 1) -replace '^\\\\\?\\', ''
$cli = Join-Path $pkg "dist\cli.js"
$root = "<chemin absolu de la racine du projet>"
node $cli "$root\<dossier>\<fichier>.md" --basedir "$root" --stylesheet "$pkg\markdown.css" --stylesheet "$root\pdf.css"
```

**Mise en page et sauts de page** : la feuille `pdf.css` (racine du projet) empêche qu'un élément soit coupé par un changement de page. Elle garde chaque titre avec le contenu qui le suit, et elle ne coupe ni les images, ni les blocs de code, ni les citations, ni les lignes de tableau ; un tableau qui tient sur une page n'est pas coupé du tout. Il faut passer les deux `--stylesheet`, car l'option remplace la feuille par défaut de md-to-pdf au lieu de s'y ajouter. Lors de la rédaction :
- garder les tableaux courts (une dizaine de lignes au plus) ; un tableau plus long doit être découpé en plusieurs tableaux introduits chacun par une phrase ;
- dimensionner les schémas pour qu'ils tiennent sur une page avec leur légende ;
- si un saut reste mal placé, insérer `<div class="saut-de-page"></div>` juste avant le titre concerné.

`--basedir` est indispensable et doit pointer vers la **racine du projet**, car les images sont rangées dans `images/` à la racine. Comme md-to-pdf résout les chemins relatifs depuis le dossier du fichier `.md`, un fichier placé dans un sous-dossier doit référencer ses images en `../images/...`. Si l'une de ces deux conditions manque, les images sont remplacées dans le PDF par une icône d'image cassée, sans aucun message d'erreur. Pour le vérifier, on cherche `/Width` dans le PDF : une image de 14 × 16 pixels correspond à cette icône, et non à la vraie image. Enfin, à cause des crochets du nom de dossier (`[M2]`), il faut utiliser `-LiteralPath` avec `Get-Item` en PowerShell.
