# Image de base des outils de build (sandbox de jobs) : JDK 25 Temurin (contient la JRE 25) sur Ubuntu 24.04.
# Amont épinglé par digest, mis à jour par Dependabot (PR). Correctifs OS appliqués à chaque build (lundi + manuel).
# Pas de USER ici : chaque produit choisit le sien.
FROM eclipse-temurin:25-jdk-noble@sha256:95e350ad1eb738f943e141640f7e3d797e483359a3b33c3864255bfd1ad71f8a

ENV DEBIAN_FRONTEND=noninteractive TZ=UTC LANG=C.UTF-8
RUN apt-get update \
 && apt-get upgrade -y --no-install-recommends \
 && apt-get install -y --no-install-recommends ca-certificates tzdata \
 && apt-get clean && rm -rf /var/lib/apt/lists/* /var/log/apt /var/log/dpkg.log /tmp/*
