# TP 3 : We do a little scripting

```
#!/bin/bash
# 03/01/23 - logards
# idcard de la machine 

echo "Machine Name : $(hostnamectl | grep hostname| cut -d ' ' -f4)."
echo "OS $(cat /etc/os-release | grep 'NAME=' -m1 | cut -d '"' -f2) and kernel version is $(uname -r)"
echo "IP : $(ip a | grep enp0s8 | grep inet | tr -s ' ' | cut -d '/' -f 1 | cut -d ' ' -f 3 )"
echo "RAM : $(free -t -h --mega | grep 'Mem' | tr -s ' ' | cut -d ' ' -f 3)  memory available on $(free -t -h --mega | grep 'Mem' | tr -s ' ' | cut -d ' ' -f 2) total memory"
echo "DISK : $(df -h | grep root | tr -s ' ' | cut -d ' ' -f 4 ) space left"
echo "Top 5 processes by RAM usage :"
for i in $(seq 3 7)
do 
        echo "-$(ps -e -o cmd=--sort=-%mem | head -n ${i} | tail -n +${i} )"
done

echo "Listening ports :"
while read line 
do
        nb="$(echo $line | tr -s ' ' | cut -d ':' -f2 | cut -d ' ' -f1)"
        protocol="$(echo $line | tr -s ' ' | cut -d ' ' -f1) "
        type="$(echo $line | cut -d '"' -f2) "
        echo "- $nb $protocol: $type " 
done <<< "$(ss -4altunpH)"

curl https://cataas.com/cat --output cat 2>/dev/null 
echo "Here is your random cat : $(find cat)"
```

```
[logards@sakaido idcard]$ sudo ./idcard.sh 
Machine Name : sakaido.
OS Rocky Linux and kernel version is 5.14.0-70.26.1.el9_0.x86_64
IP : 10.5.1.3
RAM : 171M  memory available on 960M total memory
DISK : 5.1G space left
Top 5 processes by RAM usage :
-[kthreadd]
-[rcu_gp]
-[rcu_par_gp]
-[kworker/0:0H-events_highpri]
-[mm_percpu_wq]
Listening ports :
- 323 udp : chronyd  
- 22 tcp : sshd  
Here is your random cat : cat
```
# II. Script youtube-dl


## Rendu

```
#!/bin/bash

url=$1
web_title=$(youtube-dl -e ${1})
directory='/srv/yt/downloads'
cmp_directory="${directory}/${web_title}/${web_title}.mp4"
log_yt='/var/log/yt/download.log'
description=$(youtube-dl --get-description ${1})
if [ ! -f ${log_yt} ] 
then
    echo "Error: Directory ${log_yt}  does not exists."
    exit 1
fi
youtube-dl ${url} --output "${directory}/${web_title}/${web_title}.mp4" > /dev/null
echo "${description}" > "${directory}/${web_title}/description.txt"
echo "Video ${url} was downloaded"
echo "File path ${cmp_directory}"
echo "[$(date '+%D %T')] Video ${url} was downloaded. File path : ${cmp_directory}" >> ${log_yt}

```

```
[logards@sakaido idcard]$ cat /var/log/yt/download.log
[01/14/23 15:24:18] Video https://www.youtube.com/watch?v=jhFDyDgMVUI was downloaded. File path : /srv/yt/downloads/One Second Video/One Second Video.mp4
[01/14/23 15:24:28] Video https://www.youtube.com/watch?v=jhFDyDgMVUI was downloaded. File path : /srv/yt/downloads/One Second Video/One Second Video.mp4
[01/14/23 15:25:27] Video https://www.youtube.com/watch?v=jhFDyDgMVUI was downloaded. File path : /srv/yt/downloads/One Second Video/One Second Video.mp4
[01/14/23 15:29:48] Video https://www.youtube.com/watch?v=jhFDyDgMVUI was downloaded. File path : /srv/yt/downloads/One Second Video/One Second Video.mp4
[01/14/23 15:30:40] Video https://www.youtube.com/watch?v=Wch3gJG2GJ4 was downloaded. File path : /srv/yt/downloads/1 Second Video/1 Second Video.mp4
```

```
[logards@sakaido yt]$ bash yt.sh https://www.youtube.com/watch?v=jhFDyDgMVUI
Video https://www.youtube.com/watch?v=jhFDyDgMVUI was downloaded
File path /srv/yt/downloads/One Second Video/One Second Video.mp4
```

# III. MAKE IT A SERVICE

## Rendu

📁 **Le script `/srv/yt/yt-v2.sh`**

```
#!/bin/bash

url_file='/srv/yt/url.txt'
directory='/srv/yt/downloads'
log_yt='/var/log/yt/download.log'
if [[ ! -f ${log_yt} ]] 
then
  echo "Error: Directory ${log_yt}  does not exists."
  exit 1
fi
while :
do
  if [[ -s ${url_file} ]] ; then
    while IFS= read -r line ; do
      web_title=$(youtube-dl -e ${line} 2> /dev/null)
      description=$(youtube-dl --get-description ${line} 2> /dev/null)
      cmp_directory="${directory}/${web_title}/${web_title}.mp4"
      youtube-dl ${line} --output "${directory}/${web_title}/${web_title}.mp4" &>> /dev/null
      echo "${description}" > "${directory}/${web_title}/description.txt"
      echo "Video ${line} was downloaded"
      echo "File path ${cmp_directory}"
      echo "[$(date '+%D %T')] Video ${url} was downloaded. File path : ${cmp_directory}" >> ${log_yt}
      sed -i '1d' "${url_file}" > /dev/null
      sleep 5
    done < "${url_file}"
  else 
    echo "Erreur le url_file est vide" > /dev/null
  fi
done
```

📁 **Fichier `/etc/systemd/system/yt.service`**

```
[Unit]
Description=Dl une video youtube a partir d'un fichier contenant les url

[Service]
ExecStart=/srv/yt/yt-v2.sh
User=yt

[Install]
WantedBy=multi-user.target
```
```
[logards@sakaido yt]$ sudo systemctl status yt
● yt.service - Dl une video youtube a partir d'un fichier contenant les url
     Loaded: loaded (/etc/systemd/system/yt.service; enabled; vendor preset: disabled)
     Active: active (running) since Sat 2023-01-14 15:27:28 CET; 27min ago
   Main PID: 658 (yt-v2.sh)
      Tasks: 1 (limit: 5905)
     Memory: 1008.0K
        CPU: 26min 53.456s
     CGroup: /system.slice/yt.service
             └─658 /bin/bash /srv/yt/yt-v2.sh

Jan 14 15:27:28 sakaido systemd[1]: Started Dl une video youtube a partir d'un fichier contenant les url.
```

```
[logards@sakaido yt]$ journalctl -xe -u yt
Jan 14 15:27:28 sakaido systemd[1]: Started Dl une video youtube a partir d'un fichier contenant les url.
░░ Subject: A start job for unit yt.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ A start job for unit yt.service has finished successfully.
░░ 
░░ The job identifier is 240.
Jan 14 16:00:12 sakaido yt-v2.sh[658]: Video https://www.youtube.com/watch?v=Wch3gJG2GJ4&t=1s&ab_channel=Zetzu500 was downloaded
Jan 14 16:00:12 sakaido yt-v2.sh[658]: File path /srv/yt/downloads/1 Second Video/1 Second Video.mp4
Jan 14 16:00:23 sakaido yt-v2.sh[658]: Video https://www.youtube.com/watch?v=jhFDyDgMVUI&ab_channel=WaitOneSecondHere was downloaded
Jan 14 16:00:23 sakaido yt-v2.sh[658]: File path /srv/yt/downloads/One Second Video/One Second Video.mp4
```


# IV. Bonus

Quelques bonus pour améliorer le fonctionnement de votre script :

➜ **en accord avec les règles de [ShellCheck](https://www.shellcheck.net/)**

- bonnes pratiques, sécurité, lisibilité

➜  **fonction `usage`**

- le script comporte une fonction `usage`
- c'est la fonction qui est appelée lorsque l'on appelle le script avec une erreur de syntaxe
- ou lorsqu'on appelle le `-h` du script

➜ **votre script a une gestion d'options :**

- `-q` pour préciser la qualité des vidéos téléchargées (on peut choisir avec `youtube-dl`)
- `-o` pour préciser un dossier autre que `/srv/yt/`
- `-h` affiche l'usage

➜ **si votre script utilise des commandes non-présentes à l'installation** (`youtube-dl`, `jq` éventuellement, etc.)

- vous devez TESTER leur présence et refuser l'exécution du script

➜  **si votre script a besoin de l'existence d'un dossier ou d'un utilisateur**

- vous devez tester leur présence, sinon refuser l'exécution du script

➜ **pour le téléchargement des vidéos**

- vérifiez à l'aide d'une expression régulière que les strings saisies dans le fichier sont bien des URLs de vidéos Youtube