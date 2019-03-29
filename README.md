# AzureKeyVault

R interface to [Azure Key Vault](https://azure.microsoft.com/services/key-vault/), a secure service for managing private keys, secrets and certificates.

You can install the development version of the package from GitHub:

```r
devtools::install_github("cloudyr/AzureKeyVault")
```

## Resource Manager interface

AzureKeyVault extends the [AzureRMR](https://github.com/cloudyr/AzureRMR) package to handle key vaults. In addition to creating and deleting vaults, it provides methods to manage access policies for user and service principals.

```r
# create a key vault
kv <- AzureRMR::get_azure_login()$
    get_subscription("sub_id")$
    get_resource_group("rgname")$
    create_key_vault("mykeyvault")

# list current principals (by default includes logged-in user)
kv$list_principals()

# get details for a service principal
svc <- AzureGraph::get_graph_login()$
    get_service_principal("app_id")

# give the service principal read-only access to vault keys and secrets
kv$add_principal(svc,
    key_permissions=c("get", "list", "backup"),
    secret_permissions=c("get", "list", "backup"),
    certificate_permissions=NULL)
```

## Client interface

The client interface is R6-based, with methods for keys, secrets and certificates. To access the vault, instantiate a new object of class `key_vault`.

```r
vault <- key_vault("mykeyvault")

# can also be done from the ARM resource object
#vault <- kv$get_endpoint()

# create a new secret
vault$secrets$set("newsecret", "secret value")
vault$secrets$show("newsecret")

# create a new RSA key with 4096-bit key size
vault$keys$create("newkey", type="RSA", rsa_key_size=4096)

# create a new self-signed certificate (will also create the associated key and secret)
vault$certificates$create("newcert",
    issuer=list(name="self"),
    secret=list(contentType="application/x-pkcs12"),
    x509=list(subject="CN=mydomain.com", sans=list(dns_names=list("mydomain.com")))
)
```

---
[![cloudyr project logo](https://i.imgur.com/JHS98Y7.png)](https://github.com/cloudyr)
