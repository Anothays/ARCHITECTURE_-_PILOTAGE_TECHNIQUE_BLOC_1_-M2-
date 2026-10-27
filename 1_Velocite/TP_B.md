# Partie B — Mises en situation

## Situation 1 — Vélocité (projet neuf)

### 1.0 Compréhension du contexte

Le client est un réseau de 40 ateliers de réparation de vélos, tous franchisés. Aujourd'hui, le siège manque de visibilité sur ses ateliers, ce qui freine la prise de décision. Chaque atelier applique ses propres règles de gestion, de logistique, de facturation et d'organisation, parfois sans outil informatique. L'entreprise souhaite harmoniser ces processus autour d'une application unique :

* le client y dépose sa demande, reçoit son devis et suit l'avancement de sa réparation ;
* les gérants et mécaniciens y gèrent les dépôts, les devis et le planning de l'atelier ;
* le siège y consulte les indicateurs du réseau.

Cycle de vie d'une réparation (vue métier) :

```
 \[Déposé] ──► \[En diagnostic] ──► \[Devis envoyé] ──┬──► \[Accepté] ──► \[En réparation] ──► \[Prêt] ──► \[Restitué / payé]
                                                   │                      │
                                                   │                      └──► (travaux supplémentaires) ──► \[Avenant envoyé] ──► \[Accepté]
                                                   └──► \[Refusé] ──► \[Prêt à restituer] ──► \[Restitué] (diagnostic facturé ou non)
```

\---

### 1.1 Règles métier

Légende : **🤔** = incertitude à valider avec Nadia et les gérants.

|#|Règle|
|-|-|
|R1|Toute prise en charge commence par un dépôt enregistré : client, vélo, symptôme décrit, date, photos de l'état.|
|R2|Aucune intervention n'est réalisée sans devis accepté par le client. **🤔** Existe-t-il un seuil (ex. < 30 €) en dessous duquel le mécanicien intervient sans devis ?|
|R3|Le devis est calculé à partir de la grille de prix **de l'atelier** (forfaits de main-d'œuvre + pièces).|
|R4|Le mécanicien peut ajuster le prix du devis (remise ou majoration) ; l'ajustement est tracé avec un motif. **🤔** Faut-il un plafond ?|
|R5|Un devis a une durée de validité. **🤔** Durée (15 jours ?).|
|R6|L'accord du client est tracé (clic sur lien, signature sur tablette, ou accord oral saisi avec date et auteur).|
|R7|Si des travaux supplémentaires sont découverts, un avenant est émis et doit être accepté avant de poursuivre.|
|R8|En cas de refus du devis, le vélo est restitué en l'état ; le diagnostic est facturé selon la politique de l'atelier.|
|R9|Chaque atelier choisit sa politique de diagnostic : gratuit, payant, ou payant mais déduit si le devis est accepté. **🤔** Le siège veut-il imposer une politique réseau ?|
|R10|Le siège peut proposer une grille de référence ; chaque atelier reste libre de ses prix. **🤔** Contrat de franchise à vérifier.|
|R11|Les prestations sont codifiées dans un référentiel commun au réseau (ex. « remplacement chaîne »), même si le prix diffère par atelier.|
|R12|VAE : un tarif de main-d'œuvre spécifique s'applique. **🤔** Montant par atelier ou réseau ?|
|R13|VAE : les interventions sur batterie/moteur sont réservées aux mécaniciens habilités. **🤔** Habilitation exigée par les constructeurs (Bosch, Shimano…) ?|
|R14|VAE : une panne couverte par la garantie constructeur est orientée vers le fabricant, pas facturée au client. **🤔** Procédure exacte.|
|R15|Le paiement a lieu à la restitution. **🤔** Acompte demandé pour les grosses réparations ou commandes de pièces ?|
|R16|Une facture est émise à chaque encaissement ; elle est numérotée, inaltérable et conservée 10 ans.|
|R17|Un vélo non récupéré déclenche des relances. **🤔** Après combien de jours ? Frais de gardiennage ?|
|R18|Une réparation est garantie pendant une durée donnée. **🤔** Durée (3 mois ?).|
|R19|Le gérant voit toutes les données de son atelier ; le siège ne voit que les indicateurs que le gérant a accepté de partager. **🤔** Liste précise des indicateurs « non négociables » pour le siège (contrat de franchise).|

\---

### 1.2 Contraintes non négociables

|Contrainte|Nature|Ce qu'elle élimine|
|-|-|-|
|**Budget 90 000 €** (tout compris)|Financière|Les applications natives iOS + Android + web séparées ; le développement « from scratch » de briques standard (caisse, paiement, envoi SMS) ; une équipe interne.|
|**Livraison avant le printemps** (\~mi-mars 2027, soit \~20 semaines)|Calendrier|Le déploiement « big bang » des 40 ateliers ; tout périmètre au-delà du cycle dépôt → devis → restitution ; les technologies que le prestataire ne maîtrise pas déjà.|
|**Pas d'équipe informatique interne**|Organisation|L'hébergement sur serveur maison ; les architectures complexes à exploiter (microservices, Kubernetes) ; tout composant qui demande une administration quotidienne. Impose un hébergement managé (PaaS/SaaS) et un contrat de maintenance.|
|**Franchisés indépendants, réticents au contrôle**|Organisation / contractuelle|La grille de prix nationale imposée ; la visibilité totale du siège sur le CA de chaque atelier. Impose un paramétrage par atelier et des droits d'accès fins.|
|**RGPD** (\~150 000 clients)|Légale|Le stockage hors UE sans garanties ; la conservation illimitée des données ; l'envoi de SMS marketing sans consentement. Impose registre, durées de conservation, contrats sous-traitants.|
|**Loi anti-fraude TVA** (art. 286 I-3° bis CGI)|Légale|Le développement maison de l'encaissement : tout logiciel qui enregistre des paiements de particuliers doit être certifié (NF525 ou attestation éditeur). Impose d'intégrer une caisse certifiée existante.|
|**DSP2 / PCI-DSS** (paiement par carte)|Légale|Le stockage de numéros de carte par l'application. Impose un prestataire de paiement (PSP).|
|**Facturation électronique** (réforme 2026-2027)|Légale|Concerne les clients professionnels (flottes d'entreprise) : la caisse/outil de facturation retenu doit être compatible avec une plateforme agréée.|

\---

### 1.3 Exigences fonctionnelles (user stories)

**Livrées au printemps (V1)**

|#|User story|
|-|-|
|US1|En tant que **mécanicien**, je veux enregistrer un dépôt (client, vélo, symptôme, photos) sur tablette en moins de 2 minutes, afin de ne pas faire attendre la file au comptoir.|
|US2|En tant que **mécanicien**, je veux créer un devis à partir de la grille de mon atelier, afin de chiffrer vite et de façon cohérente.|
|US3|En tant que **client**, je veux recevoir mon devis par SMS ou email et l'accepter ou le refuser en un clic, afin de ne pas avoir à rappeler l'atelier.|
|US4|En tant que **mécanicien**, je veux envoyer un avenant quand je découvre une panne supplémentaire, afin d'obtenir l'accord du client avant de continuer.|
|US5|En tant que **client**, je veux suivre l'état de ma réparation via un lien, sans créer de compte.|
|US6|En tant que **client**, je veux être prévenu quand mon vélo est prêt.|
|US7|En tant que **gérant**, je veux paramétrer ma grille de prix et ma politique de diagnostic, afin de garder mon autonomie commerciale.|
|US8|En tant que **gérant**, je veux voir le planning des réparations en cours de mon atelier, afin de répartir le travail entre mécaniciens.|
|US9|En tant que **gérant**, je veux encaisser et éditer la facture à la restitution via une caisse certifiée, afin d'être en règle.|
|US10|En tant que **gérant**, je veux choisir quels indicateurs je partage avec le siège, afin de garder la maîtrise de mes données.|
|US11|En tant que **dirigeant du siège**, je veux voir par atelier le nombre de réparations, le délai moyen et le taux d'acceptation des devis, afin de piloter le réseau.|

**Après le printemps (V2 / V3)**

|#|User story|
|-|-|
|US12|En tant que **client**, je veux prendre rendez-vous en ligne pour déposer mon vélo.|
|US13|En tant que **client**, je veux consulter l'historique d'entretien de mon vélo.|
|US14|En tant que **gérant**, je veux suivre mon stock de pièces et être alerté en cas de rupture.|
|US15|En tant que **dirigeant du siège**, je veux voir le CA consolidé du réseau (pour les ateliers qui l'acceptent).|
|US16|En tant que **client**, je veux souscrire un abonnement « entretien annuel » payé mensuellement.|
|US17|En tant que **client**, je veux acheter un vélo d'occasion révisé par le réseau.|
|US18|En tant que **gérant**, je veux commander mes pièces via la centrale d'achat du siège.|

\---

### 1.4 Scénarios de qualité et utility tree

**Scénarios chiffrés**

|#|Attribut|Qui|Quoi|Quand|Mesure observable|
|-|-|-|-|-|-|
|QS1|Utilisabilité|Un mécanicien formé en moins d'1 h|enregistre un dépôt complet (client, vélo, symptôme, 2 photos)|au comptoir, un samedi de saison, avec une file d'attente|≤ 2 min en médiane, mesuré au chronomètre sur les ateliers pilotes|
|QS2|Disponibilité|Un mécanicien ou gérant|ouvre l'application pour enregistrer un dépôt ou restituer un vélo|pendant les heures d'ouverture (mar.–sam., 9 h–19 h), en saison|Service disponible ≥ 99,5 % du temps d'ouverture (≈ 1 h 30 d'arrêt max par mois) ; en cas de coupure, procédure papier puis ressaisie < 10 min|
|QS3|Confidentialité|Un utilisateur du siège|tente d'accéder au CA détaillé d'un atelier qui n'a pas accepté le partage|à tout moment|100 % des tentatives refusées et journalisées ; vérifié par tests automatisés d'autorisation à chaque livraison|

**Utility tree** — (importance métier, difficulté technique) : H = haute, M = moyenne, L = basse

```
Utilité
├── Utilisabilité
│   ├── (H,M) QS1 – Dépôt ≤ 2 min au comptoir
│   └── (M,L) Accord du devis par le client en 1 clic, sans compte
├── Disponibilité
│   ├── (H,M) QS2 – 99,5 % pendant les heures d'ouverture
│   └── (M,H) Fonctionnement hors ligne complet en atelier
├── Sécurité / confidentialité
│   ├── (H,M) QS3 – Cloisonnement des données atelier / siège
│   └── (H,L) Conformité RGPD (consentements, durées de conservation)
├── Performance
│   └── (L,L) Page de suivi client affichée < 2 s en 4G
└── Modifiabilité
    └── (M,M) Ajouter un produit (abonnement, occasion) sans refonte du modèle
```

**Par où commencer : QS3 (cloisonnement des données), puis QS1.**

* QS3 est **structurant** : le cloisonnement par atelier se décide dans le modèle de données dès le premier jour ; le rajouter après coûte une refonte. C'est aussi la **condition d'adhésion des gérants** réticents : sans garantie, ils n'utiliseront pas l'outil.
* QS1 vient juste après : l'adoption au comptoir est le risque n°1 du projet ; on la valide sur maquette avec 2–3 mécaniciens avant de coder.
* QS2 est en grande partie « acheté » avec un hébergement managé ; le hors-ligne complet (M,H) est écarté en V1 au profit d'une procédure papier (voir compromis C1).

\---

### 1.5 Compromis

|#|Option retenue|On gagne|On perd|Justification|
|-|-|-|-|-|
|C1|**Application web responsive (PWA)** plutôt qu'applis natives|Une seule base de code (≈ 40 % de coût en moins), pas de publication sur les stores, mises à jour instantanées|Hors-ligne limité, notifications push restreintes sur iOS, expérience photo un peu moins fluide|Le budget et le délai ne permettent pas trois applications ; le hors-ligne est compensé par une procédure papier ; les notifications client passent par SMS/email.|
|C2|**Intégrer une caisse certifiée du marché** plutôt que coder l'encaissement|Conformité NF525 immédiate, délai réduit, pas de responsabilité légale sur la caisse|Dépendance à un éditeur, abonnement mensuel par atelier, commission par transaction, UX moins intégrée|Coder une caisse certifiée est hors budget et hors délai ; c'est une brique générique, pas un avantage concurrentiel.|
|C3|**Prix libres par atelier sur un référentiel de prestations commun** plutôt qu'une grille nationale imposée|Adhésion des gérants, déploiement rapide, respect du modèle de franchise|Harmonisation des prix non atteinte, comparaisons de prix entre ateliers plus difficiles|Imposer une grille bloquerait l'adoption ; le référentiel commun permet tout de même au siège de comparer les volumes et délais par type de prestation.|

\---

### 1.6 Calcul de charge

**Hypothèses**

* Saison : 40 ateliers × 60 réparations/semaine = **2 400 réparations/semaine**.
* Une réparation génère \~16 actions utilisateur (dépôt, diagnostic, devis, consultation et acceptation client, 3 changements de statut, \~4 consultations du suivi par le client, restitution/facture) ; une action ≈ 5 requêtes HTTP → **\~80 requêtes/réparation**.
* Usage hors réparation (planning, tableaux de bord) : \~50 actions/jour/atelier × 5 requêtes × 6 jours × 40 ateliers = **60 000 requêtes/semaine**.
* Ouverture : 6 jours × 10 h = 216 000 s/semaine.

**Calcul**

```
Requêtes/semaine = 2 400 × 80 + 60 000 = 252 000
Moyenne          = 252 000 / 216 000       ≈ 1,2 req/s
Pic (samedi matin ×5, semaine de rush ×2)  ≈ 12 req/s
```

**Volume de données** : \~90 000 réparations/an. Le poste lourd est la photo : 2 photos × 300 Ko (compressées) ≈ **55 Go/an**, à placer dans un stockage objet.

**Conclusion : la charge n'est pas critique.** Une seule petite instance applicative gère plusieurs centaines de req/s. Les vrais risques sont ailleurs : **disponibilité** pendant les heures d'ouverture, **intégrité** des factures, **adoption** par les ateliers. Il est inutile d'investir dans la scalabilité (cache distribué, microservices).

\---

### 1.7 V1 / V2 / V3

|Version|Échéance|Périmètre|
|-|-|-|
|**V1**|Pilote 3 ateliers mi-février 2027, généralisation mars 2027|Dépôt, devis, avenant, accord client par lien, suivi de statut, notifications SMS/email, grille et politique de diagnostic par atelier, planning atelier, encaissement via caisse certifiée intégrée, tableau de bord siège (indicateurs d'activité consentis), gestion des rôles.|
|**V2**|Été – automne 2027|Prise de RDV en ligne, historique du vélo côté client, stock de pièces simple, CA consolidé (avec accord des gérants), paiement en ligne avant restitution.|
|**V3**|2028|Abonnement « entretien annuel », vente de vélos d'occasion révisés, centrale d'achat de pièces.|

**Décisions prises en V1 à cause de la V3**

|#|Décision V1|Coût en V1|Ce qu'elle rend possible en V3|
|-|-|-|-|
|D1|Le **vélo est une entité à part entière** (identifiant, n° de cadre, marque, type VAE ou non) et non un champ texte du dépôt.|Quasi nul|**Abonnement** : l'entretien est rattaché au vélo. **Occasion** : un vélo révisé a un historique traçable, argument de vente.|
|D2|**Référentiel commun des prestations et des pièces** (codes réseau), même si prix et fournisseurs restent locaux.|Quelques jours de cadrage|**Centralisation des pièces** : le siège connaît déjà la consommation réelle par référence pour négocier.|
|D3|**Client unique au niveau du réseau** (pas un fichier par atelier), avec consentements RGPD, et choix d'un prestataire de paiement qui gère le **paiement récurrent**.|Faible|**Abonnement** valable dans tout le réseau, sans migration de clients ni changement de prestataire.|

\---

### 1.8 Réponse à Nadia

> Bonjour Nadia,
>
> Le budget de 90 000 € et l'échéance du printemps sont \*\*tenables, à condition de réduire la V1\*\* au cœur du besoin : dépôt, devis, accord client par lien, suivi de la réparation, encaissement et un tableau de bord réseau.
> Estimation V1 : \*\*70 à 80 k€\*\* de réalisation, plus \*\*10 à 15 k€\*\* de réserve (pilote, formation, imprévus). Prévoir ensuite \*\*≈ 1 à 2 k€/mois\*\* de fonctionnement (hébergement, SMS, maintenance), et un abonnement caisse par atelier (\~30 à 70 €/mois).
> Calendrier : cadrage en novembre, pilote sur 3 ateliers mi-février, déploiement sur les 40 en mars.
> Ce qui \*\*attend\*\* : prise de RDV en ligne, stock et CA consolidé (V2, \~40–60 k€), puis abonnement, occasion et centrale d'achat (V3, \~80–120 k€).
> Hypothèses : \~90 000 réparations/an, une caisse certifiée du marché, des prix libres par atelier, et 3 ateliers pilotes volontaires.
> Pour démarrer, j'ai besoin de votre arbitrage sur ce que le siège doit voir obligatoirement dans chaque atelier.

\---

### 1.9 Hypothèses et informations manquantes

|Information manquante|Hypothèse retenue|Impact si elle est fausse|
|-|-|-|
|Volume réel de réparations (« environ 60/semaine »)|60/semaine en saison (26 semaines), 25 hors saison → **\~88 000 réparations/an**|Faible : même ×5, la charge reste non critique.|
|Nombre de clients (« peut-être 150 000/an »)|**Incohérent** avec \~88 000 réparations/an : soit le chiffre inclut les ventes au comptoir, soit il est surestimé. On dimensionne sur les réparations.|Faible sur la charge, fort sur le coût des SMS et les obligations RGPD.|
|Circuit actuel dépôt → restitution, acompte éventuel|Cycle décrit en 1.0, paiement à la restitution|Ajout d'un module d'acompte (V1 ou V2).|
|Prix « à la tête du client »|Remise/majoration manuelle tracée (R4)|Si le siège veut l'interdire : règle de plafond à ajouter.|
|Raison des grilles différentes et des diagnostics facturés ou non|Liberté tarifaire prévue au contrat de franchise|Si le siège veut harmoniser : grille nationale, risque de rejet par les gérants.|
|Commandes fournisseurs (canal, API disponibles, portail commun ?)|Hors V1 ; seul le référentiel de pièces est posé (D2)|Si la centralisation devient urgente, V3 avancée en V2.|
|Données que le siège doit voir obligatoirement|Indicateurs d'activité oui, CA détaillé seulement avec accord (R19)|Si le contrat impose le CA : moins de paramétrage, plus de résistance.|
|Équipement des ateliers (tablette, PC, connexion internet)|Chaque atelier a une connexion et au moins une tablette ou un PC|Si non : achat de matériel (\~400 €/atelier, soit 16 k€) à ajouter au budget.|
|Maintenance après livraison|Contrat de maintenance avec le prestataire (TMA)|Sans maintenance, l'application se dégrade dès la première évolution réglementaire.|
|Abonnement entretien et vélos d'occasion (absents du brief initial)|Prévus en V3, préparés par D1 et D3|Si avancés en V1 : le budget et le délai ne tiennent plus.|

\---

### Usage de l'IA

IA (Claude) utilisée pour structurer les livrables et vérifier la cohérence des calculs ; contenu relu, hypothèses et arbitrages validés par l'auteur.

