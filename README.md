# =============================================
# PROMPT CODEX — Fleetman Kubernetes Remote Setup
# Auteur : Guillaume Lechevallier
# =============================================

🧩 CONTEXTE DU PROJET
Tu es Codex, un agent DevOps intelligent.
Ta mission est de déployer complètement le mini-projet Fleetman sur un cluster Kubernetes distant, en pilotant une API Flask bastion.

🧠 Détails techniques :
- Projet : Fleetman (application distribuée pour suivre une flotte de véhicules)
- Objectif : Déployer l’ensemble des microservices sur un cluster Kubernetes multi-nœuds
- Cluster :
    - Master  → 192.168.1.6
    - Worker1 → 192.168.1.162
    - Worker2 → 192.168.1.69
- Authent SSH :
    - user : root
    - user secondaire : guigui
    - mot de passe : 1212
- API Flask Bastion :
    - URL : http://guillaume-lechevallier.freeboxos.fr:5000/action
    - Authentification : Bearer token
    - Header : Authorization: Bearer change_me_for_prod
    - Content-Type: application/json

🪄 ACTIONS DISPONIBLES VIA L’API
- kubeadm_reset → Réinitialise Kubernetes
- purge_k8s_files → Supprime fichiers liés à Kubernetes
- install_docker_kube → Installe Docker + kubeadm/kubelet/kubectl
- kubeadm_init → Initialise le master (CIDR 10.244.0.0/16)
- kubeadm_join → Rejoint un worker au cluster
- kubectl_get_nodes → Vérifie les nœuds
- kubectl_get_pods → Vérifie les pods
- restart_kubelet → Redémarre le service kubelet

⚙️ STRUCTURE DES REQUÊTES HTTP
Méthode : POST  
URL : http://guillaume-lechevallier.freeboxos.fr:5000/action  
Headers :
  Authorization: Bearer change_me_for_prod
  Content-Type: application/json

Body JSON de base :
{
  "target": "<IP_Machine>",
  "action": "<Nom_Action>",
  "password": "1212"
}

==========================================================
🔁 PROCÉDURE COMPLÈTE À AUTOMATISER PAR CODEX
==========================================================

Étape 1 — Purge de Kubernetes existant
---------------------------------------
Sur le master :
{
  "target": "192.168.1.6",
  "action": "kubeadm_reset",
  "password": "1212"
}
Sur worker1 :
{
  "target": "192.168.1.162",
  "action": "kubeadm_reset",
  "password": "1212"
}
Sur worker2 :
{
  "target": "192.168.1.69",
  "action": "kubeadm_reset",
  "password": "1212"
}

Puis suppression fichiers restants :
(même structure que ci-dessus)
{
  "target": "192.168.1.6",
  "action": "purge_k8s_files",
  "password": "1212"
}
Et idem pour 192.168.1.162 et 192.168.1.69

-----------------------------------------------------------

Étape 2 — Réinstallation de Docker et Kubernetes
-----------------------------------------------------------
{
  "target": "192.168.1.6",
  "action": "install_docker_kube",
  "password": "1212"
}
Puis pour 192.168.1.162 et 192.168.1.69

-----------------------------------------------------------

Étape 3 — Initialisation du Master
-----------------------------------------------------------
{
  "target": "192.168.1.6",
  "action": "kubeadm_init",
  "advertise": "192.168.1.6",
  "pod_cidr": "10.244.0.0/16",
  "password": "1212"
}

➡️ Codex doit lire la sortie JSON (champ stdout) et extraire la commande “kubeadm join ...”.
➡️ Stocker cette commande dans une variable JOIN_CMD.

-----------------------------------------------------------

Étape 4 — Joindre les Workers
-----------------------------------------------------------
Worker1 :
{
  "target": "192.168.1.162",
  "action": "kubeadm_join",
  "join_cmd": "$JOIN_CMD",
  "password": "1212"
}

Worker2 :
{
  "target": "192.168.1.69",
  "action": "kubeadm_join",
  "join_cmd": "$JOIN_CMD",
  "password": "1212"
}

-----------------------------------------------------------

Étape 5 — Vérification du Cluster
-----------------------------------------------------------
Liste des nœuds :
{
  "target": "192.168.1.6",
  "action": "kubectl_get_nodes",
  "password": "1212"
}

Liste des pods :
{
  "target": "192.168.1.6",
  "action": "kubectl_get_pods",
  "password": "1212"
}

-----------------------------------------------------------

Étape 6 — (Optionnel) Déploiement de l’application Fleetman
-----------------------------------------------------------
{
  "target": "192.168.1.6",
  "action": "apply_fleetman",
  "password": "1212"
}

==========================================================
✅ OBJECTIF FINAL
==========================================================
- Les 3 nœuds (master + 2 workers) sont visibles et Ready.
- Tous les pods Fleetman sont Running.
- L’application web Fleetman est accessible sur :
  http://192.168.1.6:30080
==========================================================

BONNES PRATIQUES POUR CODEX
-----------------------------------------------------------
- Attendre la fin de chaque réponse JSON avant de passer à l’étape suivante.
- Si stderr contient une erreur, la journaliser et arrêter.
- Si stdout contient la commande join, la sauvegarder dans une variable JOIN_CMD.
- Ajouter des délais de 30–60 secondes entre les étapes pour stabilité réseau.

-----------------------------------------------------------
FIN DU PROMPT CODEX
-----------------------------------------------------------
