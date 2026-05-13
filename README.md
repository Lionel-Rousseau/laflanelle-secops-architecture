# Architecture SecOps & Continuité - Infrastructure Linux multi-sites

> Dossier technique d'une infrastructure Linux en production, conçue, déployée et opérée en autonomie depuis 2018.
> 7 années d'exploitation continue · 99,9 % d'uptime cumulé · défense en profondeur · PCA documenté et testé.
> 25+ années d’expérience en administration de systèmes Linux exposés.

---

## 📄 Dossier technique

Présentation complète de l'architecture, des contrôles de sécurité, 
des procédures de continuité et des retours d'expérience associés :  
[**LRO_Dossier_Technique_2026.pdf**](./LRO_Dossier_Technique_2026.pdf) - 16 pages.

### Vérification d'intégrité (optionnel)

Le PDF est signé numériquement avec ma clé OpenPGP.

**Fingerprint** : `111D 0326 757E C7EC B3EC  17F5 D28C 8EB0 557B 5620`

**Clé publique disponible** :
- Sur le keyserver : [keys.openpgp.org](https://keys.openpgp.org/vks/v1/by-fingerprint/111D0326757EC7ECB3EC17F5D28C8EB0557B5620)
- Dans ce dépôt : [signatures/lionel-rousseau-pubkey.asc](./signatures/lionel-rousseau-pubkey.asc)

**Signature détachée** : [signatures/LRO_Dossier_Technique_2026.pdf.asc](./signatures/LRO_Dossier_Technique_2026.pdf.asc)

***Vérification :
\`\`\`bash

***Import depuis le keyserver
gpg --keyserver hkps://keys.openpgp.org --recv-keys 111D0326757EC7ECB3EC17F5D28C8EB0557B5620

***Vérification de la signature
gpg --verify signatures/LRO_Dossier_Technique_2026.pdf.asc LRO_Dossier_Technique_2026.pdf
\`\`\`

## Pourquoi ce dépôt

Ce dépôt sert de vitrine technique. Il documente une infrastructure réelle qui supporte une activité économique 24/7 et traite chaque jour ~170 emails entrants et ~19 000 requêtes HTTP. Il s'adresse aux recruteurs, RSSI et responsables d'exploitation qui souhaitent évaluer concrètement ma pratique au-delà d'un CV.

Tous les éléments présentés ici ont été anonymisés : aucun hostname interne, adresse IP publique ou élément exploitable à des fins offensives n'est divulgué.

## Vue d'ensemble

L'infrastructure se déploie sur six environnements distincts :

- **2 serveurs dédiés OVH** (Ubuntu LTS) portant les services exposés à Internet : web, mail, MX secondaires, sauvegardes hors site.
- **3 sites physiques** interconnectés via un mesh VPN Tailscale : site principal hébergeant le cœur de l'infrastructure interne, site secondaire (bureau externe), site tertiaire (résidence secondaire avec domotique et exit node redondant).

La défense en profondeur est appliquée sur quatre niveaux : WAF Cloudflare en frontal, OPNsense NGFW + Zenarmor au périmètre, segmentation VLAN avec deny-by-default à l'intérieur du LAN, durcissement basé sur les benchmarks CIS au niveau hôte. Un SOC interne basé sur Wazuh (XDR/SIEM) et Security Onion (NSM/IDS) assure la corrélation des sources hôte et réseau, ainsi que la détection des menaces périmétriques.

## Stack technique

| Domaine | Outils |
|---|---|
| Systèmes | Debian, Ubuntu Server LTS, Rocky, Oracle Linux, FreeBSD (OPNsense) |
| Services exposés | Apache 2, Nginx, Postfix, Dovecot, Rspamd, Unbound DNS, MySQL |
| Virtualisation | Proxmox VE, Proxmox Backup Server, Docker |
| Réseau & sécurité | OPNsense NGFW, Zenarmor IPS, Tailscale (mesh VPN), Cloudflare WAF/CDN, Let's Encrypt |
| Détection & supervision | Wazuh, Security Onion (Suricata + Zeek), Zabbix, Fail2ban, SNORT, RKhunter, Tripwire, ClamAV |
| Messagerie | SPF, DKIM, DMARC, OpenARC, OpenPGP, TLS 1.2+ strict |
| Sauvegardes | Rsync/restic (AES-256, conteneurs LUKS), Proxmox Backup Server, règle 3-2-1 |
| Automatisation | Ansible, Bash, Git, systemd timers |
| Gestion des secrets | Proton Pass, pass (GPG + Yubikey), Kleopatra |

## Indicateurs opérationnels

| Indicateur | Valeur |
|---|---|
| Uptime cumulé sur 7 ans | 99,9 % |
| Volume mail traité | ~170 messages/jour - ~63 000/an sur 13 domaines |
| Trafic web | ~19 000 requêtes HTTP/jour |
| RPO/RTO messagerie | 15 min / 1 h, testé mensuellement |
| RPO/RTO web e-commerce | 1 h / 1 h, testé mensuellement |
| Incidents avec fuite de données | 0 |
| Incidents > 3 h d'interruption | 0 |

## Documents disponibles

- [`docs/Architecture.pdf`](./docs/Architecture.pdf) - dossier technique complet (15 pages) : contexte, architecture, contrôles de sécurité, plan de continuité, automatisation, roadmap, retours d'expérience.
- [`docs/Network-Diagram.pdf`](./docs/Network-Diagram.pdf) - diagramme d'architecture réseau anonymisé.

## Points forts mis en avant

- **Visibilité réseau en couches.** NSM passif (Security Onion + Suricata + Zeek) connecté à un port SPAN du switch L2+ pour analyser le trafic nord-sud, y compris le 10 Gbps en passthrough vers OPNsense. Défense est-ouest sur les flux inter-VLAN assurée par le filtrage OPNsense (ACL deny-by-default) et par la corrélation Wazuh des logs hôtes. Extension NSM aux flux est-ouest inscrite à la roadmap.
- **PCA interne, bascule du firewall et services critiques.** Nœud Proxmox de secours (HP ProDesk) hébergeant des copies de la VM OPNsense (NGFW + IPS + Tailscale Router) et de la VM FreeRADIUS/Unbound, synchronisées toutes les 24 heures avec rétention de 2 jours. Procédure de bascule documentée, testée régulièrement, RTO cible inférieur à 15 minutes sur les services critiques (accès Internet, authentification Wi-Fi).
- **PCA externe distribué géographiquement.** Les deux serveurs OVH dédiés (Sites FR A et FR B) sont hébergés sur des datacenters distincts pour résister à une panne site. MX secondaire (priorité 20) sur FR B garantissant la continuité de la réception mail si FR A devient indisponible. Sauvegardes restic/rsync chiffrées répliquées de FR A vers FR B, reconstruction possible depuis backup en cas de perte totale. Le retour à une architecture sur 3 nœuds externes (configuration en place entre 2018 et 2024) est inscrit à la roadmap pour découpler complètement web et mail et améliorer la résilience web.
- **Sauvegardes régulières vérifiées.** Règle 3-2-1 avec restic/rsync chiffré (LUKS) répliqué hors-site sur OVH, et copie quotidienne sur support amovible pour les données NAS les plus sensibles. Tests de restauration mensuels effectifs avec rotation des cibles, plusieurs problèmes silencieux détectés et corrigés grâce à ces tests sur la période.
- **Cloisonnement strict du lab offensif.** VLAN Lab isolé du LAN de production, sortie Internet en deny-all + whitelist explicite des cibles d'entraînement, activation à la demande uniquement. Migration sur un Proxmox dédié physiquement séparé inscrite à la roadmap pour éliminer définitivement le risque résiduel de pivotement.

## Roadmap

Quelques chantiers en cours ou programmés pour les douze prochains mois :

- Intégration de Security Onion avec Wazuh.
- Industrialisation de playbooks Ansible publics anonymisés (durcissement Debian/Ubuntu, déploiement Wazuh, déploiement Postfix/Rspamd).
- Bascule semi-automatique du PCA OPNsense via heartbeat CARP/VRRP.
- IDS DNS sur Unbound (passive DNS + détection DGA / tunneling).
- POC HashiCorp Vault dans le VLAN Lab - étude de la rotation dynamique de secrets.
- Découplage des services web & mail pour un retour à 3 serveurs externes.
- Étude NixOS pour les serveurs OVH publics (reproductibilité atomique, rollback transactionnel).

## À propos

**Lionel Rousseau** - Administrateur Systèmes Linux & Sécurité Opérationnelle
Certifié CompTIA Security+ et CySA+ · TOEIC 980 (C1/C2)
Disponible pour un poste en exploitation Linux, administration systèmes ou SecOps.

📧 [`lionel@rousseau.kr`](mailto:lionel@rousseau.kr)
💼 [LinkedIn](https://www.linkedin.com/in/lionel-rousseau-kr/)
[GitHub](https://github.com/Lionel-Rousseau).

## Licence

Documentation publiée sous [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — réutilisation autorisée avec attribution et sous la même licence.

## Vérification d'authenticité

Les PDFs sont signés avec ma clé GPG. Pour vérifier qu'ils n'ont pas été modifiés :

# Importer ma clé publique
gpg --import lionel-rousseau-pubkey.asc

# Vérifier les signatures détachées
gpg --verify docs/Architecture.pdf.sig docs/Architecture.pdf
gpg --verify docs/Network-Diagram.pdf.sig docs/Network-Diagram.pdf

Empreinte de la clé : `111D 0326 757E C7EC B3EC 17F5 D28C 8EB0 557B 5620`
