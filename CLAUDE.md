# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Contexte métier

PSG Energy (Provider Services Groupe) est une régie commerciale française (SASU) spécialisée en énergies renouvelables. Elle **vend** des équipements au nom d'installateurs RGE certifiés sans réaliser d'installations. Marges : Kwanthic 28,5% / Le Solariste 37,5%. Deux axes : France IDF (77, 94, 91, 93) + Congo Brazzaville / zone CEMAC.

## Architecture

Projet 100% front-end statique — fichiers HTML autonomes, zéro framework, zéro build step. Chaque fichier HTML contient son propre CSS inline et JS inline. Les fonts sont auto-hébergées dans `fonts/`.

| Fichier | Rôle | Accès |
|---------|------|-------|
| `psg-energy-site.html` | Site vitrine institutionnel desktop (psg-energy.fr) | Public |
| `psg-energy-v5-mvp.html` | App mobile B2C/B2B (simulateur, espace B2B SIRET, i18n 6 langues) | Public |
| `psg-chatbot-elia.html` | Chatbot IA Élia (nécessite un proxy API Claude côté serveur) | Public |
| `psg-commercial-signature.html` | Tunnel devis + signature électronique YouSign | Public |
| `psg-africa-intelligence.html` | Dashboard Intelligence Afrique (noindex) | Semi-privé |
| `psg-dashboard-admin.html` | Dashboard interne CRM/pipeline — **NE PAS COMMITTER** | Privé |
| `mentions-legales.html` | Mentions légales LCEN | Public |
| `cgv.html` | Conditions Générales de Vente (loi Hamon, rétractation 14j) | Public |
| `politique-confidentialite.html` | Politique de confidentialité RGPD (10 sections) | Public |

## Règles absolues

- `psg-dashboard-admin.html` est dans `.gitignore` — ne jamais l'ajouter au repo (contient marges, leads, données commerciales)
- Le dossier `memory/` est dans `.gitignore` — ne jamais le committer (profil Charly, décisions internes)
- Ne jamais exposer de clé API Anthropic côté client — utiliser un proxy Vercel serverless function
- Toujours répondre et commenter en **français**
- Deux domaines distincts : `psg-energy.fr` (France) ≠ `psglobal.energy` (International) — ne pas confondre

## Fonts (RGPD)

Les fonts Outfit et Inter sont auto-hébergées dans `fonts/` (woff2, licence SIL OFL). Référencées via `<link rel="stylesheet" href="fonts/fonts.css">`. Ne jamais réintroduire `fonts.googleapis.com` — interdit RGPD en France depuis la décision CNIL 2022.

## Formulaire de contact

`psg-energy-site.html` → fonction `submitForm()` : poste un payload JSON `{nom, tel, cp, projet, date, source}` vers `LEAD_WEBHOOK` (constante en haut du script). Si vide, fallback `mailto:contact@psglobal.energy`. Pour brancher Make.com / Zoho CRM : renseigner l'URL du webhook dans `LEAD_WEBHOOK`.

## Chatbot Élia

`psg-chatbot-elia.html` tente `https://api.anthropic.com/v1/messages` sans header `x-api-key` (aucune clé exposée, mais le chatbot ne fonctionne pas sans proxy). La solution est une Vercel Edge Function qui proxifie l'appel avec la clé côté serveur.

## Simulateur d'aides ANAH 2026

Dans `psg-energy-site.html` et `psg-energy-v5-mvp.html` : objet `AC` → barèmes MPR par tranche fiscale, CEE, TVA 5,5%, Éco-PTZ, prime EDF OA, crédit ADVENIR. Montants codés en dur — à mettre à jour à chaque révision ANAH.

## Déploiement

- Cible France : OVH → `psg-energy.fr` (uploader `psg-energy-site.html` comme `index.html` dans `www/`, inclure le dossier `fonts/`)
- Cible app mobile : Vercel → `psglobal.energy` (connecter le repo GitHub, le dossier `fonts/` est servi automatiquement)
- `psg-africa-intelligence.html` et `psg-dashboard-admin.html` ne doivent jamais être servis publiquement

## Variables CSS communes

Tous les fichiers partagent les mêmes tokens (copiés manuellement) :
- `--blue:#1A4DFF` / `--green:#00C48C` / `--gold:#F5A000`
- `--ink:#08091A` / `--surf:#F2F5FB` / `--ff:'Outfit'` / `--fb:'Inter'`
- Toute modification de charte doit être propagée dans tous les fichiers HTML

## Conformité CNIL / RGPD — règles à respecter impérativement

### Ce qui est déjà en place (ne pas régresser)
- **Fonts auto-hébergées** : `fonts.googleapis.com` interdit — décision CNIL 10/02/2022 (tribunal Munich, Allemagne également). Toujours `<link rel="stylesheet" href="fonts/fonts.css">`.
- **Aucun cookie tiers** : pas de Google Analytics, Meta Pixel, Hotjar, etc. → aucun bandeau de consentement requis (pas de cookies non-essentiels).
- **Pas de clé API côté client** : les appels Anthropic passent par un proxy serveur uniquement.
- **`politique-confidentialite.html`** : créée, couvre les 10 obligations RGPD (art. 13/14).
- **`mentions-legales.html`** : conforme LCEN art. 6 (hébergeur, éditeur, DPO).
- **`cgv.html`** : loi Hamon — droit de rétractation 14 jours, médiateur MEDICYS.

### Règles à respecter lors de toute modification
1. **Ne jamais introduire de ressource externe** (CDN, image tracker, font, analytics) sans vérifier qu'elle ne transmet pas d'IP à un serveur hors UE sans consentement préalable.
2. **Tout nouveau formulaire** doit afficher une mention d'information RGPD (art. 13) au point de collecte : finalité, responsable, durée de conservation, droits, lien vers `politique-confidentialite.html`.
3. **Si des cookies sont introduits** (ex. : analytics, A/B test), un bandeau de consentement conforme CNIL 2022 est obligatoire avant tout dépôt — les cookies analytiques ne sont pas exemptés sans anonymisation stricte.
4. **Sous-traitants** : tout nouveau prestataire traitant des données personnelles doit être ajouté à la section 4 de `politique-confidentialite.html` avec base légale et localisation des serveurs.
5. **Durées de conservation** : ne pas collecter de données sans durée définie dans `politique-confidentialite.html`.
6. **Email DPO** `dpo@psg-energy.fr` doit être opérationnel avant la mise en production (délai de réponse légal : 1 mois, RGPD art. 12).

### Points légaux encore en attente
- SIREN non encore disponible (immatriculation Jurisociété N°1727182) — mentionné dans `mentions-legales.html` et footer
- Emails `contact@psg-energy.fr` et `dpo@psg-energy.fr` à créer sur OVH **avant mise en ligne**
- Connexion Zoho CRM dans Make.com (OAuth, UI Make.com) — leads non encore transmis au CRM
- Tester le formulaire en production et vérifier réception dans Zoho CRM
