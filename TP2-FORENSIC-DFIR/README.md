# TP2 — Investigation Numérique & Forensic (DFIR)

![DFIR](https://img.shields.io/badge/-DFIR-red)
![Forensic](https://img.shields.io/badge/-Forensic-purple)
![Linux](https://img.shields.io/badge/-Linux-yellow)
![LVM](https://img.shields.io/badge/-LVM-blue)
![Timeline](https://img.shields.io/badge/-Timeline-orange)

## Objectif

Ce TP démontre la capacité d'analyse technique **post-mortem** : retracer, après coup, les actions d'un attaquant sur un système compromis, sans avoir assisté à l'attaque en direct.

Un serveur (VM-CIBLE, réutilisée du [TP1](../tp1-elk-soar)) a été volontairement compromis via un scénario réaliste : accès initial par identifiants faibles, exploitation d'un binaire SUID mal configuré pour élever ses privilèges, puis création d'un compte de persistance caché. L'objectif est de reconstituer cette chronologie **uniquement à partir des preuves numériques**, en respectant les bonnes pratiques d'intégrité forensic (copie bit-à-bit, hash, analyse en lecture seule).

## Ce que ce projet démontre

- Acquisition d'image disque bit-à-bit et vérification d'intégrité (hash SHA-256)
- Montage forensic en lecture seule, y compris la prise en charge de volumes **LVM** (partitions non triviales)
- Threat hunting dans les journaux système (`auth.log`, `syslog`, historiques shell)
- Reconstitution de timeline d'incident à partir de sources multiples
- Identification d'une faille d'élévation de privilèges (SUID) et explication technique de son exploitation

## Scénario reconstitué

1. **Accès initial** — connexion SSH réussie sur un compte aux identifiants faibles, depuis la machine attaquante (VM-KALI)
2. **Reconnaissance** — commandes d'exploration (`whoami`, `id`)
3. **Exploitation** — élévation de privilèges via un binaire SUID mal configuré (`find` copié avec le bit SUID actif)
4. **Persistance** — création d'un compte caché (`backdoor_user`) avec les droits obtenus
5. **Fin de session** — déconnexion de l'attaquant

➡️ Timeline complète et preuves détaillées : [`timeline.md`](./timeline.md)
➡️ Guide pas à pas complet (acquisition, montage LVM, analyse) : [`GUIDE_DETAILLE.md`](./GUIDE_DETAILLE.md)

## Stack technique

- **Acquisition** : `qemu-img` (conversion VMDK → RAW), partage réseau VMware (hgfs)
- **Intégrité** : `sha256sum`
- **Montage forensic** : `losetup`, LVM (`pvscan`, `vgscan`, `vgchange`, `vgrename`), `mount -o ro`
- **Analyse** : `fdisk`, `grep`, exploration de logs Linux (`auth.log`, `syslog`, `.bash_history`)

## Limites connues et enseignements

- **Acquisition depuis l'hyperviseur (fichier `.vmdk`)** plutôt que `dd` interne à la VM compromise — évite d'utiliser des binaires potentiellement altérés par l'attaquant, plus proche d'une bonne pratique forensic réelle
- **LVM et intégrité de la preuve** : une divergence de hash a été constatée entre l'acquisition initiale et une analyse ultérieure, causée par `losetup` sans le flag `-r` combiné à l'activation LVM (`vgchange -ay`), qui écrit des métadonnées internes même lorsque le point de montage final est en lecture seule. Voir la section dédiée dans le guide détaillé — c'est une limite réelle et documentée de LVM en contexte forensic, pas une simple erreur de manipulation
- **Rotation des logs** : une partie des preuves clés (connexion SSH initiale) se trouvait dans un fichier archivé (`auth.log.1`), pas dans le fichier actif — rappel qu'une investigation doit toujours couvrir les logs rotés
- **`.bash_history` incomplet après un pivot de shell** : les commandes exécutées depuis le shell obtenu via l'exploit SUID (`/bin/sh`) ne sont pas journalisées comme le ferait `bash` — la preuve d'impact repose alors sur les journaux système plutôt que sur l'historique shell

## Structure du repo

```
tp2-forensic-dfir/
├── README.md
├── GUIDE_DETAILLE.md
├── timeline.md
└── screenshots/
    ├── 01 à 09 (guide détaillé)
    └── timeline/ (preuves de la chronologie)
```

⚠️ L'image disque (`.raw`) et le fichier de hash ne sont pas versionnés ici (trop volumineux) — seules les captures d'écran et les commandes documentées le sont.
