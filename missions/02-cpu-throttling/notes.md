# Mission 02 —  Rapide mais lent

## Scénario
### Mission 2 —  Rapide mais lent
**Scénario** :  une application est lente par à-coups — des pics de latence qui vont et viennent. Tu regardes le dashboard du node qui l'héberge : le CPU est à 40%. Le développeur dit "c'est l'infra qui est sous-dimensionnée", l'équipe infra répond "le node est à moitié vide, c'est ton code". Les deux ont tort. Tranche.
**Objectif** : reproduire le phénomène. Déploie un pod qui veut consommer beaucoup plus de CPU que la limite que tu lui imposes (par exemple un process qui veut 4 cœurs avec une limite à 0,5). Puis trouve la métrique qui prouve ce qui se passe réellement.
**Critères de réussite**

**Questions de verbalisation** (réponds comme en entretien) :
1. Quel est le statut du pod ? (Attention, c'est un piège compare-le mentalement à la Mission 1.)
Quelle métrique précise prouve le problème ? Où la trouves-tu ? 
Premiére (sans rien faire) : Je pense le pods vas être running 
2. Quel est le mécanisme kernel sous-jacent, avec ses valeurs par défaut ?
3. Pourquoi la latence est-elle en dents de scie (intermittente) plutôt que constante ?

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
