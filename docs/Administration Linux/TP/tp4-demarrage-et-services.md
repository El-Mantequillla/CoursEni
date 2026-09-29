# TP4 : Gestion du démarrage et des services

!!! abstract "En bref"
    **Objectifs** : mesurer le temps de démarrage, changer la cible de démarrage par défaut, et gérer des services avec `systemctl` (`sshd` et `crond`).
    **Prérequis** : avoir terminé le TP3.
    **Machine** : la VM graphique (`srv-gui` dans l'énoncé = ton `srvclient`).
    **Durée** : 30 à 40 min.

Ce TP a **deux parties** :

```mermaid
flowchart LR
    A["1. Configuration du système<br/>temps de démarrage<br/>cible par défaut"] --> B["2. Gestion des services<br/>sshd : retrouver ses éléments<br/>crond : désactiver / réactiver"]
```

---

## Partie 1 : configuration du système

### Question 1 : en combien de temps le système a-t-il démarré ?

```bash
# systemd-analyze
```

Exemple de résultat :

```text
Startup finished in 1.2s (kernel) + 2.5s (initrd) + 18.3s (userspace) = 22.0s
graphical.target reached after 17.9s in userspace
```

| Élément | Signification |
|---|---|
| `kernel` | Temps du **noyau** pour s'initialiser |
| `initrd` | Temps de l'**initramfs** (mini-système temporaire) |
| `userspace` | Temps de **systemd** et des services |
| Total | Somme des trois |

Pour savoir **quels services** ralentissent le plus le démarrage :

```bash
# systemd-analyze blame
```

!!! tip "Note ce temps **avant** de continuer"
    Tu vas changer la cible de démarrage : garde la valeur pour la **comparer** après.

!!! note "Ce que le chiffre ne compte pas"
    `systemd-analyze` ne compte ni l'**arrêt** du système, ni les secondes d'attente sur le **menu GRUB**. Pour mesurer le redémarrage complet, utilise un chronomètre (et retire les secondes de GRUB).

### Question 2 : démarrer par défaut en `multi-user.target`

D'abord, regarde la cible actuelle :

```bash
# systemctl get-default
graphical.target
```

Puis change-la :

```bash
# systemctl set-default multi-user.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target → /usr/lib/systemd/system/multi-user.target.
```

Vérifie :

```bash
# systemctl get-default
multi-user.target
```

Redémarre pour valider :

```bash
# reboot
```

**Résultat attendu** : un écran de connexion **en mode texte**, sans le bureau GNOME. Relance ensuite `systemd-analyze` et compare : le démarrage est en général un peu plus rapide, car l'interface graphique n'est plus lancée.

```mermaid
flowchart LR
    subgraph AVANT
    A["graphical.target<br/>bureau GNOME"]
    end
    subgraph APRES
    B["multi-user.target<br/>console seule"]
    end
    A -->|"systemctl set-default multi-user.target<br/>+ redémarrage"| B
```

!!! warning "Piège : `set-default` ≠ `isolate`"
    - `systemctl set-default multi-user.target` : change la cible **à chaque démarrage** (modifie le lien `/etc/systemd/system/default.target`). **C'est ce que demande l'énoncé.**
    - `systemctl isolate multi-user.target` : change la cible **maintenant**, mais on revient à l'ancienne au prochain redémarrage.

!!! tip "Revenir au bureau"
    `systemctl set-default graphical.target` remet le mode graphique par défaut. Pour y passer tout de suite : `systemctl isolate graphical.target`.

---

## Partie 2 : gestion des services

### Le service SSH (`sshd`) : retrouver ses éléments

SSH permet la **connexion sécurisée à distance**. Sous Oracle Linux, le service s'appelle **`sshd`** (et non `ssh` comme sous Debian).

| À trouver | Commande | Résultat |
|---|---|---|
| **Le fichier de configuration systemd du service** | `systemctl status sshd` (ligne `Loaded:`) ou `systemctl show -p FragmentPath sshd` | `/usr/lib/systemd/system/sshd.service` |
| **Le nom du binaire exécuté** | `systemctl cat sshd` puis la ligne `ExecStart=` | `/usr/sbin/sshd` |
| **Le fichier de configuration du démon** | `rpm -qc openssh-server` ou `man sshd` (section FILES) | `/etc/ssh/sshd_config` |

Exemple :

```bash
# systemctl show -p FragmentPath sshd
FragmentPath=/usr/lib/systemd/system/sshd.service

# grep ExecStart /usr/lib/systemd/system/sshd.service
ExecStart=/usr/sbin/sshd -D $OPTIONS

# rpm -qc openssh-server
/etc/ssh/sshd_config
/etc/sysconfig/sshd
...
```

!!! warning "Ne pas confondre"
    - **`/etc/ssh/sshd_config`** configure le **serveur** SSH (le démon).
    - `/etc/ssh/ssh_config` configure le **client** `ssh`.
    - `/etc/sysconfig/sshd` ne contient que des options de lancement (`$OPTIONS`).

!!! info "Et si on veut modifier le fichier systemd ?"
    Le fichier d'origine dans `/usr/lib/systemd/system` **ne se modifie pas**. Un fichier de **même nom** dans `/etc/systemd/system` est **prioritaire** : on copie le fichier là-bas (ou `systemctl edit --full sshd`), on le modifie, puis `systemctl daemon-reload`. Par défaut, ce fichier prioritaire n'existe pas.

### Le service cron (`crond`) : désactiver puis restaurer

`cron` est le **planificateur de tâches**. Sous Oracle Linux, il s'appelle **`crond`**.

#### Étape 1 : est-il lancé automatiquement au démarrage ?

```bash
# systemctl is-enabled crond
enabled

# systemctl status crond
```

`enabled` signifie qu'il démarre avec le système. Dans `status`, la ligne `Active: active (running)` indique qu'il tourne en ce moment.

#### Étape 2 : désactiver son démarrage automatique

```bash
# systemctl disable crond
Removed /etc/systemd/system/multi-user.target.wants/crond.service.

# systemctl is-enabled crond
disabled
```

!!! warning "Piège : `disable` n'arrête pas le service"
    `disable` supprime seulement le lien qui le lançait **au démarrage**. Le service **continue de tourner** jusqu'au redémarrage. Pour l'arrêter maintenant : `systemctl stop crond`.

!!! note "« Totalement » : `disable` ou `mask` ?"
    `disable` empêche le lancement au boot, mais on peut encore le démarrer à la main ou via une dépendance. `systemctl mask crond` le bloque **complètement** (lien vers `/dev/null`). Le cours ne présente que `enable`/`disable`, donc `disable` est sans doute la réponse attendue. Si tu utilises `mask`, il faudra `systemctl unmask crond` avant de le réactiver.

#### Étape 3 : redémarrer et valider

```bash
# reboot
```

Après la reconnexion :

```bash
# systemctl status crond
# systemctl is-enabled crond
# ps aux | grep crond
```

**Résultat attendu** : `inactive (dead)`, `disabled`, et `ps aux` ne montre que la ligne du `grep` lui-même. Cela prouve que `crond` n'a pas démarré tout seul.

#### Étape 4 : restaurer le comportement par défaut

```bash
# systemctl enable crond
Created symlink /etc/systemd/system/multi-user.target.wants/crond.service → /usr/lib/systemd/system/crond.service.

# systemctl start crond
```

(ou en une seule commande : `systemctl enable --now crond`)

Vérifie :

```bash
# systemctl is-enabled crond
enabled
# systemctl status crond
```

Tu dois voir `enabled` et `active (running)`.

```mermaid
flowchart LR
    A["enabled + active<br/>(état d'origine)"] -->|"systemctl disable crond<br/>+ reboot"| B["disabled + inactive"]
    B -->|"systemctl enable --now crond"| A
```

### Tableau récapitulatif de `systemctl`

| Je veux... | Commande |
|---|---|
| Voir l'état d'un service | `systemctl status <service>` |
| Le démarrer / l'arrêter **maintenant** | `systemctl start` / `systemctl stop <service>` |
| Le redémarrer | `systemctl restart <service>` |
| Le lancer **au démarrage** / ne plus le lancer | `systemctl enable` / `systemctl disable <service>` |
| Savoir s'il démarre au boot | `systemctl is-enabled <service>` |
| Faire `enable` + `start` en une fois | `systemctl enable --now <service>` |

!!! warning "Piège : `start` ≠ `enable`"
    - `start` / `stop` : agissent **maintenant**, sont oubliés au redémarrage.
    - `enable` / `disable` : agissent **au prochain démarrage**, sans démarrer ni arrêter le service tout de suite.

## À retenir

- `systemd-analyze` : temps de démarrage ; `systemd-analyze blame` : les services les plus lents.
- **`set-default`** modifie la cible au démarrage ; **`isolate`** change seulement l'état actuel.
- Les fichiers de service d'origine sont dans `/usr/lib/systemd/system` ; la copie dans `/etc/systemd/system` est **prioritaire**.
- Sous Oracle Linux : `sshd` (SSH) et `crond` (cron).
- **`enable`/`disable`** gèrent le démarrage automatique, **`start`/`stop`** gèrent l'état immédiat.
