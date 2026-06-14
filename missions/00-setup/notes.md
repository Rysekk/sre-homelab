## Questions :
- De quoi est constitué un "node" Kubernetes, au minimum ? (Indice : va voir dans un conteneur k3d.) : 
  - **Un node est constituer d'un kubectl, d'un containerd et d'un kube-proxy qui s'occupe des Services.**
- Quand tu lances un pod (kubectl run), liste les composants traversés entre ta commande et le process qui tourne.

Moitié 1 — Du clic à l'enregistrement (control plane)

1. kubectl prend ta commande, la traduit en un objet Pod JSON, et l'envoie via HTTPS à l'API server.
2. L'API server authentifie ta requête, la valide, et écrit l'objet dans etcd (la base de données du cluster). À ce stade, le pod existe dans le cluster mais ne tourne nulle part. C'est juste une intention.
3. Le scheduler observe l'API server, voit "tiens, un nouveau pod sans node assigné", choisit un node en fonction des ressources/contraintes, et écrit cette décision dans l'API server.

Moitié 2 De l'enregistrement au process qui tourne (le node)

4. Sur le node choisi, kubelet observe aussi l'API server, voit "j'ai un pod à faire tourner", et passe à l'action.
5. Kubelet demande au container runtime (containerd) de créer le conteneur : pull de l'image, création des namespaces (PID, network, mount...), application des cgroups (limites mémoire/CPU dont on a parlé en Mission 1), démarrage du process.
6. CoreDNS et kube-proxy mettent à jour ce qu'il faut pour que le pod soit joignable (DNS, règles iptables des Services).