# base-images

Images de base de tous les produits katappult (GTP, katappult, IMB, Vivasites) : une seule source, scannée et mise à
jour ici. Les produits n'utilisent plus directement `eclipse-temurin` ou `node`.

| Image | Contenu | Utilisée par |
|---|---|---|
| `ghcr.io/katappult/base-jre25` | JRE 25 Temurin, Ubuntu 24.04 | backends Java (Jib, `jib.base.image` du `pom.xml`) |
| `ghcr.io/katappult/base-jdk25` | JDK 25 Temurin (contient la JRE 25), Ubuntu 24.04 | katappult `sandbox-runner` |
| `ghcr.io/katappult/base-node22` | Node 22 Alpine, sans npm / npx / corepack, avec `envsubst` | étape `runner` des frontends |
| `ghcr.io/katappult/base-node22-build` | Node 22 Alpine avec npm | étapes `deps` / `builder` des frontends |

Toutes : correctifs OS (`apt-get upgrade` / `apk upgrade`), `ca-certificates`, `tzdata`, `TZ=UTC`. Pas de `USER` :
chaque produit choisit le sien (OpenShift impose de toute façon un UID aléatoire, groupe 0).

## Tags

- `<version>-<AAAAMMJJHHMM>` (ex. `25.0.5_8-202611090400`) : **immuable**. C'est ce que les produits épinglent, avec le
  digest : `ghcr.io/katappult/base-jre25:25.0.5_8-202611090400@sha256:…`.
- `25` / `22` : **mobile**, dernière publication (référence de comparaison du workflow, jamais utilisé par un produit).

## Cycle de mise à jour

```
Dependabot (lundi 03:00)  PR "deps(amont)" : nouveau digest eclipse-temurin / node   -> tu merges (CI = build + Trivy)
build.yml (lundi 04:00, push main, manuel)
   build -> Trivy (BLOQUANT sur CRITICAL corrigeable) -> publication SI le contenu a changé (paquets, version, Dockerfile)
   -> propagation : PR "chore: images de base à jour" vers develop dans chaque produit qui utilise l'image
produit : merge de la PR -> Deploy dev ne reconstruit QUE les composants concernés -> Deploy prod (version X.Y.Z)
prod-image-scan.yml des produits (lundi 06:30) : scan des images réellement en prod
```

Le rebuild du lundi prend les correctifs OS même sans nouvelle version amont. Contenu identique : rien n'est publié,
aucune PR.

## Alerte de sécurité

- **Scan d'un produit rouge le lundi** (CVE dans l'OS ou dans le runtime) : la PR « images de base » du jour la
  corrige en général. Merger, Deploy dev, puis Deploy prod. Si la PR n'existe pas (pas encore de correctif amont),
  attendre ou justifier la CVE dans `.trivyignore`.
- **Build d'une image de base rouge** : CVE CRITICAL corrigeable pas encore corrigée par l'amont. L'ancienne image
  reste en place, rien n'est cassé. Attendre le correctif (Dependabot / rebuild suivant), ou justifier dans
  `.trivyignore` (raison + date de revue).
- Reconstruire tout de suite : Actions → **Images de base** → **Run workflow** (`force` pour publier même sans changement),
  ou `gh workflow run build.yml -R katappult/base-images -f force=true`.

## Configuration

- Packages GHCR **publics** (ni code ni secret dedans) : org `katappult` → **Packages** → chaque `base-*` →
  **Package settings** → **Change visibility** → Public. Sinon, chaque repo produit doit avoir l'accès `Read`
  (**Manage Actions access**), et Jib / Docker / Cloud Build des identifiants.
- Secret `BASE_IMAGES_TOKEN` : PAT fine-grained, repos `global-travel-planner`, `katappult-cloud`, `imb-backmarket`,
  `vivasite`, droits **Contents** et **Pull requests** en lecture/écriture.
  `gh secret set BASE_IMAGES_TOKEN -R katappult/base-images`
- Ajouter un produit : `REPOS` dans `.github/workflows/build.yml`, puis y référencer l'image épinglée (tag + digest).
