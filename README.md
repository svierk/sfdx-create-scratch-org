# 🌩️ SFDX Create Scratch Org
 
This repository implements a simple GitHub composite action for creating Salesforce scratch orgs.

## Usage

In a GitHub workflow, the use of the action after installing the SF CLI and authorizing the respective DevHub Org could look like this:

```
jobs:
  create-scratch-org:
    name: Create Scratch Org
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@main
        with:
          fetch-depth: 0

      - name: Select Node Version
        uses: svierk/get-node-version@main

      - name: Install SF CLI
        uses: svierk/sfdx-cli-setup@main

      - name: Authorize DevHub
        uses: svierk/sfdx-login@main
        with:
          client-id: ${{ secrets.SFDX_CONSUMER_KEY }}
          jwt-secret-key: ${{ secrets.SFDX_JWT_SECRET_KEY }}
          username: ${{ secrets.SFDX_USERNAME }}
          set-default: true
          set-default-dev-hub: true
          alias: devhub

      - name: Create Scratch Org
        uses: svierk/sfdx-create-scratch-org@main
        with:
          alias: scratch-${{ github.run_id }}
          name: ${{ inputs.name }}
          description: ${{ inputs.description }}
          admin-email: ${{ inputs.email }}
          set-default: true
          definition-file: config/project-scratch-def.json
          target-dev-hub: devhub
          duration-days: ${{ inputs.lifespan }}
          wait: 10

      - name: Package Installation
        if: ${{ inputs.install-packages }}
        uses: svierk/sfdx-package-installation@main
        with:
          packages: ${{ inputs.packages }}
          wait: 30
          publish-wait: 20

      - name: Deploy Metadata
        if: ${{ inputs.deploy-metadata }}
        run: sf project deploy start

      - name: Generate Password
        run: sf org generate password --target-org scratch-${{ github.run_id }}
```

The following actions were also used in the example workflow above:

- [Get Node Version](https://github.com/svierk/get-node-version) | Pulls Node.js version to be used from the _package.json_ of the project
- [SFDX CLI Setup](https://github.com/svierk/sfdx-cli-setup) | Installs the Salesforce CLI and related plugins
- [SFDX Login](https://github.com/svierk/sfdx-login) | Handles Salesforce login using a JSON web token (JWT)
- [SFDX Package Installation](https://github.com/svierk/sfdx-package-installation) | Handles Salesforce package installation on target org

Of course, the create scratch org action can be used flexibly and the respective approach can vary.

## Inputs

| Name              | Required | Default | Description                                                                                   |
| ----------------- | -------- | ------- | --------------------------------------------------------------------------------------------- |
| `target-dev-hub`  | yes      |         | Username or alias of the Dev Hub org.                                                          |
| `alias`           | no       |         | Alias for the scratch org.                                                                     |
| `set-default`     | no       |         | Set the scratch org as your default org.                                                       |
| `definition-file` | no       |         | Path to a scratch org definition file (blueprint for the scratch org).                         |
| `edition`         | no       |         | Salesforce edition, e.g. `developer`, `enterprise`. Overrides the definition file.             |
| `duration-days`   | no       |         | Number of days before the org expires (1–30).                                                  |
| `wait`            | no       |         | Number of minutes to wait for the scratch org to be ready.                                     |
| `api-version`     | no       |         | Override the api version used for api requests.                                                |
| `client-id`       | no       |         | Consumer key of the Dev Hub connected app.                                                     |
| `username`        | no       |         | Username of the scratch org admin user. Omit to auto-generate a unique username.               |
| `description`     | no       |         | Description of the scratch org in the Dev Hub.                                                 |
| `name`            | no       |         | Name of the org, e.g. `Acme Company`.                                                          |
| `release`         | no       |         | Release of the scratch org relative to the Dev Hub release.                                    |
| `admin-email`     | no       |         | Email address applied to the org's admin user.                                                |
| `source-org`      | no       |         | 15-character ID of the org whose shape the new scratch org is based on.                        |
| `step-summary`    | no       | `true`  | Write a result section to the GitHub Actions [job summary](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary). Set to `false` to avoid collisions with a custom workflow summary. |

## Outputs

The action exposes the details of the newly created scratch org so that follow-up steps (deploy, test, delete) can target it without guessing the generated username:

| Name       | Description                          |
| ---------- | ------------------------------------ |
| `username` | Username of the created scratch org. |
| `org-id`   | ID of the created scratch org.       |

```yaml
- name: Create Scratch Org
  id: scratch
  uses: svierk/sfdx-create-scratch-org@main
  with:
    target-dev-hub: devhub
    definition-file: config/project-scratch-def.json
    duration-days: 1

- name: Deploy Metadata
  run: sf project deploy start --target-org ${{ steps.scratch.outputs.username }}

- name: Delete Scratch Org
  if: always()
  run: sf org delete scratch --target-org ${{ steps.scratch.outputs.username }} --no-prompt
```

## References

The scratch org creation option supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here: 

- [org create scratch](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_create_scratch_unified)

More details can be found in the related Salesforce DX Developer Guide: [Create Scratch Orgs](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs_create.htm)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-create-scratch-org/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-create-scratch-org/blob/main/LICENSE).