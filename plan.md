# Plan d'apprentissage SRE — Missions Homelab

**Méthode** : pour chaque mission, tu n'as que le scénario, l'objectif et les critères de réussite. Pas de commandes. Tu luttes seul 30 à 60 minutes minimum avant toute aide. Le guide précédent (`homelab-sre-guide.md`) est ton **corrigé** : tu ne l'ouvres qu'après ta tentative, pour comparer ta démarche.

**Le rituel, pour chaque mission :**
1. **Lutte** (30-60 min) — man pages, doc officielle, tâtonnement autorisés. Copier-coller un tuto qui résout la mission : interdit.
2. **Corrigé** — compare ta démarche avec le guide. Note ce que tu as raté ou fait autrement.
3. **Verbalisation** — réponds aux questions d'entretien de la mission **à voix haute, sans notes**, puis écris les réponses dans ton repo `homelab-notes`. Si tu hésites ou baratines, c'est non acquis : retourne manipuler.
4. **Cartes** — crée les flashcards indiquées (Anki ou équivalent). Révision : 10 min chaque matin.

**Si tu bloques pendant la lutte** : reviens me voir avec ce que tu as tenté, je te donne des indices gradués — jamais la solution directe.

**Rythme suggéré** : 3-4 soirées par semaine, 1 mission par soirée. Le plan fait 4 semaines. Aller plus vite n'est pas un objectif ; pouvoir tout expliquer sans notes, oui.

---

## Semaine 0 — Plomberie (copier-coller autorisé, 1 soirée)

Installe Docker, kubectl, k3d, Helm, et les outils de debug, puis monte le cluster 3 nodes et kube-prometheus-stack en suivant les phases 0 à 2 du guide. Déploie l'app de démo microservices. Ici, zéro culpabilité à copier-coller : c'est de l'installation, pas de l'apprentissage.

**Seule exigence** : avant de passer à la semaine 1, réponds à ceci sans regarder :
- De quoi est constitué un "node" Kubernetes, au minimum ? (Indice : va voir dans un conteneur k3d.)
- Quand tu lances un pod, liste les composants traversés entre ta commande et le process qui tourne.

---

## Semaine 1 — Linux internals (ton 7/10 → 9/10)

### Mission 1 — Le pod qui meurt en silence
**Scénario** : un pod redémarre en boucle. L'équipe applicative jure que son code est correct.
**Objectif** : provoque toi-même ce scénario — déploie un pod qui consomme plus de mémoire que sa limite — puis mène l'investigation jusqu'à la cause racine **côté système**, pas seulement côté Kubernetes.
**Critères de réussite** : tu connais le statut exact du pod, son exit code, l'identité précise de ce qui a tué le process, et l'endroit où le système a journalisé l'événement.

**Questions de verbalisation** (réponds comme en entretien) :
1. Qui a tué le process ? Sois précis : « Kubernetes » est une réponse fausse.
2. Pourquoi exit code 137 et pas un autre ? Décompose le calcul.
3. Une limite mémoire Kubernetes, ça se matérialise comment côté Linux ?
4. Question piège d'interviewer : « Le pod utilisait 80 Mi de RSS pour une limite de 100 Mi et s'est quand même fait tuer. Hypothèses ? »

**Cartes à créer** : exit codes 137/139/143 et leurs signaux ; définition cgroup v2 ; RSS vs page cache.

### Mission 2 — Rapide mais lent
**Scénario** : une app est lente par à-coups. Le dashboard montre un node utilisé à 40% CPU. Le dev accuse l'infra, l'infra accuse le dev.
**Objectif** : reproduis le phénomène avec un pod dont la limite CPU est très inférieure à sa demande réelle, et trouve la métrique qui prouve ce qui se passe.
**Critères de réussite** : tu peux montrer la métrique de throttling dans Grafana et expliquer le mécanisme de quota sous-jacent avec ses valeurs par défaut.

**Questions de verbalisation** :
1. Explique le throttling CFS à quelqu'un qui ne connaît que « les limites CPU » de Kubernetes.
2. Pourquoi la latence apparaît en dents de scie plutôt que constante ?
3. Débat d'équipe : « faut-il mettre des limits CPU en prod ? » Donne les deux camps et ta position.
4. Quelle différence de mécanisme entre une limite CPU et une limite mémoire ? (L'une throttle, l'autre tue — pourquoi cette asymétrie ?)

**Cartes** : période CFS par défaut ; signification de 500m ; nom de la métrique de throttling.

### Mission 3 — Le serveur mystère
**Scénario** : un serveur « rame ». Aucun APM, aucun dashboard. Tu as un shell et 5 minutes.
**Objectif** : fais-toi générer une charge surprise (lance un stress I/O sur un node sans regarder lequel ni quoi), puis identifie la nature de la charge avec une méthode structurée — pas en essayant des commandes au hasard.
**Critères de réussite** : tu as identifié qu'il s'agit d'I/O (pas de CPU) en moins de 5 minutes, et tu peux nommer et dérouler la méthode que tu as suivie.

**Questions de verbalisation** :
1. Déroule tes 5 premières minutes sur un serveur lent inconnu, dans l'ordre, en justifiant chaque étape.
2. Load average à 12 sur une machine 8 CPU, mais CPU idle à 70% : explique.
3. C'est quoi un process en D-state, et pourquoi c'est important pour interpréter le load ?
4. Décris la méthode USE et applique-la au sous-système disque.

**Cartes** : les 3 chiffres du load average ; %wa dans top ; méthode USE (les 3 lettres et leur sens).

---

## Semaine 2 — Réseau (ton 6/10 → 8/10, le plus gros chantier)

### Mission 4 — Tout le monde timeout
**Scénario** : après un déploiement, une app multi-services part en erreurs. Symptôme troublant : chaque requête inter-service met exactement ~5 secondes avant d'échouer.
**Objectif** : casse la résolution DNS du namespace de démo avec une NetworkPolicy (à toi de découvrir laquelle et pourquoi elle casse ça), constate les symptômes, puis debug depuis l'intérieur d'un pod ET depuis le node avec une capture de paquets.
**Critères de réussite** : tu peux montrer dans une capture les requêtes DNS qui partent sans réponse, expliquer pourquoi le timeout est de 5 s précisément, et réparer.

**Questions de verbalisation** :
1. Déroule le chemin complet d'une résolution DNS depuis un pod : fichiers consultés, composants traversés, jusqu'à la réponse.
2. Pourquoi « exactement 5 secondes » est un indice en soi ?
3. C'est quoi `ndots:5`, et quel effet pervers ça a sur les résolutions de noms externes ?
4. Latence intermittente entre deux services, déroule ton plan de debug complet (la question où tu as perdu des points : prépare 2 minutes structurées).

**Cartes** : timeout DNS par défaut ; rôle de CoreDNS ; contenu type d'un resolv.conf de pod ; ndots.

### Mission 5 — L'autopsie des connexions
**Scénario** : un serveur accumule des milliers de connexions dans un état bizarre et finit par refuser les nouvelles.
**Objectif** : pendant que l'app de démo tourne, inventorie les états TCP présents sur un node, identifie les états normaux et ceux qui seraient pathologiques, et provoque (ou simule) une accumulation de CLOSE_WAIT pour voir la signature.
**Critères de réussite** : tu peux dessiner de tête le cycle de vie d'une connexion TCP (ouverture ET fermeture) et dire, pour TIME_WAIT et CLOSE_WAIT, lequel est sain et lequel révèle un bug — et chez qui.

**Questions de verbalisation** :
1. Dessine (ou décris) le handshake d'ouverture puis la séquence de fermeture, avec les états de chaque côté.
2. TIME_WAIT : pourquoi ça existe, combien de temps, et de quel côté ?
3. CLOSE_WAIT qui s'accumule : c'est un bug de qui, et quelle en est la conséquence finale pour le process ?
4. Tu envoies un SYN : dans un cas tu reçois un RST, dans l'autre rien. Qu'en déduis-tu dans chaque cas ?

**Cartes** : handshake 3 étapes ; durée TIME_WAIT ; RST vs silence ; fuite de file descriptors.

### Mission 6 — Le chemin invisible
**Scénario** : question d'entretien pure : « un pod appelle un Service ClusterIP — racontez-moi tout ce qui se passe. »
**Objectif** : va vérifier la réponse toi-même. Depuis un node, retrouve les règles que kube-proxy a installées pour un Service de l'app de démo, et suis le chemin jusqu'à l'IP d'un pod réel.
**Critères de réussite** : tu peux citer la chaîne de traduction ClusterIP → pod avec le nom des mécanismes impliqués, preuves à l'appui dans la sortie de tes commandes.

**Questions de verbalisation** :
1. Un Service ClusterIP, est-ce qu'il « existe » quelque part ? Où vit cette IP ?
2. Rôle exact de kube-proxy — et que se passe-t-il s'il meurt sur un node ? (Réfléchis avant de répondre, la réponse surprend.)
3. iptables vs IPVS pour kube-proxy : pourquoi ce choix existe ?
4. Différence ClusterIP / NodePort / LoadBalancer / Headless — et un cas d'usage chacun.

**Cartes** : les 4 types de Service ; rôle des Endpoints/EndpointSlices ; où vit une ClusterIP.

---

## Semaine 3 — Kubernetes internals (ton 6.5/10 → 8.5/10)

### Mission 7 — L'app saine qu'on assassine
**Scénario** : une app parfaitement fonctionnelle mais lente à démarrer est en CrashLoopBackOff permanent depuis qu'un collègue a « amélioré le monitoring ».
**Objectif** : reproduis ce scénario avec une liveness probe trop agressive sur une app à démarrage lent, observe le cycle, puis corrige de deux façons différentes (au moins).
**Critères de réussite** : tu peux expliquer la différence d'effet entre liveness, readiness et startup probe, et la progression exacte du backoff de CrashLoopBackOff.

**Questions de verbalisation** :
1. Liveness vs readiness : conséquence exacte d'un échec de chacune. Lequel restart, lequel non ?
2. Pourquoi mettre la connexion à la base de données dans une liveness probe est une bombe à retardement ? Raconte le scénario catastrophe complet.
3. À quoi sert la startup probe et quand l'utiliser ?
4. CrashLoopBackOff : décris le backoff (valeurs, plafond) et ce que ça implique pour le MTTR.

**Cartes** : effets liveness/readiness/startup ; backoff CrashLoop ; anti-pattern « probe → dépendance externe ».

### Mission 8 — Le node qui craque
**Scénario** : un node passe en état dégradé et des pods disparaissent — mais pas n'importe lesquels, et pas au hasard.
**Objectif** : remplis le disque d'un node jusqu'à déclencher la pression disque, et observe quels pods sont évincés en premier. Avant de regarder, **prédit** l'ordre d'éviction par écrit, puis vérifie.
**Critères de réussite** : ta prédiction était argumentée par les classes QoS ; tu sais retrouver les conditions du node et les événements d'éviction.

**Questions de verbalisation** :
1. Les 3 classes QoS : comment chacune s'obtient (requests/limits) et son effet sur l'éviction et l'OOM score.
2. Différence entre éviction par kubelet et OOMKill par le kernel — déclencheur, décideur, granularité.
3. Quels sont les signaux d'éviction surveillés par kubelet et leurs seuils par défaut ?
4. Question senior : « comment garantir qu'un pod critique ne sera jamais évincé ? » (Plusieurs couches de réponse.)

**Cartes** : 3 classes QoS et leur recette ; ordre d'éviction ; seuil nodefs par défaut ; PriorityClass.

### Mission 9 — Reconstruire la stack (mission longue, 2-3 soirées)
**Scénario** : un client veut ta stack LGTM, mais dans Kubernetes, proprement, en Helm.
**Objectif** : porte ton repo `LGTM-stack` (docker-compose) en chart Helm déployé dans ton cluster. Choisis toi-même les bons objets K8s pour chaque composant — et sache justifier chaque choix.
**Critères de réussite** : la stack tourne dans le cluster ; tu peux justifier StatefulSet vs Deployment pour chaque composant, l'usage des PVC, des ConfigMaps, et pourquoi l'OTel Collector est (ou n'est pas) un DaemonSet ; le chart est publié sur ton GitHub.

**Questions de verbalisation** :
1. StatefulSet vs Deployment : différences réelles (identité, stockage, ordre) et pourquoi Loki en a besoin (ou pas, selon ton mode de déploiement — défends ton choix).
2. DaemonSet : définition, et pourquoi c'est le pattern naturel d'un collecteur de logs/métriques node-level.
3. Que se passe-t-il pour un PVC quand le pod meurt ? Quand le StatefulSet est supprimé ?
4. Pitch de 2 minutes : présente ce projet comme en entretien (« parlez-moi d'un projet perso »).

**Cartes** : StatefulSet vs Deployment (3 différences) ; cycle de vie PVC ; pattern DaemonSet.

---

## Semaine 4 — Consolidation et simulation

### Mission 10 — La panne composée
**Scénario** : tu me demandes (dans une conversation) de te concevoir une panne **combinant deux mécanismes des semaines précédentes** sans te dire lesquels. Tu debug en aveugle et tu rédiges un mini post-mortem : symptômes, démarche, cause racine, mécanisme, remédiation.
**Critères de réussite** : cause racine trouvée sans corrigé, post-mortem d'une page dans ton repo.

### Mission 11 — L'entretien blanc
**Format** : une session avec moi en mode interviewer strict, couvrant les 3 domaines. Je creuse chaque réponse avec des « pourquoi » successifs jusqu'à toucher le fond de ta compréhension, comme un vrai entretien senior. On note ce qui tient et ce qui craque, et on re-cible.

### Rituel final de la semaine 4
Relis ton repo `homelab-notes` et, pour chaque mission, réponds à la question de verbalisation n°1 à voix haute, sans notes, en moins de 2 minutes. Tout ce qui dépasse 2 minutes ou hésite : c'est ta liste de révision pour la suite.

---

## Règles du jeu (rappel)
- La frustration pendant la lutte n'est pas un signal d'échec, c'est le mécanisme d'apprentissage lui-même. 30 minutes de blocage = normal et productif.
- Le corrigé après la tentative, jamais avant.
- Une réponse de verbalisation que tu ne peux pas donner à voix haute sans notes n'est pas acquise.
- 10 min de cartes chaque matin — c'est non négociable, c'est ce qui empêche la semaine 1 de s'évaporer pendant la semaine 3.
- Quand tu bloques : viens me voir avec tes tentatives et tes sorties de terminal, je t'indice sans te solutionner.