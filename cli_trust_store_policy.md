---
tags: UX, CLI
---
# Use CLI to manage trust store and trust policy
## Overview
According to [trust store and trust policy spec](https://github.com/notaryproject/notaryproject/blob/main/trust-store-trust-policy-specification.md), user need to copy certs into directories following specific rules to use trust store, and to create a policy json file with properties well configured to use trust policy. It is a cumbersome and error-prone process not to mention multiple platform need to be supported. This document propose to develop new CLI commands to manage trust store and trust policy in an easy way.

This documdent is based on the output of a group discussion.
## Manage trust store
User wants to do CRUD on the Trust Store. The store is in the format of directory `x509/<type>/<name>/*.crt|*.cer|*.pem`. Here is an example of trust store:
```
$XDG_CONFIG_HOME/notation/trust-store
    /x509
        /ca
            /acme-rockets
                cert1.pem
                cert2.pem
                  /sub-dir       # sub directory is ignored
                    cert-3.pem   # certs under sub directory is ignored
            /acme-rockets-ca2
                cert1.pem
            /wabbit-networks
                cert3.crt
        /tsa
            /publicly-trusted-tsa
                tsa-cert1.pem
```
In this example, `type` is `ca` or `tsa`, and `name` is `acme-rockets` or `wabbit-networks`.

Existing command ```notation cert``` is to be updated to manage trust store.
### UC-TS-1: User wants to add certificates to the trust store

Command:

```notation cert add --type <type> --store <name> <cert_path>...```
- On Success: 
    - Output in console: `Succeeded to add certs to trust store <name> of <type>.`.
    - Certs are added into `{NOTATION_CONFIG}/truststore/x509/<type>/<name>/*.crt|*.cer|*.pem`
- On Failure: 
    - Output in console: `Failed to add certs to trust store <name> of <type>.`.

Here is an example: On linux, user wants to add two certs to a store named acme-rockets of CA type

User executes `notation cert add --type ca --store acme-rockets acme_root.crt acme_root_2.crt`

One success, two certs are added as the following:
- `$HOME/.config/notation/truststore/x509/ca/acme-rockets/acme_root.crt`
- `$HOME/.config/notation/truststore/x509/ca/acme-rockets/acme_root_2.crt`

### UC-TS-2: User wants to list certificates stored in the trust store
- List all certificates:
    - Command: `notation cert list`
- List all certificates of a certain named store
    - Command: `notation cert list --store <name>`
- List all certificates of a certain named store of a certain type
    - Command: `notation cert list --type <type> --store <name>`
### UC-TS-3: User wants to show details of a certain certificate
Command:

`notation cert show --type <type> --store <name> <cert_file_name>`

- On Success:
    - Output in console: <TODO>
- On Failure:
    - Output in console: `Failed to show details of certificate <cert_file_name>`

### UC-TS-4: User wants to delete certificates
- Delete all certificates of a certain named store
    - Command: `notation cert delete --store <name> --all`
- Delete all certificates of a certain named store of a certain type
    - Command: `notation cert delete --type <type> --store <name> --all`
- Delete a specific certificate of a certain named store of a certain type
    - Command: `notation cert delete --type <type> --store <name> <cert_file_name>`


## Manage trust policy
User wants to do CRUD on the Trust Policy. Here is an example of trust policy:
```
{
    "version": "1.0",
    "trustPolicies": [
        {
            // Policy for all artifacts, from any registry location.
            "name": "wabbit-networks-images",   // Name of the policy.
            "registryScopes": [ "*" ],          // The registry artifacts to which the policy applies.
            "signatureVerification": {          // The level of verification - strict, permissive, audit, skip.
              "level" : "audit" 
            },
            "trustStores": ["ca:acme-rockets"], // The trust stores that contains the X.509 trusted roots.
            "trustedIdentities": [              // Identities that are trusted to sign the artifact.
              "x509.subject: C=US, ST=WA, L=Seattle, O=acme-rockets.io, OU=Finance, CN=SecureBuilder"
            ]
        }
    ]
}
```
A new command ```notation policy``` to be introduced.
### UC-TP-1: User wants to create trust policies
### UC-TP-2: User wants to inspect trust policies
### UC-TP-3: User wants to update trust policies
### UC-TP-4: User wants to delete trust policies

