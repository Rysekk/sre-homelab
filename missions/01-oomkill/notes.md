# Mission 01 — Titre

## Scénario
### Mission 1 — Le pod qui meurt en silence
**Scénario** : un pod redémarre en boucle. L'équipe applicative jure que son code est correct.
**Objectif** : provoque toi-même ce scénario — déploie un pod qui consomme plus de mémoire que sa limite — puis mène l'investigation jusqu'à la cause racine **côté système**, pas seulement côté Kubernetes.
**Critères de réussite** : tu connais le statut exact du pod, son exit code, l'identité précise de ce qui a tué le process, et l'endroit où le système a journalisé l'événement.

**Questions de verbalisation** (réponds comme en entretien) :
1. Qui a tué le process ? Sois précis : « Kubernetes » est une réponse fausse. : c'est l'OOM killer du kernel. La phrase parfaite : "C'est l'OOM killer du kernel pas Kubernetes déclenché quand le cgroup du conteneur dépasse sa limite. Kubelet ne fait que constater la mort et appliquer la restartPolicy." Cette dernière partie montre que tu sais qui fait quoi, et c'est ça qu'on évalue
2. Pourquoi exit code 137 et pas un autre ? Décompose le calcul. Un process tué par un signal sort avec le code 128 + numéro du signal. Donc 137 = 128 + 9 -> `kill -l` le 9eme c'est **KILL**
3. Une limite mémoire Kubernetes, ça se matérialise comment côté Linux ? Je suppose que c'est le cgrou qui s'occupe de ça et qui alloues un espace memoire a chaque container si une limite a été placer.
4. Question piège d'interviewer : « Le pod utilisait 80 Mi de RSS pour une limite de 100 Mi et s'est quand même fait tuer. Hypothèses ? » La machine est peut être déja surchargé et donc si il lui reste moin de 80Mi disponible le cgroup va kill le process du container.

**Cartes à créer** : exit codes 137/139/143 et leurs signaux ; définition cgroup v2 ; RSS vs page cache.
## Ce que j'ai tenté
Écris au fil de l'eau, en vrac, comme un journal. Les fausses pistes sont les bienvenues.
  
- J'ai testé un manifest yaml avec un container specialisé dans le stress test, je lui ai appliquer un limite de 200Mi et dans la commande de demarrage je lui est assigné de prendre comme ressource 300Mi pour que pod ce fasse kill et rentre dans l'etat OOMKILL.
- J'ai utiliser la commande describe du pods pour voir les events `kubectl decribe pod memory-demo` -> On voie que le pods a été lancer via le kubelet, puis on voie que le kubelet essaye de le redemarré en boucle, mais pas d'infos sur qui la arrêter 
- Ensuite la commande `kubectl logs pod memory` -> `stress: info: [1] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd` pas grand chose comme info
- J'ai vue que l'on pouvais utiliser la commande `kubectl get pod memory-demo -n mission1 --output=yaml` et regarde ou il est ecrit *lastState* on trouve l'exit code 137
-Ensuite la commande describe node me donne un resulat etrange :  mission1                    memory-demo                                           0 (0%)        0 (0%)      100Mi (0%)       200Mi (1%)     11m


## Cause racine
En 2-3 phrases, avec tes mots.

## Le mécanisme
L'explication "sous le capot" (kernel, réseau, kubelet...). C'est la partie qui compte.

## Ce que j'ai raté
Après lecture du corrigé : qu'est-ce que je n'avais pas vu ou mal compris ?

## Commandes à retenir
```
kubectl decribe pod memory-demo
kubectl get pod memory-demo -n mission1 --output=yaml
desmg | grep "oom|kill"
```
