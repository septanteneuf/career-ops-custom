# Mode : pipeline -- Inbox d'URLs (Second Brain)

Traite les URLs d'offres accumulees dans `data/pipeline.md`. Le candidat ajoute des URLs quand il veut et lance ensuite `/career-ops pipeline` pour toutes les traiter d'un coup.

## Workflow

1. **Lire** `data/pipeline.md` -> trouver les items `- [ ]` dans la section "En attente" / "Pending" / "Pendientes"
2. **Pour chaque URL en attente** :
   a. Calculer le prochain `REPORT_NUM` sequentiel (lire `reports/`, prendre le numero le plus eleve + 1)
   b. **Extraire l'offre** avec Playwright (`browser_navigate` + `browser_snapshot`) -> WebFetch -> WebSearch
   c. Si l'URL n'est pas accessible -> marquer comme `- [!]` avec une note et continuer
   d. **Executer l'auto-pipeline complet** :
   Evaluation A-F -> Report .md -> PDF (si score >= 3.5) -> Tracker
   Avant l'évaluation A-F, détecter automatiquement le domaine principal de l'offre :
   - IT / Support / Helpdesk
   - Web / WordPress / CMS
   - Hébergement / DNS / Infrastructure légère
   - Photographie / Communication
   - Restauration / Cuisine
   - Gestion opérationnelle
   - Médico-social / Collectivité
   - Hybride
   Adapter ensuite automatiquement :
   - les proof points priorisés
   - le résumé CV
   - les expériences mises en avant
   - les expériences minimisées
   - le ton de la lettre de motivation
	
	Ne pas mélanger inutilement les expériences restauration et IT/web.
   - Si l'offre est IT/web :
     réduire fortement les expériences restauration sauf si elles apportent :
     - gestion d'équipe
     - relation client
     - gestion opérationnelle
     - autonomie
     - résistance au stress
   - Si l'offre est restauration :
     réduire fortement les expériences IT/web sauf si elles apportent:
     - organisation
     - communication
     - gestion
     - digitalisation
     - réseaux sociaux
   - Si l'offre est photographie ou communication :
     mettre en avant portfolio, WordPress, publication web et communication visuelle.
   e. **Deplacer de "En attente" vers "Traitees"** : `- [x] #NNN | URL | Entreprise | Role | Score/5 | PDF oui/non`
3. **Si 3+ URLs en attente**, lancer des agents en parallele (Agent tool avec `run_in_background`) pour maximiser la vitesse.
4. **A la fin**, afficher un tableau recapitulatif :

```
| # | Entreprise | Role | Score | PDF | Action recommandee |
```
Action recommandée possible :
- "Postuler rapidement"
- "Bon match mais CV à retravailler"
- "Match partiel"
- "Faible pertinence"
- "Ne pas postuler"

## Format de pipeline.md

```markdown
## En attente
- [ ] https://jobs.example.com/posting/123
- [ ] https://boards.greenhouse.io/company/jobs/456 | Company SAS | Senior PM
- [!] https://private.url/job -- Erreur : login requis

## Traitees
- [x] #143 | https://jobs.example.com/posting/789 | Acme SAS | AI PM | 4.2/5 | PDF oui
- [x] #144 | https://boards.greenhouse.io/xyz/jobs/012 | BigCo | SA | 2.1/5 | PDF non
```

> Note : Les en-tetes de section peuvent etre en EN ("Pending"/"Processed"), ES ("Pendientes"/"Procesadas"), DE ("Offen"/"Verarbeitet") ou FR ("En attente"/"Traitees"). Etre flexible a la lecture, fidele au style existant a l'ecriture.

## Detection intelligente de l'offre depuis l'URL

1. **Playwright (prefere) :** `browser_navigate` + `browser_snapshot`. Fonctionne avec toutes les SPAs.
2. **WebFetch (fallback) :** Pour les pages statiques ou quand Playwright n'est pas disponible.
3. **WebSearch (dernier recours) :** Chercher sur des portails secondaires qui indexent l'offre.

**Cas particuliers :**
- **LinkedIn** : Peut necessiter un login -> marquer `[!]` et demander au candidat de coller le texte
- **PDF** : Si l'URL pointe vers un PDF, le lire directement avec le Read tool
- **Prefixe `local:`** : Lire le fichier local. Exemple : `local:jds/linkedin-pm-ai.md` -> lire `jds/linkedin-pm-ai.md`
- **Welcome to the Jungle / Indeed FR / APEC** : Portails francophones courants. Playwright gere bien les cookie banners
- **France Travail (ex-Pole emploi)** : Offres structurees, bien lisibles par machine. WebFetch suffit generalement

## Adaptation au marché suisse

Par défaut, privilégier :
- le marché suisse
- les salaires suisses
- les formulations suisses/francophones
- les attentes RH suisses
- les formats de CV suisses

Éviter :
- le ton startup américain
- les formulations exagérées
- les buzzwords inutiles
- les lettres trop longues

## Fiabilité et exactitude

Ne jamais :
- inventer un diplôme
- inventer une responsabilité hiérarchique
- inventer une certification
- transformer une activité associative ou amateur en activité professionnelle
- prétendre à une expertise senior en développement logiciel si elle n'est pas documentée

## Portfolio et références réelles

Quand pertinent, utiliser les éléments suivants comme preuves :
- septanteneuf.com
- photo.septanteneuf.com
- tircourgevaux.com
- administration WordPress
- DNS / domaines / hébergement
- photographie aviation et événementielle
- communication associative

## Numerotation automatique

1. Lister tous les fichiers dans `reports/`
2. Extraire le numero du prefixe (ex : `142-medispend...` -> 142)
3. Nouveau numero = maximum trouve + 1

## Synchronisation des sources

Avant de traiter une URL, verifier la sync :

```bash
node cv-sync-check.mjs
```

En cas de desynchronisation, alerter le candidat avant de continuer.
