# My Laptop Provision scripts

> Set of [Ansible](https://www.ansible.com/) playbooks to provision a Fedora machine with all my applications and settings.

## Motivation

Setup a new machine from scratch is always a very time consuming task. Last time I did it, it took an entire weekend to install everything.

And there is a high change of forget some less used command line tool. Also as the Developer I install many tools directly from GitHub and so checking if they are updated is a very manual process, like going to GitHub, check latest release, download the binary and move it to the correct place.

What if most of this setup, can be automated? That´s where [Ansible](https://ansible.readthedocs.io/en/latest/) and this reposiory comes in.

## Pre-requisites

* A machine running Fedora OS. This playbook was tested with Fedora 38
* Git (`dnf install -y git`)
* GitHub personal access token. You can get one [here](https://github.com/settings/tokens).

## What is included

- Initial system update.
- Install Dotfiles from my private dotfiles repository, using [Yet Another Dotfiles Manager - yadm](https://yadm.io/)
- Install all my applications. Check [APPLICATIONS.md](APPLICATIONS.md) for a detailed list.

## Provision a new machine

To provision a new machine open a terminal window and run the following commands:

```sh
export GITHUB_TOKEN=<my_github_token>
git clone https://github.com/brpaz/my-linux-setup
sudo chmod +x setup.sh
./setup.sh
```

**Note** When installing dotfiles you will be prompted for the "pgp" key to decrypt the secure files. Make sure to have it at hand.

---

## Post install steps

Unfortunately not everything can be automated and some manual steps will be required after running this scripts.

### Restoring $HOME directory from backup

The Ansible playbook syncs mostly of the dotfiles. Still, user data like Pictures, Music etc, must be restored manually from backup.

To do so, you can run the [scripts/restore_home.sh] script.

```bash
sudo chmod +x scripts/restore_home.sh
BACKUP_PATH=/path/to/home/backup scripts/restore_home.sh
```

This will rsync the most important folders like `Documents`, `Video`, `Music`, `Code` etc from the specified backup location.


### Enabling Restic backups

The setup should have installed systemd units to backup the system using Restic, both to Backblaze B2 and to external NAS. These units are disabled by default, for better control, since we don´t want to start backup, until we have restored our files.

To enable the restic backups, run the following commands on your terminal:

```bash
systemctl --user enable restic-nas-backup.timer
systemctl --user enable restic-nas-prune.timer
systemctl --user enable restic-b2-backup.timer
systemctl --user enable restic-b2-prune.timer
```

### Install Web Apps

I use [WebCatalog](https://webcatalog.io/webcatalog/) to allow easy installation of web applications.

I have the following:

- Feedly
- Instagram
- Google Keep
- Google Photos
- Fast
- TickTick
- ProtonMail

### Mount Home NAS

In order to be able to backup the machine and access the files stored on the home NAS, we should mount their folders using NFS.

For that, first make sure you have `nfs` installed on your machine.

Then we can try to mount the volumes manually, using:

```bash
sudo mkdir /mnt/qnap-media
sudo mkdir /mnt/qnap-backups
sudo mount -t nfs 192.168.1.108:/Multimedia /mnt/qnap-media
```

If the test is successfull, we can now instruct the system to mount the volumes automatically on boot, by adding the following lines to the `/etc/fstab` file:


```
192.168.1.108:/Multimedia /mnt/qnap-media  nfs      defaults,nofail,nobootwait,noauto,x-systemd.automount    0       0
192.168.1.108:/Backups /mnt/qnap-backups  nfs      defaults,nofail,nobootwait,noauto,x-systemd.automount    0       0
```

**Replace the IP Address and mount path accordingly.**

### Other tasks

* [] Execute [Jetbrains toolbox](https://www.jetbrains.com/toolbox-app/) and install the IDEs. (DataGrip, Goland, IDEA, PHPStorm, WebStorm, CLion, Android Studio).
* [] Open Chrome and Firefox browsers and login to start syncing all the extensions, bookmarks etc.
* [] Enable Restic Backups
* [] Login into applications (Gnome Accounts, Spotify, etc).
* [] Configure Pika Backup

---

## Keep the system updated

Most of the tasks are idenpotent and you can use this playbook to keep your system updated. This is useful, for example, to automatically update all the programs installed from GitHub, as the playbooks will try to fetch and install always the latest release.

You can execute a specific tag, by running:

```bash
 TAG=node task run-tag
```

## Keep GitHub installed software updated

```bash
TAG=github task run-tag
```

---

## Reference

* http://radeksprta.eu/automatically-setup-computer-ansible-playbook/
* http://blog.josephkahn.io/articles/ansible/
* https://github.com/Benoth/ansible-ubuntu

## Author

👤 **Bruno Paz**

* Website: [brunopaz.dev](https://brunopaz.dev)
* Github: [@brpaz](https://github.com/brpaz)

## 📝 License

Copyright © 2019 [Bruno Paz](https://github.com/brpaz).

This project is [MIT](https://opensource.org/licenses/MIT) licensed.

