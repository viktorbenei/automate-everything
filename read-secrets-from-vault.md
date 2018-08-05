# Read Secrets from Hashicorp Vault **WIP**

Work in progress.

Base config:

```
---
format_version: 5
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
workflows:
  vault-test:
    steps:
    - script@1.1.5:
        inputs:
        - content: |-
            #!/usr/bin/env bash
            set -ex

            # download vault CLI & move the binary into PATH
            mkdir -p /tmp/vault && cd /tmp/vault
            wget -q -O vault.zip https://releases.hashicorp.com/vault/0.10.4/vault_0.10.4_linux_amd64.zip
            unzip vault.zip
            mv ./vault /usr/local/bin/vault
        title: install
    - script@1.1.5:
        title: login
        inputs:
        - content: |-
            #!/usr/bin/env bash
            set -ex

            # the vault server address - define VAULT_ADDR in Secrets
            export VAULT_ADDR="$VAULT_ADDR"

            # vault login - define VAULT_LOGIN_TOKEN in Secrets
            vault login "$VAULT_LOGIN_TOKEN"
    - script@1.1.5:
        title: use
        inputs:
        - content: |-
            #!/usr/bin/env bash
            set -ex

            # the vault server address - define VAULT_ADDR in Secrets
            export VAULT_ADDR="$VAULT_ADDR"

            # test
            vault status
            vault kv list secret

            foo_value="$(vault kv get -field=foo secret/hello)"
            echo "foo_value: $foo_value"
trigger_map: []

```

Define `VAULT_ADDR` and `VAULT_LOGIN_TOKEN` in Bitrise Secrets.
