# Image de base des backends Java (Jib) : JRE 25 Temurin sur Ubuntu 24.04 (noble).
# Amont épinglé par digest, mis à jour par Dependabot (PR). Correctifs OS appliqués à chaque build (lundi + manuel).
# Pas de USER ici : chaque produit choisit le sien (OpenShift impose de toute façon un UID aléatoire, groupe 0).
FROM eclipse-temurin:25-jre-noble@sha256:e3b6f7887a50f20d8f07abf279d8a81a5953fb398a9555bd9edd56b9bd15968c

ENV DEBIAN_FRONTEND=noninteractive TZ=UTC LANG=C.UTF-8
RUN apt-get update \
 && apt-get upgrade -y --no-install-recommends \
 && apt-get install -y --no-install-recommends ca-certificates tzdata \
 && apt-get clean && rm -rf /var/lib/apt/lists/* /var/log/apt /var/log/dpkg.log /tmp/*
