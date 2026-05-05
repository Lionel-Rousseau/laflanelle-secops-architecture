# Architecture SecOps & Continuité — Infrastructure Linux multi-sites

> Dossier technique d'une infrastructure Linux en production, conçue, déployée et opérée en autonomie depuis 2018.
> Sept années d'exploitation continue · 99,9 % d'uptime cumulé · défense en profondeur · PCA documenté et testé.

---

## Pourquoi ce dépôt

Ce dépôt sert de vitrine technique. Il documente une infrastructure réelle — pas un lab — qui supporte une activité économique 24/7 et traite chaque jour ~172 emails entrants et ~19 000 requêtes HTTP. Il s'adresse aux recruteurs, RSSI et responsables d'exploitation qui souhaitent évaluer concrètement ma pratique au-delà d'un CV.

Tous les éléments présentés ici ont été anonymisés : aucun hostname interne, adresse IP publique ou élément exploitable à des fins offensives n'est divulgué.

## Vue d'ensemble

L'infrastructure se déploie sur six environnements distincts :

- **2 serveurs dédiés OVH** (Ubuntu LTS) portant les services exposés à Internet : web, mail, MX secondaires, sauvegardes hors site.
- **3 sites physiques** interconnectés via un mesh VPN Tailscale : site principal hébergeant le cœur de l'infrastructure interne, site secondaire (bureau externe), site tertiaire (résidence secondaire avec domotique et exit node redondant).

La défense en profondeur est appliquée sur quatre niveaux : WAF Cloudflare en frontal, OPNsense NGFW + Zenarmor au périmètre, segmentation VLAN avec deny-by-default à l'intérieur du LAN, durcissement CIS au niveau hôte. Un SOC interne basé sur Wazuh (XDR/SIEM) et Security Onion (NSM/IDS) assure la corrélation et la détection des mouvements latéraux.

## Stack technique

| Domaine | Outils |
|---|---|
| Systèmes | Debian, Ubuntu Server LTS, Rocky, Oracle Linux, FreeBSD (OPNsense) |
| Services exposés | Apache 2, Nginx, Postfix, Dovecot, Rspamd, Unbound DNS, MariaDB |
| Virtualisation | Proxmox VE, Proxmox Backup Server, Docker |
| Réseau & sécurité | OPNsense NGFW, Zenarmor IPS, Tailscale (mesh VPN), Cloudflare WAF/CDN, Let's Encrypt |
| Détection & supervision | Wazuh, Security Onion (Suricata + Zeek), Zabbix, Fail2ban, RKhunter, Tripwire, ClamAV |
| Messagerie | SPF, DKIM, DMARC, OpenARC, OpenPGP, TLS 1.3 strict |
| Sauvegardes | rsync/restic (AES-256, conteneurs LUKS), Proxmox Backup Server, règle 3-2-1 |
| Automatisation | Ansible, Bash, Git, systemd timers |
| Gestion des secrets | Proton Pass, pass (GPG + Yubikey), Kleopatra |

## Indicateurs opérationnels

| Indicateur | Valeur |
|---|---|
| Uptime cumulé sur 7 ans | 99,9 % |
| Volume mail traité | ~170 messages/jour · ~63 000/an |
| Trafic web | ~19 000 requêtes HTTP/jour |
| RPO/RTO messagerie | 15 min / 1 h, testé mensuellement |
| RPO/RTO web e-commerce | 1 h / 1 h, testé mensuellement |
| Incidents avec fuite de données | 0 |
| Incidents > 3 h d'interruption | 0 |

## Documents disponibles

- [`docs/Architecture.pdf`](./docs/Architecture.pdf) — dossier technique complet (~12 pages) : contexte, architecture, contrôles de sécurité, plan de continuité, automatisation, roadmap, retours d'expérience.
- [`docs/Network-Diagram.pdf`](./docs/Network-Diagram.pdf) — diagramme d'architecture anonymisé.

## Points forts mis en avant

- **Visibilité réseau étendue.** Security Onion connecté à un port SPAN du switch L2+ pour analyser le trafic LAN intégral, y compris le 10 Gbps en passthrough vers OPNsense — détection des mouvements latéraux est-ouest, pas seulement du périmètre nord-sud.
- **PCA opérationnel et testé.** Nœud Proxmox de secours (HP ProDesk) hébergeant des copies froides synchronisées de la VM OPNsense (NGFW + IPS + Tailscale Router) et de la VM FreeRADIUS/Unbound. Procédure de bascule documentée, testée semestriellement, RTO < 15 min.
- **Sauvegardes vérifiées, pas seulement déclarées.** Tests de restauration mensuels effectifs, journal d'exploitation tenu, deux problèmes silencieux détectés et corrigés sur la période grâce à cette discipline.
- **Cloisonnement strict du lab offensif.** VLAN Lab isolé du LAN de production, sortie Internet en deny-all + whitelist explicite, cibles d'entraînement activées à la demande uniquement.

## Roadmap publique

Quelques chantiers en cours ou programmés pour les douze prochains mois :

- Bascule semi-automatique du PCA OPNsense via heartbeat CARP/VRRP.
- Industrialisation de playbooks Ansible publics anonymisés (durcissement Debian, déploiement Wazuh, déploiement Postfix/Rspamd).
- IDS DNS sur Unbound (passive DNS + détection DGA / tunneling).
- POC HashiCorp Vault dans le VLAN Lab — étude de la rotation dynamique de secrets.
- Étude NixOS pour les serveurs OVH publics (reproductibilité atomique, rollback transactionnel).

## À propos

**Lionel Rousseau** — Administrateur Systèmes Linux & Sécurité Opérationnelle
Certifié CompTIA Security+ et CySA+ · TOEIC 980 (C1/C2)
Disponible pour un poste en exploitation Linux, administration systèmes ou SecOps.

📧 lionel@rousseau.kr

## Licence

Documentation publiée sous [Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) — réutilisation autorisée avec attribution et sous la même licence.
