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

## References

The scratch org creation option supported by this GitHub composite action can be found in the Salesforce CLI Command Reference here: 

- [org create scratch](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/cli_reference_org_commands_unified.htm#cli_reference_org_create_scratch_unified)

More details can be found in the related Salesforce DX Developer Guide: [Create Scratch Orgs](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_scratch_orgs_create.htm)

## Releases

Latest release notes can be found on the [release page](https://github.com/svierk/sfdx-create-scratch-org/releases).

## License

The scripts and documentation in this project are released under the [MIT License](https://github.com/svierk/sfdx-create-scratch-org/blob/main/LICENSE).