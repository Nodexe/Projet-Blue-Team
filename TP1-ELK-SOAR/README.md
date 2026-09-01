# TP1 — Détection SIEM & Réponse Automatisée SOAR

![SIEM](https://img.shields.io/badge/-SIEM-005571)
![SOAR](https://img.shields.io/badge/-SOAR-orange)
![MITRE](https://img.shields.io/badge/-MITRE-blue)
![Docker](https://img.shields.io/badge/-Docker-2496ED)
![pfSense](https://img.shields.io/badge/-pfSense-black)
![Linux](https://img.shields.io/badge/-Linux-yellow)

## Objectif

Ce TP démontre la capacité à construire une chaîne de défense SOC complète et autonome, de la détection à la remédiation, **sans intervention humaine**. Une attaque par brute force SSH est détectée en temps réel par un SIEM (Elastic Stack), déclenche automatiquement un playbook SOAR (Shuffle) qui extrait l'IP malveillante et l'envoie au firewall (pfSense) pour blocage immédiat.

L'objectif n'est pas de configurer des outils isolément, mais de prouver une **orchestration end-to-end** typique d'un SOC moderne, réduisant le MTTR (Mean Time To Respond) par l'automatisation de la réponse à incident.

**Trois points clés de ce projet :**
- Chaîne complète, pas un outil isolé
- Automatisation réelle (pas de playbook exécuté à la main)
- Mapping MITRE ATT&CK (méthodologie)

---

## Architecture


<img width="819" height="1030" alt="finale" src="https://github.com/user-attachments/assets/c4776665-6c79-4243-8dc6-8dfd52466bac" />



**4 VMs** :

| VM | IP | Rôle |
|---|---|---|
| VM-DOCKER | 192.168.56.10 | Elasticsearch, Kibana, Fleet Server, Shuffle |
| VM-CIBLE | 192.168.56.20 | Serveur SSH surveillé, Elastic Agent |
| VM-KALI | 192.168.57.50 | Machine d'attaque (Hydra) |
| VM-PFSENSE | LAN 192.168.56.1 / WAN 192.168.57.1 | Firewall, point de coupure réseau |

---

## Stack technique

- **SIEM** : Elasticsearch, Kibana, Fleet / Elastic Agent
- **SOAR** : Shuffle
- **Firewall** : pfSense + package REST API
- **Infrastructure** : Docker Compose
- **Attaque simulée** : Hydra (brute force SSH)
- **Détection** : règle de seuil Kibana, mappée sur **MITRE ATT&CK T1110** (Brute Force)

---

## Déroulement de la chaîne

1. **Attaque** — Hydra lance une attaque par brute force SSH contre VM-CIBLE
2. **Ingestion** — Elastic Agent collecte les logs d'authentification (`/var/log/auth.log`) et les envoie à Elasticsearch
3. **Détection** — une règle Kibana déclenche une alerte critique après 5 échecs d'authentification en moins de 2 minutes depuis la même IP
4. **Automatisation** — l'alerte envoie un Webhook à Shuffle, qui exécute un playbook : appel à l'API REST de pfSense et application du changement
5. **Remédiation** — pfSense ajoute l'IP à un alias `SOAR_Blocklist`, bloqué par une règle firewall en amont — l'attaquant est banni automatiquement

*(Captures d'écran à insérer : Discover avec logs bruts, alerte déclenchée, playbook Shuffle exécuté, IP bloquée dans pfSense)*



## Ce que ce projet démontre

- Déploiement et administration d'une stack SIEM/SOAR conteneurisée (Docker Compose)
- Écriture de règles de détection et mapping MITRE ATT&CK
- Construction d'un playbook SOAR avec appel API REST
- Segmentation réseau et administration pare-feu (pfSense)
- Troubleshooting réseau et applicatif en environnement conteneurisé multi-VM

---

## Limites connues et pistes d'amélioration

- **TLS auto-signé** : l'enrôlement des agents Elastic utilise `--insecure` (acceptable en lab, à remplacer par une CA interne en production)
- **Licence Elastic** : les actions sur les règles de détection nécessitent une licence (trial utilisé ici)
- **Package pfSense REST API** : non officiel, doit être réinstallé après chaque mise à jour de pfSense
- **Ressources limitées** : Elasticsearch réduit (1.2 Go) pour tenir sur 8 Go de RAM — non représentatif d'un déploiement de production

---

## Structure du repo

```
tp1-elk-soar/
├── README.md
├── docker-compose.yml
├── kibana.yml
├── .env.example
├── .gitignore
├── screenshots/
└── playbook-shuffle-export.json
```
