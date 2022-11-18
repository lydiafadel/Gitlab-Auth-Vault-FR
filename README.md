# Gitlab-Auth-Vault

Etape 0: Paramétrer varaibles environnement :

export VAULT_ADDR=https://vault-cluster-prod-public-vault-fa7b3917.bc6c60bc.z1.hashicorp.cloud:8200
export VAULT_TOKEN=
export VAULT_NAMESPACE=admin

Etape 1 : Paramétrer l'application oidc application dans votre Gitlab 

Récupérer l'application ID et secret 


Etape 2 : Activer la méthode authentification oidc sur le namespace voulu à travers l'interface utilisateur ou le CLI

      vault auth enable  oidc

ETape 3 : Paramétrer la configuration oidc

  vault write auth/oidc/config \
    oidc_discovery_url="https://gitlab.com" \
    oidc_client_id="mettre l'application ID généré sur l'application gitlab" \
    oidc_client_secret="mettre le secret généré sur l'application gitlab" \
    default_role="demo" \
    bound_issuer="localhost"
  
Ma configuration est rattaché au rôle "demo" que je vais créer dans l'étape suivante

Etape 4 : Definir un role et y attacher une policy déjà créé dans le namespace admin

  vault write auth/oidc/role/demo -<<EOF
{
   "user_claim": "sub",
   "allowed_redirect_uris": ["http://localhost:8250/oidc/callback", "https://vault-cluster-prod-public-vault-fa7b3917.bc6c60bc.z1.hashicorp.cloud:8200/ui/vault/auth/oidc/oidc/callback"],
   "bound_audiences": "mettre l'application ID généré sur l'application gitlab",
   "oidc_scopes": ["openid"], 
   "role_type": "oidc", 
   "policies": ["default"],
   "ttl": "1h",
   "bound_claims": {"groups": ["vault20/developers"]}
}   
EOF

Bound claims correspond aux groupes/projet et sous groupes gitlab que vous aurez créé. Le nom du groupe diffère de celui que vous avez créé (dans mon cas j'ai créé vault mais le claim correspondant est vault20. Vous pouvez le trouver dans les paramètres avancés de votre groupe ou en faisant un appel API avec postman ou autre outil.

Définissez un tt, temps pendant lequel le token d'authentification généré sera valide.

Connectez vous sur l'interface utilisateur en mettant le nom du role ou depuis un shell en local :
   vault login -method=oidc port=8250 role=demo
   
Autre option: lorsque que vous vous connectez en local, il vous génère un token que vous pouvez utiliser pour vous connecter depuis l'interface utilisateur sans passer par le role.

Lien vers la documentation gitlab :
   https://docs.gitlab.com/ee/integration/vault.html
