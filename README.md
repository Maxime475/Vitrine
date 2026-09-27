# Vitrines.pi

Marketplace decentralisee pour la communaute Pi Network. Les Pionniers achetent et
vendent des biens/services en Pi (π), avec un systeme d'escrow applicatif et une
commission de 1.5 % sur les transactions reussies.

Conforme aux Pi Platform Policies : authentification exclusive via Pi SDK, paiements
uniquement via les API Pi officielles (A2U/U2A), aucune conservation de cles privees
ou de fonds utilisateurs, minimisation des donnees (piUid + username uniquement).

## Stack technique

**Frontend** : Next.js 14 (App Router) + TypeScript, Tailwind CSS, Zustand, React Query,
Axios, Socket.io client, @pinetwork-js (script officiel charge en CDN).

**Backend** : Node.js 20 + Express, PostgreSQL + Prisma, JWT (session applicative),
Zod, Socket.io, BullMQ + Redis (cron de liberation d'escrow a 72h).

## Structure du depot

```
vitrines-pi/
├── backend/          # API Express + Prisma + logique d'escrow
├── frontend/          # Application Next.js
├── postman/           # Collection Postman pour tester l'API
├── docker-compose.yml # Postgres + Redis + backend + frontend
└── GUIDE_SOUMISSION_PI.md
```

## Demarrage rapide (local, sans Docker)

### 1. Base de donnees et Redis

Installez PostgreSQL et Redis localement, ou utilisez le `docker-compose.yml`
fourni pour ne lancer que ces deux services :

```bash
docker compose up -d postgres redis
```

### 2. Backend

```bash
cd backend
cp .env.example .env
# Renseignez DATABASE_URL, PI_API_KEY, JWT_SECRET, etc.
npm install
npx prisma migrate dev --name init
npm run dev
```

L'API demarre sur `http://localhost:4000`. Le worker d'auto-liberation d'escrow
est lance automatiquement en mode developpement ; en production, lancez-le a
part avec `npm run worker`.

### 3. Frontend

```bash
cd frontend
cp .env.local.example .env.local
# Renseignez NEXT_PUBLIC_PI_CLIENT_ID (developer.minepi.com)
npm install
npm run dev
```

L'application demarre sur `http://localhost:3000`. Elle doit etre ouverte dans
**Pi Browser** pour que le SDK Pi fonctionne (authentification et paiements).

### 4. Tout lancer avec Docker Compose

```bash
docker compose up --build
```

## Variables d'environnement essentielles

| Variable | Ou | Description |
|---|---|---|
| `PI_API_KEY` | backend/.env | Cle serveur obtenue sur developer.minepi.com |
| `PI_SANDBOX` | backend/.env | `true` pour Testnet, `false` pour Mainnet |
| `DATABASE_URL` | backend/.env | Connexion PostgreSQL |
| `JWT_SECRET` | backend/.env | Secret de signature des sessions applicatives |
| `NEXT_PUBLIC_PI_CLIENT_ID` | frontend/.env.local | Client ID Pi de l'application |
| `NEXT_PUBLIC_PI_SANDBOX` | frontend/.env.local | Doit correspondre a PI_SANDBOX cote backend |

## Flux d'achat (escrow)

1. L'acheteur clique sur "Acheter avec Pi" -> une transaction locale `PENDING_APPROVAL`
   est creee, puis `pi.createPayment()` est appele cote client.
2. `onReadyForServerApproval` -> le backend approuve le paiement aupres de l'API Pi
   et enregistre le `piPaymentId`.
3. `onReadyForServerCompletion` -> le backend enregistre le `txid`, la transaction
   passe en `PENDING_DELIVERY` (fonds bloques en escrow applicatif).
4. L'acheteur confirme la reception -> la transaction passe a `DELIVERED` puis
   `COMPLETED`, l'annonce passe a `SOLD`.
5. Si l'acheteur ne confirme rien, le job `autoReleaseEscrow` (toutes les 30 min)
   libere automatiquement les transactions livrees depuis plus de 72h sans litige.
6. A tout moment avant liberation, acheteur ou vendeur peuvent ouvrir un litige ;
   un administrateur le resout via le panel `/admin`.

## Tester l'API

Importez `postman/Vitrines.pi.postman_collection.json` dans Postman ou Insomnia.
Recuperez un `accessToken` Pi valide via le flux d'authentification frontend pour
tester `/api/auth/verify`, puis reutilisez le JWT retourne comme variable `token`.

## Points de vigilance avant soumission

- Ne jamais committer les fichiers `.env` (deja exclus via `.gitignore`).
- Tester integralement sur Testnet (sandbox) avant toute demande de Mainnet.
- Auditer la securite (rate limiting, validation Zod, JWT) avant soumission.
- Toujours valider les tokens Pi cote backend, jamais se fier au frontend seul.
- Respecter les rate limits de l'API Pi officielle.

Voir `GUIDE_SOUMISSION_PI.md` pour la checklist complete de soumission au Pi Core Team.
