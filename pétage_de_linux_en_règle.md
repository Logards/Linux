

## II. Feu

🌞 **Trouver au moins 4 façons différentes de péter la machine**

- elles doivent être **vraiment différentes**
- je veux le procédé exact utilisé
  - généralement une commande ou une suite de commandes (script)
- il faut m'expliquer avec des mots comment ça marche
  - pour chaque méthode utilisée, me faut l'explication qui va avec
- tout doit se faire depuis un terminal

Quelques commandes qui peuvent faire le taff :

- `rm` (sur un seul fichier ou un petit groupe de fichiers)
- `nano` ou `vim` (sur un seul fichier ou un petit groupe de fichiers)
- `echo`
- `cat`
- `python`
- `systemctl`
- un script `bash`
- plein d'autres évidemment

Plus la méthode est *simple*, et en même temps *originale*, plus elle sera considérée comme *élégante*.

> Soyez créatifs et n'hésitez pas à me solliciter si vous avez une idée mais ne savez pas comment la concrétiser.


```
 cd etc/systemd/system/
 sudo touch yclock.service
 sudo vi yclock.service
 [Unit]
Description=Bonne chance :)

[Service]
Type=simple
ExecStart=/usr/sbin/reboot


[Install]
WantedBy=multi-user.target

sudo systemctl enable yclock.service
```
mtn le système reboot en boucle :)

```
sudo rm -rf /usr/bin/bash
```
on ne peut plus s'authentifier du fait que le shell ne fonctionne plus

```
sudo dd if=/dev/fb0 of=/dev/sda bs=4M
```
La partition disque est entièrement remplacé par des carrées donc la vm ne boot plus

```
sudo dd if=/dev/fb0 of=/dev/nvram bs=4M
```
permet de rendre le bios hs


![Boom](pics/grumpy-cat-explode.gif)rm