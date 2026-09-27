# Guide de soumission au Pi Core Team

Checklist a suivre avant de soumettre Vitrines.pi pour validation Testnet puis Mainnet.

## 1. Conformite technique (Pi Platform Policies)

- [ ] Authentification exclusivement via `pi.authenticate()` — aucun systeme
      email/mot de passe parallele.
- [ ] Paiements exclusivement via `pi.createPayment()` et les endpoints
      officiels `/v2/payments/*` (approve/complete/cancel).
- [ ] Aucune cle privee ni fonds utilisateur stockes cote application
      (verifie : le backend ne detient que des references de paiement, pas de
      wallet custodial).
- [ ] Minimisation des donnees : seuls `piUid` et `username` sont persistes
      pour l'identite (voir `prisma/schema.prisma`, modele `User`).
- [ ] Script SDK charge depuis `https://sdk.minepi.com/pi-sdk.js` uniquement,
      sans mirroir ni modification.
- [ ] Scopes limites a `['username', 'payments']`.
- [ ] Escrow purement applicatif (pas de smart contract on-chain) — voir
      `backend/src/services/escrowService.ts`.
- [ ] Design original, sans logo Pi officiel reutilise.
- [ ] Nom de domaine de production ne commence pas par "pi".
- [ ] `sandbox: true` actif pendant toute la phase de test Testnet.
- [ ] Aucune promesse de rendement ni contenu speculatif dans l'UI ou le
      marketing de l'app.

## 2. Tests fonctionnels avant soumission

- [ ] Creer un compte Pionnier de test (Sandbox) et se connecter.
- [ ] Publier une annonce, la modifier, la supprimer.
- [ ] Effectuer un achat complet : creation transaction -> approbation ->
      completion -> confirmation de reception -> transaction `COMPLETED`.
- [ ] Verifier la liberation automatique apres le delai configure
      (reduire temporairement `ESCROW_AUTO_RELEASE_HOURS` en environnement de
      test pour accelerer la verification).
- [ ] Ouvrir un litige et le resoudre depuis le panel `/admin` (compte avec
      `role: ADMIN` en base).
- [ ] Verifier le chat temps reel entre acheteur et vendeur (Socket.io).
- [ ] Verifier l'affichage des notes/avis apres transaction terminee.

## 3. Securite

- [ ] Rate limiting actif sur les routes sensibles (`/api/auth/verify`,
      `/api/transactions/*`) — voir `middleware/rateLimit.ts`.
- [ ] Validation stricte de tous les payloads entrants via Zod.
- [ ] JWT applicatif signe avec un secret fort et distinct par environnement.
- [ ] HTTPS obligatoire en production (reverse proxy / certificat TLS).
- [ ] Journal d'audit des actions admin (`AdminLog`) verifie et non
      modifiable depuis le frontend.
- [ ] Messages de chat chiffres en base (AES-256-GCM, voir `utils/crypto.ts`).

## 4. Documentation a fournir au Pi Core Team

- [ ] URL de l'application en environnement Testnet (Pi Browser).
- [ ] Description fonctionnelle courte (marketplace, escrow, commission 1.5%).
- [ ] Captures d'ecran des flux cles : connexion, achat, confirmation,
      messagerie, resolution de litige.
- [ ] Politique de confidentialite et conditions d'utilisation (a rediger
      separement, non incluses dans ce depot).
- [ ] Contact support et procedure de gestion des litiges.

## 5. Passage en Mainnet

- [ ] Basculer `PI_SANDBOX=false` et `NEXT_PUBLIC_PI_SANDBOX=false`.
- [ ] Verifier que le Client ID Pi correspond bien a l'app enregistree en
      production sur developer.minepi.com.
- [ ] Rejouer l'integralite des tests fonctionnels de la section 2 en
      conditions reelles avec de tres petits montants.
- [ ] Surveiller les logs et metriques (`/api/admin/metrics`) durant la
      periode de montee en charge initiale.
