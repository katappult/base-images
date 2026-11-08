# base-images

Images de base communes, construites, scannées et mises à jour ici, pour que les applications n'utilisent plus
directement `eclipse-temurin` ou `node`.

| Image | Contenu | Usage |
|---|---|---|
| `ghcr.io/katappult/base-jre25` | JRE 25 Temurin, Ubuntu 24.04 | backends Java (base Jib : `jib.from.image`) |
| `ghcr.io/katappult/base-jdk25` | JDK 25 Temurin (contient la JRE 25), Ubuntu 24.04 | outillage de build (compilation dans un conteneur) |
| `ghcr.io/katappult/base-node22` | Node 22 Alpine, sans npm / npx / corepack, avec `envsubst` | étape d'exécution d'un frontend Node (Next.js standalone…) |
| `ghcr.io/katappult/base-node22-build` | Node 22 Alpine, avec npm | étapes `npm ci` / build d'un frontend |

Toutes les images ont :
- les correctifs OS (`apt-get upgrade` / `apk upgrade`) ;
- `ca-certificates` et `tzdata`, avec `TZ=UTC` ;
- aucune `USER` : l'application choisit le sien. Elles fonctionnent avec un UID arbitraire et le groupe 0 (OpenShift,
  SCC restricted-v2), ce que le workflow teste.

## Tags

- `<version>-<AAAAMMJJHHMM>` (ex. `25.0.5_8-202611081948`) : **immuable**. C'est le tag à utiliser, avec le digest :
  `ghcr.io/katappult/base-jre25:25.0.5_8-202611081948@sha256:…`
- `25` / `22` : **mobile**, il désigne toujours la dernière publication. Il sert de référence de comparaison au
  workflow ; ne pas l'utiliser dans une application.

Exemple de Dockerfile :

```dockerfile
FROM ghcr.io/katappult/base-node22-build:<tag>@sha256:<digest> AS builder
WORKDIR /app
COPY . .
RUN npm ci && npm run build

FROM ghcr.io/katappult/base-node22:<tag>@sha256:<digest>
WORKDIR /app
COPY --from=builder /app/.next/standalone ./
USER 10001:0
CMD ["node", "server.js"]
```

## Cycle de mise à jour

```
Dependabot (lundi 03:00)   PR "deps(amont)" : nouveau digest eclipse-temurin / node      -> merge (CI = build + Trivy)
build.yml (lundi 04:00, push sur main, manuel)
   build -> test UID arbitraire -> Trivy (BLOQUANT sur CRITICAL corrigeable)
   -> publication SI le contenu a changé (Dockerfile, version, liste des paquets installés)
   -> propagation : PR "chore: images de base à jour" vers develop dans chaque repo configuré qui utilise l'image
```

Le rebuild du lundi récupère les correctifs OS même sans nouvelle version amont. Si le contenu est identique, rien
n'est publié et aucune PR n'est ouverte.

## Rapports de sécurité

- **Security → Code scanning** : vulnérabilités Trivy de chaque image (filtre `Tool: Trivy`, une catégorie par image),
  mises à jour à chaque build (au moins chaque lundi).
- Chaque run (Actions → **Images de base**) : résumé HIGH / CRITICAL par image, et artefact `rapport-<image>` (rapport
  texte, SARIF, liste des paquets installés), conservé 90 jours.
- Images publiées : onglet **Packages** du repo (ou `ghcr.io/katappult/base-*`).

## Alerte de sécurité

- **Build rouge** : une CVE CRITICAL corrigeable n'a pas encore de correctif amont. La dernière image publiée reste
  en place, rien n'est cassé. Il faut attendre le correctif (Dependabot ou rebuild suivant), ou justifier la CVE dans
  `.trivyignore` (raison + date de revue).
- **Reconstruire tout de suite** : Actions → **Images de base** → **Run workflow** (`force` pour publier même sans
  changement), ou `gh workflow run build.yml -R katappult/base-images -f force=true`.

## Configuration de la propagation

Le code n'en dit rien ; tout se règle dans les settings du repo :

- **Variable** `PRODUCT_REPOS` : repos à mettre à jour, `owner/repo` séparés par des espaces. Settings → Secrets and
  variables → Actions → **Variables**.
- **Secret** `BASE_IMAGES_TOKEN` : PAT fine-grained limité à ces repos, **Contents** et **Pull requests** en
  lecture/écriture.

Les noms des repos sont masqués dans les logs. Sans ces deux réglages, les images sont publiées sans être propagées.
