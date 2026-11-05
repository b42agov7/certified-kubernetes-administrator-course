# Authentication
  - Take me to [Video Tutorial](https://kodekloud.com/topic/authentication/)
  
In this section, we will take a look at authentication in a kubernetes cluster

## Accounts

  ![auth1](../../images/auth1.PNG)
  
#### Different users that may be accessing the cluster security of end users who access the applications deployed on the cluster is managed by the applications themselves internally.

 ![acc1](../../images/acc1.PNG)
 
- So, we left with 2 types of users
  - Humans, such as the Administrators and Developers
  - Robots such as other processes/services or applications that require access to the cluster.
  

  ![acc2](../../images/acc2.PNG)
  
- All user access is managed by apiserver and all of the requests goes through apiserver.
 
  ![acc3](../../images/acc3.PNG)
  
## Authentication Mechanisms
- There are different authentication mechanisms that can be configured.

  ![auth2](../../images/auth2.PNG)
  
## Authentication Mechanisms - Basic
  
  ![auth3](../../images/auth3.PNG)
  
## kube-apiserver configuration
- If you set up via kubeadm then update kube-apiserver.yaml manifest file with the option.
  
  ![auth4](../../images/auth4.PNG)
  
## Authenticate User

- To authenticate using the basic credentials while accessing the API server specify the username and password in a curl command.
  ```
  $ curl -v -k http://master-node-ip:6443/api/v1/pods -u "user1:password123"
  ```
  ![auth5](../../images/auth5.PNG)
  
- We can have additional column in the user-details.csv file to assign users to specific groups.

  ![auth6](../../images/auth6.PNG)
  
## Note
 
 ![note](../../images/note.PNG)
  
  
#### K8s Reference Docs
- https://kubernetes.io/docs/reference/access-authn-authz/authentication/ 
  

---

u0001 est l'UID (User Identifier), c'est-à-dire l'identifiant numérique ou textuel unique attribué à l'utilisateur au sein du cluster.   Structure des fichiers CSVDans Kubernetes, ces fichiers d'authentification statique suivent un format CSV strict à 4 colonnes :   ColonneRôleExemple de l'image1Password ou Tokenpassword123 ou KpjCVbI7...   2Nom d'utilisateur (Username)user1, user10   3Identifiant unique (UID)u0001, u0010   4 (optionnel)Groupe(s)group1, group2   L'API Server utilise le nom d'utilisateur et les groupes pour faire la liaison avec les règles de contrôle d'accès (RBAC : RoleBinding, ClusterRoleBinding), tandis que l'UID sert d'identifiant interne immuable pour différencier deux utilisateurs qui porteraient éventuellement le même nom au fil du temps.Fonctionnement du mécanismeVotre description est exacte :Création du fichier : Vous écrivez le fichier CSV sur le nœud control-plane (ex. /etc/kubernetes/pki/user-token-details.csv).   Chargement dans l'API Server : Vous passez l'argument --token-auth-file=/etc/kubernetes/... dans le manifeste /etc/kubernetes/manifests/kube-apiserver.yaml (en vous assurant que le fichier est bien monté dans le conteneur).   Prise en compte : Kubelet détecte la modification du fichier manifeste et redémarre automatiquement le pod kube-apiserver avec cette configuration en mémoire.Connexion : Vous pouvez alors envoyer des requêtes API :Soit via curl : en passant le token dans l'en-tête HTTP (-H "Authorization: Bearer <token>").   Soit via kubectl : en configurant ce token dans votre fichier ~/.kube/config sous la section users: [{name: user10, user: {token: ...}}].Remarques importantes pour la pratique et l'examenBasic Auth (mot de passe statique) supprimé : Le flag --basic-auth-file a été déprécié puis complètement retiré de Kubernetes (depuis la version 1.19).Token statique déconseillé en production : --token-auth-file fonctionne toujours pour des tests ou des labs pédagogiques, mais il oblige à redémarrer l'API Server à chaque ajout ou modification d'utilisateur et stocke les secrets en clair sur le disque.   Alternatives standards : En production et pour l'examen CKA, Kubernetes privilégie les certificats clients TLS (X.509), les jetons de comptes de service (ServiceAccounts) ou un fournisseur d'identité externe (OIDC / LDAP).
