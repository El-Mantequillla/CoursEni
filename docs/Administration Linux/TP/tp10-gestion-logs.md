# TP10 : Gestion des logs

!!! abstract "En bref"
    **Objectifs** : rendre les logs durables et limiter leur taille (journald), rediriger les logs d'un service vers un fichier dédié (rsyslog), générer et retrouver des logs, puis analyser un redémarrage brutal.
    **Machine** : `srv-gui`.
    **Durée** : 45 min à 1 h.

```mermaid
flowchart LR
    A["I. journald<br/>durable, limites de taille"] --> B["II. rsyslog<br/>fichier dédié pour cron"]
    B --> C["III. Recherche<br/>sessions, arrêt forcé, analyse"]
```

---

## I. Configuration de Journald

### Rendre les logs durables

Par défaut, journald garde ses logs en **RAM** (`/run/log/journal`), perdus à chaque redémarrage. Pour les rendre persistants, il suffit de créer le dossier que journald utilise en priorité quand il existe :

```bash
# mkdir -p /var/log/journal
# systemd-tmpfiles --create --prefix /var/log/journal
```

`systemd-tmpfiles --create` applique immédiatement les bonnes permissions et le bon propriétaire à ce dossier (habituellement `root:systemd-journal`), sans attendre le prochain redémarrage.

```bash
# systemctl restart systemd-journald
```

### Limiter la taille

Les limites se règlent dans `/etc/systemd/journald.conf` :

```bash
# vi /etc/systemd/journald.conf
```

```ini
SystemMaxFileSize=20M
SystemMaxUse=150M
```

| Paramètre | Rôle |
|---|---|
| `SystemMaxFileSize` | Taille maximale d'**un seul** fichier de log journald |
| `SystemMaxUse` | Taille maximale **cumulée** de tous les fichiers de logs journald sur le disque |

!!! note "Pourquoi deux limites différentes"
    Journald découpe ses logs en plusieurs fichiers au fil du temps (rotation automatique). `SystemMaxFileSize` limite chaque fichier individuellement ; `SystemMaxUse` plafonne le total, en supprimant les plus anciens fichiers une fois la limite atteinte.

Applique le changement :

```bash
# systemctl restart systemd-journald
```

Vérifie :

```bash
$ journalctl --disk-usage
```

---

## II. Configuration de rsyslog

### Rediriger les logs de `cron` à partir du niveau « warning »

```bash
# vi /etc/rsyslog.d/cron-warn.conf
```

```text
cron.warning   /var/log/cron_warn.log
```

| Partie | Rôle |
|---|---|
| `cron` | La **facility** (source) : les messages du service cron |
| `warning` | Le **seuil minimal** de gravité : capture `warning` et tout ce qui est plus grave (`err`, `crit`, `alert`, `emerg`), mais pas `notice`/`info`/`debug` |
| `/var/log/cron_warn.log` | Le fichier où écrire ces messages |

!!! note "Créer un fichier dans `/etc/rsyslog.d/` plutôt que modifier `/etc/rsyslog.conf` directement"
    Tout fichier `.conf` placé dans `/etc/rsyslog.d/` est automatiquement inclus par rsyslog au démarrage. C'est plus propre que de surcharger le fichier principal, et plus facile à retirer proprement si besoin (il suffit de supprimer ce fichier).

Applique :

```bash
# systemctl restart rsyslog
```

### Générer un message de test avec `logger`

```bash
# logger -p cron.err "Test de log manuel du service cron"
```

| Option | Rôle |
|---|---|
| `-p cron.err` | Facility `cron`, priorité `err` (c'est le nom technique de ce que l'énoncé appelle « error ») |
| `"Test de log manuel du service cron"` | Le message lui-même |

!!! warning "Piège : « error » ne s'écrit pas `error` dans la syntaxe syslog"
    Les priorités syslog standards sont : `emerg`, `alert`, `crit`, `err`, `warning`, `notice`, `info`, `debug`. Le mot utilisé est **`err`**, pas `error`. `logger -p cron.error ...` échouerait ou serait mal interprété selon les versions.

### Vérifier où le message a été capturé

```bash
$ cat /var/log/cron_warn.log
$ cat /var/log/cron
$ journalctl -t root -g "Test de log manuel"
```

**Réponse à « y en a-t-il plusieurs ? » : oui, probablement.**

| Fichier | Pourquoi le message y apparaît |
|---|---|
| `/var/log/cron_warn.log` | La règle que tu viens de créer (`cron.warning`) : `err` est bien ≥ `warning` |
| `/var/log/cron` | La règle **par défaut** d'Oracle Linux (`cron.* /var/log/cron`, déjà présente dans `/etc/rsyslog.conf`) capture **tous** les niveaux de la facility `cron`, y compris `err` |
| journald | journald capture **systématiquement** tout, quelle que soit la configuration de rsyslog — les deux systèmes sont indépendants |

C'est une bonne illustration du fonctionnement de rsyslog : **plusieurs règles peuvent s'appliquer au même message** si leurs critères (facility + seuil de priorité) sont tous les deux remplis, sans qu'elles s'excluent mutuellement.

---

## III. Recherche de logs

### Lister les ouvertures de session depuis le début de la semaine

```bash
# mkdir -p /adm
$ last -s "$(date -d monday +%F)" > /adm/sessions.txt
```

| Élément | Rôle |
|---|---|
| `last` | Lit l'historique des connexions (fichier `/var/log/wtmp`) |
| `-s <date>` | Ne montre que les connexions **depuis** cette date |
| `$(date -d monday +%F)` | Calcule automatiquement la date du lundi de la semaine en cours, au format `AAAA-MM-JJ` |

!!! warning "Piège : vérifie ce que `date -d monday` renvoie vraiment"
    Le comportement de `date -d monday` dépend du jour où tu l'exécutes : il peut renvoyer **aujourd'hui** si on est déjà lundi, mais sur certains systèmes une expression ambiguë peut aussi pointer vers le lundi suivant plutôt que celui de la semaine en cours. **Teste d'abord isolément** :
    ```bash
    $ date -d monday +%F
    ```
    Si la date affichée n'est pas celle du lundi de cette semaine, utilise une date explicite à la place, par exemple `last -s 2026-09-28 > /adm/sessions.txt`.

Vérifie le résultat :

```bash
$ cat /adm/sessions.txt
```

!!! tip "Alternative avec journalctl, si tu veux chercher dans les logs plutôt que dans `/var/log/wtmp`"
    ```bash
    $ journalctl --since monday -g "session opened" > /adm/sessions.txt
    ```
    `-g` (*grep*) filtre les lignes contenant ce texte, typique des connexions SSH ou locales journalisées par PAM.

### Passer les deux commandes demandées (arrêt brutal du système)

```bash
# pkill -9 crond
```

Envoie le signal **`SIGKILL`** (numéro 9) au processus `crond` : il est tué **immédiatement**, sans qu'il ait l'occasion de se fermer proprement (contrairement à `systemctl stop crond`, qui lui laisse le temps de terminer ce qu'il fait). Ça simule un crash de service.

```bash
# systemctl --force --force poweroff
```

Éteint la machine **instantanément**, en ignorant toute procédure normale d'arrêt (pas d'attente des autres services, pas de démontage propre des systèmes de fichiers). Répéter `--force` deux fois le rend encore plus radical : même les éventuels blocages (*inhibitors*) qui empêcheraient normalement l'extinction sont ignorés. C'est l'équivalent logiciel de débrancher la prise.

!!! note "Pourquoi faire ça volontairement"
    Ces deux commandes servent à **provoquer** un arrêt non propre, pour ensuite vérifier deux choses : que les logs générés avant l'extinction ont bien survécu grâce à la persistance configurée en partie I, et que le noyau remonte bien des avertissements au redémarrage suivant (démarrage après un arrêt brutal plutôt que normal).

Rallume la VM après l'extinction, puis reconnecte-toi.

### Afficher les messages du service cron de gravité « avertissement » et au-delà

```bash
$ journalctl -u crond -p warning
```

`-p warning` (*priority*) filtre pour ne garder que les messages de gravité `warning` **ou plus grave** (`err`, `crit`, `alert`, `emerg`) — c'est le comportement par défaut de `journalctl -p` : une seule valeur donnée inclut automatiquement tout ce qui est plus sévère.

### Le noyau a-t-il remonté des avertissements/erreurs au dernier redémarrage ?

```bash
$ journalctl -k -p warning -b 0
```

| Option | Rôle |
|---|---|
| `-k` | *kernel* : uniquement les messages du noyau (équivalent moderne de `dmesg`) |
| `-p warning` | Seuil de gravité, comme au-dessus |
| `-b 0` | *boot* : uniquement le démarrage **actuel** (`-b -1` donnerait le précédent, si les logs sont assez anciens et persistants) |

Si cette commande affiche des lignes, c'est oui ; si elle ne renvoie rien, le noyau n'a rien signalé d'anormal à ce démarrage.

### Afficher les messages cron apparus seulement entre mardi et jeudi

```bash
$ journalctl -u crond --since tuesday --until thursday
```

!!! warning "Piège : vérifie les bornes de dates, comme pour `last -s`"
    `--since tuesday` et `--until thursday` sont calculés par rapport à **aujourd'hui**, avec les mêmes subtilités que `date -d monday` vues plus haut. Si le résultat semble incohérent avec tes attentes, remplace par des dates explicites :
    ```bash
    $ journalctl -u crond --since "2026-09-29 00:00:00" --until "2026-10-02 00:00:00"
    ```
    Note aussi que `--until thursday` s'arrête **au tout début** de jeudi (minuit) : pour inclure la journée complète de jeudi, utilise `--until friday` à la place.

    « Si c'est possible » dans l'énoncé suggère qu'il peut ne pas y avoir de messages cron dans cette plage précise, selon l'activité réelle du service sur ta VM à ce moment-là — un résultat vide n'est pas forcément une erreur de ta part.

---

## À retenir

- **`SystemMaxFileSize`** (par fichier) et **`SystemMaxUse`** (au total) se règlent dans `/etc/systemd/journald.conf`, appliqués après `systemctl restart systemd-journald`.
- Créer `/var/log/journal` + `systemd-tmpfiles --create` rend journald **persistant** entre les redémarrages.
- Une règle rsyslog `facility.priorité fichier` capture ce niveau **et tout ce qui est plus grave**.
- Les priorités syslog valides : `emerg alert crit err warning notice info debug` — « error » se dit **`err`**.
- Plusieurs règles rsyslog peuvent capturer le **même** message si leurs critères se chevauchent, sans s'exclure.
- `journalctl -p <niveau>` inclut automatiquement tout ce qui est plus sévère que ce niveau.
- `pkill -9` = arrêt brutal immédiat ; `systemctl --force --force poweroff` = extinction instantanée sans procédure normale.
- Toujours **vérifier isolément** le résultat d'une expression de date relative (`date -d monday`, `--since tuesday`) avant de t'y fier dans une commande plus longue.
