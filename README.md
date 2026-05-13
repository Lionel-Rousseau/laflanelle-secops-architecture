# Architecture SecOps & Continuité - Infrastructure Linux multi-sites

> Dossier technique d'une infrastructure Linux en production, conçue, déployée
> et opérée en autonomie depuis 2018.
> 7 années d'exploitation continue · 99,9 % d'uptime cumulé · défense en
> profondeur · PCA documenté et testé.
> 25+ années d'expérience en administration de systèmes Linux exposés.

---

## 📄 Dossier technique

Présentation complète de l'architecture, des contrôles de sécurité,
des procédures de continuité et des retours d'expérience associés :
[**LRO_Dossier_Technique_2026.pdf**](./LRO_Dossier_Technique_2026.pdf) — 16 pages.

> Les documents sources (architecture texte et diagramme réseau anonymisé)
> sont disponibles séparément dans [`docs/`](./docs/).

---

## Pourquoi ce dépôt

Ce dépôt sert de vitrine technique. Il documente une infrastructure réelle qui
supporte une activité économique 24/7 et traite chaque jour ~170 emails entrants
et ~19 000 requêtes HTTP. Il s'adresse aux recruteurs, RSSI et responsables
d'exploitation qui souhaitent évaluer concrètement ma pratique au-delà d'un CV.

Tous les éléments présentés ici ont été anonymisés : aucun hostname interne,
adresse IP publique ou élément exploitable à des fins offensives n'est divulgué.

## Vue d'ensemble

L'infrastructure se déploie sur six environnements distincts :

- **2 serveurs dédiés OVH** (Ubuntu LTS) portant les services exposés à
  Internet : web, mail, MX secondaires, sauvegardes hors site.
- **3 sites physiques** interconnectés via un mesh VPN Tailscale : site
  principal hébergeant le cœur de l'infrastructure interne, site secondaire
  (bureau externe), site tertiaire (résidence secondaire avec domotique et
  exit node redondant).

La défense en profondeur est appliquée sur quatre niveaux : WAF Cloudflare
en frontal, OPNsense NGFW + Zenarmor au périmètre, segmentation VLAN avec
deny-by-default à l'intérieur du LAN, durcissement basé sur les benchmarks
CIS au niveau hôte. Un SOC interne basé sur Wazuh (XDR/SIEM) et Security
Onion (NSM/IDS) assure la corrélation des sources hôte et réseau ainsi que
la détection des menaces périmétriques.

## Stack technique

| Domaine | Outils |
|---|---|
| Systèmes | Debian, Ubuntu Server LTS, Rocky, Oracle Linux, FreeBSD (OPNsense) |
| Services exposés | Apache 2, Nginx, Postfix, Dovecot, Rspamd, Unbound DNS, MySQL |
| Virtualisation | Proxmox VE, Proxmox Backup Server, Docker |
| Réseau & sécurité | OPNsense NGFW, Zenarmor IPS, Tailscale, Cloudflare WAF/CDN, Let's Encrypt |
| Détection & supervision | Wazuh, Security Onion (Suricata + Zeek), Zabbix, Fail2ban, Snort, RKhunter, Tripwire, ClamAV |
| Messagerie | SPF, DKIM, DMARC, OpenARC, OpenPGP, TLS 1.2+ strict |
| Sauvegardes | Rsync/restic (AES-256, conteneurs LUKS), Proxmox Backup Server, règle 3-2-1 |
| Automatisation | Ansible, Bash, Git, systemd timers |
| Gestion des secrets | Proton Pass, pass (GPG + Yubikey), Kleopatra |

## Indicateurs opérationnels

| Indicateur | Valeur |
|---|---|
| Uptime cumulé sur 7 ans | 99,9 % |
| Volume mail traité | ~170 messages/jour · ~63 000/an sur 13 domaines |
| Trafic web | ~19 000 requêtes HTTP/jour |
| RPO/RTO messagerie | 15 min / 1 h, testé mensuellement |
| RPO/RTO web e-commerce | 1 h / 1 h, testé mensuellement |
| Incidents avec fuite de données | 0 |
| Incidents > 3 h d'interruption | 0 |

## Points forts mis en avant

- **Visibilité réseau en couches.** NSM passif (Security Onion + Suricata +
  Zeek) connecté à un port SPAN du switch L2+ pour analyser le trafic nord-sud,
  y compris le 10 Gbps en passthrough vers OPNsense. Défense est-ouest assurée
  par le filtrage OPNsense (ACL deny-by-default) et la corrélation Wazuh des
  logs hôtes.
- **PCA interne.** Nœud Proxmox de secours (HP ProDesk) hébergeant des copies
  de la VM OPNsense et de la VM FreeRADIUS/Unbound, synchronisées toutes les
  24 heures. Procédure de bascule documentée et testée, RTO cible < 15 minutes.
- **PCA externe distribué géographiquement.** Deux serveurs OVH sur datacenters
  distincts. MX secondaire garantissant la continuité mail. Sauvegardes
  restic/rsync chiffrées répliquées hors site.
- **Sauvegardes régulières vérifiées.** Règle 3-2-1, tests de restauration
  mensuels effectifs avec rotation des cibles. Plusieurs problèmes silencieux
  détectés et corrigés grâce à ces tests.
- **Cloisonnement strict du lab offensif.** VLAN Lab isolé du LAN de
  production, sortie Internet deny-all + whitelist explicite, activation
  à la demande uniquement.

## Roadmap

- Intégration Security Onion ↔ Wazuh.
- Industrialisation de playbooks Ansible publics anonymisés.
- Bascule semi-automatique du PCA OPNsense via CARP/VRRP.
- IDS DNS sur Unbound (passive DNS + détection DGA / tunneling).
- POC HashiCorp Vault dans le VLAN Lab.
- Découplage web & mail - retour à 3 serveurs externes.
- Étude NixOS pour les serveurs OVH publics.

---

## À propos

**Lionel Rousseau** - Administrateur Systèmes Linux & Sécurité Opérationnelle
Certifié CompTIA Security+ et CySA+ · TOEIC 980 (C1/C2)
Disponible pour un poste en exploitation Linux, administration systèmes ou SecOps.

📧 [`lionel@rousseau.kr`](mailto:lionel@rousseau.kr)
💼 [LinkedIn](https://www.linkedin.com/in/lionel-rousseau-kr/)
🇫🇷 [Profile](https://lionel.rousseau.kr/)

---

## Vérification d'intégrité

Le PDF du dossier technique est signé numériquement avec ma clé OpenPGP.

**Fingerprint** : `111D 0326 757E C7EC B3EC  17F5 D28C 8EB0 557B 5620`

**Clé publique disponible** :
- Sur le keyserver : [keys.openpgp.org](https://keys.openpgp.org/vks/v1/by-fingerprint/111D0326757EC7ECB3EC17F5D28C8EB0557B5620)
- Dans ce dépôt : [`signatures/lionel-rousseau-pubkey.asc`](./signatures/lionel-rousseau-pubkey.asc)

**Signature détachée** : [`signatures/LRO_Dossier_Technique_2026.pdf.asc`](./signatures/LRO_Dossier_Technique_2026.pdf.asc)

```bash
# Importer la clé publique depuis le keyserver
gpg --keyserver hkps://keys.openpgp.org \
    --recv-keys 111D0326757EC7ECB3EC17F5D28C8EB0557B5620

# Vérifier la signature du dossier technique
gpg --verify signatures/LRO_Dossier_Technique_2026.pdf.asc \
             LRO_Dossier_Technique_2026.pdf
```

## Licence

Documentation publiée sous
[Creative Commons BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/) -
réutilisation autorisée avec attribution et sous la même licence.
