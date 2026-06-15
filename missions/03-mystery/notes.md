# Mission 3 — Le serveur mystère

## Scénario
Un serveur "rame". Aucun APM, aucun dashboard, aucune métrique pré-installée. Tu as un shell et 5 minutes. Une charge mystère tourne sur un node tu ne sais pas si c'est du CPU, de la mémoire, de l'I/O disque ou du réseau. Identifie sa nature avec une méthode structurée, pas en tâtonnant au hasard.

## Ma prédiction (avant de manipuler)
> À remplir AVANT de lancer quoi que ce soit. Avec quelle commande comptes-tu commencer, et pourquoi celle-là en premier ?
La commande top qui permet d'avoir une vue d'enssemble sur la consommation cpu, ram, IO et permet rapidement d'identifier quelles sont les process les plus consomateur.



## Ce que j'ai tenté
Journal au fil de l'eau, fausses pistes incluses.

- Commande : `kubectl top node` → ce que j'ai lu : On voit les 3 noeud k3d et on remarque que le noeud k3d-homelab-server-0 a un CPU de 4101m et un CPU a 29% → ce que j'en déduis → le probléme viens de ce noeud.
- On va allé sur ce noeud pour comprendre le soucis → `sudo docker exec -it k3d-homelab-server-0 sh` pour acceder au noeud qui pose soucis, on fait la commande top puis on regarde les valeurs cpu, sys, idle, io,... on voie que le cpu est a 32% avec un idle de 66% et un load avarage a 1m de 4,57 ce qui est beaucoup. On a donc en moyenne sur 1m 4 process qui sont en attente et quand on regarde la conf sur container on vois viens les 4cpu.



## Commandes à retenir
```
kubectl top node
sudo docker exec -it k3d-homelab-agent-0 sh
top
```

## Questions de verbalisation
À traiter dans verbalisation.md, à voix haute sans notes :
1. Déroule tes 5 premières minutes sur un serveur lent inconnu, dans l'ordre, en justifiant chaque étape.
Voic les 5 premiére minute, si on observe des lanteur sur un serveur :
- Kubectl top node pour identifier le noeud qui pose soucis.
- commande top
- On regarde les valeurs CPU, usr, sys , nic, idle, io, irq, sirq (methode USE on fait par elemination)
- Cela nous permet d'identifier en un seul coup d'oeuil si le probléme viens du CPU, de l'IO.
- On regarde le load average vois si il n'y a pas des processus en attente
- Ensuite on regarde la RAM, pour voir si il n'y a pas un soucis.
- Normalement on a ce qui notre coupable au bout de 5 min.
2. Load average à 12 sur une machine 8 CPU mais CPU idle à 70% : explique.
Le load average compte les process runnable ou en attente ininterruptible (D-state), pas l'occupation CPU. Un load à 12 sur 8 cœurs avec 70% d'idle signifie que ~12 process veulent avancer mais que la plupart sont bloqués sur autre chose que le CPU, inon le CPU ne serait pas idle. Le suspect n°1 est l'I/O : je vérifie le wa (I/O wait) dans top et je confirme avec iostat (%util, await). Ça peut être de l'I/O disque directe, ou un manque de RAM qui force le swap (à vérifier avec les lignes Mem/Swap et les colonnes si/so de vmstat). Plus rarement, une attente réseau (NFS gelé). Ce n'est pas un problème CPU.
3. C'est quoi un process en D-state, et pourquoi c'est important pour interpréter le load ?
Le D state c'est un process en indistruptive sleep, qui est en attente de quelque chose d'un autre process, souvent un IO qui ne reviens pas (disk lent..)
4. Décris la méthode USE et applique-la au sous-système disque.
La methode USE, correspond a la Methode pour chaque ressource il faut check l'utilisation, la saturation et les Error. Dans le cas d'un sous system de disque ce serais de verifier l'utilisation des disk (I/O, commande top pour le wa et iostat, iotop), la saturation ça correspond a la place restance sur les files system (avec les commande df et du), et les erreurs ce sont les erreurs d'ecriture sur les filesystem (verifier dans le journal du kernel : journalctl ou desmg).