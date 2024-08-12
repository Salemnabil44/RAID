Pour ajouter trois disques virtuels à votre machine virtuelle Ubuntu sous VirtualBox afin de créer un RAID 1, vous pouvez suivre les étapes suivantes :

### Étape 1 : Créer les disques virtuels dans VirtualBox
1. **Ouvrez VirtualBox** et sélectionnez votre machine virtuelle Ubuntu.
2. Cliquez sur le bouton **"Configuration"** (Settings).
3. Allez dans l'onglet **"Stockage"** (Storage).
4. Sous le contrôleur SATA (ou IDE), cliquez sur l'icône **"Ajouter un disque dur"** (Add Hard Disk) qui ressemble à un disque avec un signe `+`.
5. Choisissez **"Créer un nouveau disque dur"**.
6. Sélectionnez le type de fichier (par défaut `VDI`) et l'option de stockage dynamique (sauf si vous voulez un espace fixe).
7. Définissez la taille du disque (par exemple, 10 Go) et nommez-le (par exemple `Disk1`).
8. Répétez les étapes 4 à 7 pour créer les deux autres disques (`Disk2`, `Disk3`).
9. Une fois les trois disques créés, assurez-vous qu'ils sont tous connectés au contrôleur SATA.

### Étape 2 : Initialiser les disques dans Ubuntu
1. **Démarrez votre machine virtuelle Ubuntu**.
2. Ouvrez un terminal.
3. Utilisez la commande `lsblk` pour vérifier que les nouveaux disques sont détectés (`/dev/sdb`, `/dev/sdc`, `/dev/sdd` par exemple).

### Étape 3 : Configurer le RAID 1 avec `mdadm`
1. **Installer `mdadm`** si ce n'est pas déjà fait :
   ```bash
   sudo apt-get update
   sudo apt-get install mdadm
   ```
2. **Créer le RAID 1** avec deux disques (`/dev/sdb` et `/dev/sdc`) et utiliser le troisième comme disque de spare :
   ```bash
   sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc --spare-devices=1 /dev/sdd
   ```
   - `/dev/md0` est le nom du dispositif RAID.
   - `--level=1` spécifie le type de RAID (RAID 1).
   - `--raid-devices=2` signifie que le RAID 1 est configuré avec 2 disques actifs.
   - `--spare-devices=1` ajoute un disque de spare pour remplacer automatiquement un disque défaillant.

3. **Vérifier la configuration** :
   ```bash
   cat /proc/mdstat
   ```
   Cette commande montre l'état actuel de vos RAID, vous devriez voir votre `md0` en construction.

4. **Créer un système de fichiers** sur le RAID :
   ```bash
   sudo mkfs.ext4 /dev/md0
   ```

5. **Monter le RAID** dans un répertoire :
   ```bash
   sudo mkdir -p /mnt/raid1
   sudo mount /dev/md0 /mnt/raid1
   ```

6. **Vérifiez le montage** :
   ```bash
   df -h
   ```

### Étape 4 : Configurer le montage automatique au démarrage
1. **Obtenez l'UUID** du RAID :
   ```bash
   sudo blkid /dev/md0
   ```
2. **Modifier le fichier `/etc/fstab`** pour ajouter la ligne suivante :
   ```bash
   UUID=<votre-UUID> /mnt/raid1 ext4 defaults 0 0
   ```
3. **Tester le montage** sans redémarrer :
   ```bash
   sudo mount -a
   ```

Votre RAID 1 est maintenant configuré avec deux disques actifs et un disque de spare. Si un des disques actifs échoue, le disque de spare sera utilisé automatiquement pour remplacer le disque défaillant.

Voici les étapes pour créer une partition primaire qui occupe tout le disque et pour modifier son type en RAID Linux en utilisant `fdisk` sur le disque `/dev/sdb` :

### Étape 1 : Lancer `fdisk` sur le disque cible
Ouvrez un terminal sur votre machine Ubuntu et exécutez la commande suivante pour lancer `fdisk` sur le disque `/dev/sdb` :

```bash
sudo fdisk /dev/sdb
```

### Étape 2 : Créer une nouvelle partition
1. **Créer une nouvelle partition** :
   - Tapez `n` pour créer une nouvelle partition.
   - Choisissez `p` pour spécifier que vous voulez créer une partition primaire.
   - Tapez `1` pour spécifier que c'est la première partition.
   - Vous pouvez appuyer sur `Entrée` pour accepter les valeurs par défaut pour le premier et le dernier secteur, ce qui créera une partition qui occupe tout l'espace disponible sur le disque.

2. **Vérifier la création** :
   - Tapez `p` pour afficher la table des partitions et vérifier que la partition a bien été créée.

### Étape 3 : Modifier le type de la partition en RAID Linux
1. **Changer le type de partition** :
   - Tapez `t` pour changer le type de la partition.
   - Si vous avez créé la partition `1`, tapez `1` pour sélectionner cette partition (vous pouvez appuyer sur Entrée directement si c'est la seule partition).
   - Ensuite, tapez `fd` pour choisir le type "RAID Linux".

2. **Vérifier la modification** :
   - Tapez `p` encore une fois pour vérifier que le type de la partition est maintenant défini sur `fd` (Linux RAID).

### Étape 4 : Sauvegarder et quitter
1. **Sauvegarder les modifications** :
   - Tapez `w` pour écrire les modifications sur le disque et quitter `fdisk`.

### Étape 5 : Vérifier la partition
Une fois que vous avez quitté `fdisk`, vous pouvez vérifier la partition en utilisant la commande suivante :

```bash
lsblk -f
```

Cette commande affiche les informations sur les partitions, y compris le type, et vous devriez voir `/dev/sdb1` avec le type `linux_raid_member`.

Vous avez maintenant une partition primaire sur `/dev/sdb` qui prend tout le disque et est configurée pour être utilisée dans un RAID Linux. Effectuez la même pour les autres disques.

Pour créer, formater et monter un RAID 1 sous Ubuntu, vous pouvez suivre les étapes suivantes après avoir configuré vos disques comme précédemment mentionné.

### Étape 1 : Créer le RAID 1

1. **Assurez-vous que `mdadm` est installé** :
   ```bash
   sudo apt-get update
   sudo apt-get install mdadm
   ```

2. **Créer le RAID 1** :
   - Supposons que vous ayez deux partitions prêtes pour le RAID sur `/dev/sdb1` et `/dev/sdc1`.
   - Exécutez la commande suivante pour créer un RAID 1 :
     ```bash
     sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
     ```
   - `--create /dev/md0` crée un nouveau périphérique RAID nommé `/dev/md0`.
   - `--level=1` spécifie que c'est un RAID 1.
   - `--raid-devices=2` indique qu'il y a 2 périphériques dans ce RAID (vous pouvez adapter le nombre si vous avez plus de disques).

3. **Vérifiez le statut du RAID** :
   ```bash
   cat /proc/mdstat
   ```
   Cette commande affiche l'état du RAID. Vous devriez voir le RAID en cours de synchronisation.

### Étape 2 : Formater le RAID

1. **Créer un système de fichiers** sur le RAID :
   - Une fois que le RAID est prêt (la synchronisation peut prendre du temps), formatez-le avec `ext4` ou un autre système de fichiers de votre choix :
     ```bash
     sudo mkfs.ext4 /dev/md0
     ```
   - Remplacez `ext4` par `xfs`, `btrfs`, ou un autre système de fichiers si vous préférez.

### Étape 3 : Monter le RAID

1. **Créer un point de montage** :
   - Créez un répertoire où vous souhaitez monter le RAID :
     ```bash
     sudo mkdir -p /mnt/raid1
     ```

2. **Monter le RAID** :
   - Montez le RAID sur le point de montage que vous avez créé :
     ```bash
     sudo mount /dev/md0 /mnt/raid1
     ```

3. **Vérifiez que le RAID est monté** :
   - Utilisez la commande suivante pour vérifier que le RAID est bien monté :
     ```bash
     df -h
     ```

### Étape 4 : Configuration du montage automatique au démarrage

1. **Obtenez l'UUID** du RAID :
   - Pour vous assurer que le RAID est monté automatiquement après un redémarrage, vous devez ajouter son UUID dans `/etc/fstab` :
     ```bash
     sudo blkid /dev/md0
     ```
   - Notez l'UUID affiché.

2. **Modifier `/etc/fstab`** :
   - Éditez le fichier `/etc/fstab` pour y ajouter le montage automatique :
     ```bash
     sudo nano /etc/fstab
     ```
   - Ajoutez la ligne suivante en remplaçant `<UUID>` par l'UUID obtenu :
     ```bash
     UUID=<UUID> /mnt/raid1 ext4 defaults 0 0
     ```
   - Sauvegardez et quittez l'éditeur (`Ctrl+X`, `Y`, puis `Enter`).

3. **Tester le montage automatique** :
   - Testez la configuration sans redémarrer en exécutant :
     ```bash
     sudo mount -a
     ```
   - Vérifiez que tout est monté correctement avec `df -h`.

### Conclusion
Vous avez maintenant configuré un RAID 1, formaté avec `ext4`, et monté automatiquement sur `/mnt/raid1`. Votre RAID sera prêt à être utilisé à chaque démarrage de votre système.

