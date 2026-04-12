---
name: security-audit
description: Effectue un audit de sécurité complet sur une route, un fichier ou une feature. Utiliser avant de merger du code sensible ou pour vérifier une surface d'attaque.
---

Effectue un audit de sécurité sur : $ARGUMENTS

Analyse méthodiquement chaque vecteur d'attaque ci-dessous.

## 1. Injection SQL

- Toutes les requêtes utilisent-elles des paramètres (`$1, $2, ...`) ?
- Aucune concaténation de chaînes dans les requêtes SQL ?
- Les valeurs triées/filtrées dynamiquement (ORDER BY, noms de colonnes) sont-elles validées par whitelist ?

## 2. Authentification & Sessions

- Le middleware `authenticateToken` est-il appliqué sur toutes les routes protégées ?
- Le JWT est-il vérifié (signature + expiration) à chaque requête ?
- Les tokens sont-ils stockés de façon sécurisée (httpOnly cookie côté client) ?
- Les tokens expirés/révoqués sont-ils correctement rejetés ?

## 3. Autorisation & IDOR

- Les routes d'écriture (POST/PUT/DELETE) vérifient-elles le rôle requis (`requireRole`) ?
- Les GET par ID filtrent-ils par `id_context` (ou équivalent) directement en SQL ?
- Un utilisateur peut-il accéder à une ressource en devinant un ID ?
- Les actions sont-elles vérifiées côté **serveur** (pas seulement masquées côté frontend) ?

## 4. Validation des entrées

- Un schéma Zod valide-t-il le body de chaque requête POST/PUT ?
- Les types, longueurs maximales et formats sont-ils contraints ?
- Les inputs passent-ils par `sanitizeInputs` (DOMPurify) pour prévenir le XSS ?
- Les paramètres d'URL (`:id`) sont-ils validés (integer, UUID, etc.) ?

## 5. Exposition de données sensibles

- Les réponses API retournent-elles des champs sensibles inutiles (mots de passe hashés, tokens internes) ?
- Les logs contiennent-ils des données personnelles ou des secrets ?
- Les messages d'erreur révèlent-ils des informations sur la structure interne (stack traces, noms de tables) ?
- Les variables d'environnement sensibles sont-elles absentes du code versionné ?

## 6. Sécurité HTTP

- Les headers de sécurité sont-ils présents (CSP, HSTS, X-Frame-Options, etc.) ?
- CORS est-il configuré avec des origines explicites (pas `*`) ?
- Le rate limiting est-il en place sur les endpoints sensibles (`/login`, reset password) ?
- Les fichiers uploadés (si applicable) sont-ils validés (type MIME, taille, contenu) ?

## 7. Dépendances

- Y a-t-il des dépendances npm avec des vulnérabilités connues ? (`npm audit`)
- Les dépendances sont-elles à jour sur les patches de sécurité ?

## Format de réponse

Pour chaque vulnérabilité trouvée :
- **Fichier + ligne**
- **Sévérité** : 🔴 Critique / 🟠 Haute / 🟡 Moyenne / 🔵 Faible
- **Type** : (ex: IDOR, SQL Injection, Missing Auth...)
- **Description** : comment cette vulnérabilité peut être exploitée
- **Correction** : code corrigé ou étapes précises

Si un axe est sûr, l'indiquer avec ✅ en une ligne.

Terminer par un **score de risque global** et les 3 actions prioritaires à mener.
