# Mission 02 —  Rapide mais lent

## Scénario
### Mission 2 —  Rapide mais lent
**Scénario** :  une application est lente par à-coups — des pics de latence qui vont et viennent. Tu regardes le dashboard du node qui l'héberge : le CPU est à 40%. Le développeur dit "c'est l'infra qui est sous-dimensionnée", l'équipe infra répond "le node est à moitié vide, c'est ton code". Les deux ont tort. Tranche.
**Objectif** : reproduire le phénomène. Déploie un pod qui veut consommer beaucoup plus de CPU que la limite que tu lui imposes (par exemple un process qui veut 4 cœurs avec une limite à 0,5). Puis trouve la métrique qui prouve ce qui se passe réellement.
**Critères de réussite**

**Questions de verbalisation** (réponds comme en entretien) :
1. Quel est le statut du pod ? (Attention, c'est un piège compare-le mentalement à la Mission 1.)
Quelle métrique précise prouve le problème ? Où la trouves-tu ? 
Le pod est running, mais lorsque que l'on regarde la consomation, par exemple avec une limite de cpu a 200m le container va être bloqué a cette limite sans la depassé. Ce qui crée du throttling. On peut voir ça avec la commande kubectl top pod cpu-demo ou voie que le CPU est a 200m (la limite dans le yaml du pods) ou dans le dashbaord Kubernetes/Compute Resources/Pod de grafana (dans la tuile CPU throttling).

2. Quel est le mécanisme kernel sous-jacent, avec ses valeurs par défaut ?
C'est le CFS (conpletly fair scheduler) du kernel linux qui va être utilisé pour rallentire le CPU sur pod et ne pas depassé la limite, ce qui va engendré des requête et une application plus lente. limite `200m` → écrite dans le cgroup (fichiers `cpu.max` en cgroup v2, qui contient quota période) → le CFS du kernel fait respecter ça.
La période CFS est de `100 ms` par défaut. Une limite de `200m` signifie donc : "le process a droit à 20 ms de CPU par tranche de 100 ms"
3. Pourquoi la latence est-elle en dents de scie (intermittente) plutôt que constante ? Il y a 20 ms de quota par période de 100 ms donc pendant 80ms le process est gelé, le dent de cies c'est le côta qui ce vide puis que ce recharge, et dans les 20ms il peut y avoir des requête rapide (plusieur requête executer) ou des requête lourde (moins de requête executer).

**Cartes à créer** : exit codes 137/139/143 et leurs signaux ; définition cgroup v2 ; RSS vs page cache.
## Ce que j'ai tenté
Écris au fil de l'eau, en vrac, comme un journal. Les fausses pistes sont les bienvenues.
  

## Cause racine
En 2-3 phrases, avec tes mots.

## Le mécanisme
L'explication "sous le capot" (kernel, réseau, kubelet...). C'est la partie qui compte.

## Ce que j'ai raté
Après lecture du corrigé : qu'est-ce que je n'avais pas vu ou mal compris ?

## Commandes à retenir
```

```
