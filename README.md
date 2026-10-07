# Statuttendring

Plain HTML app for reviewing Samfunnet i Bergen's statutes and preparing
amendment proposals, written by Morten Stene.

`index.html` is restored unchanged from
[the original app](https://github.com/Mortey12/Mortey12.github.io/blob/3b6f0277c7e28da5cadd38d29ae7e84821b52e32/Samf-statutter/index.html).

The app is hosted at [statuttendring.samfunnetibergen.no](https://statuttendring.samfunnetibergen.no).

## Local development

Serve the repository with any static HTTP server, for example:

```sh
python3 -m http.server 8000
```

Open `http://localhost:8000`. No build or package installation is required.
The statutes, styles, application code, and Word form template are embedded in
`index.html`. JSZip and fonts load from their public CDNs. Drafts are stored in
the visitor's browser and can be exported or imported as JSON files.

## Deploy to Azure Static Web Apps

The Azure resource is `samfunnet-statutter`, in resource group
`rg-samfunnet-statutter` (West Europe, Free tier). Its default hostname is
`black-river-05d6a6903.6.azurestaticapps.net`; the custom hostname is
`statuttendring.samfunnetibergen.no`.

Pushes to `main` automatically deploy through the
[Deploy to Azure workflow](https://github.com/kvarteret/statuttendring/actions/workflows/deploy-azure.yml).
It can also be run manually on `main` from GitHub Actions. The job uploads only
`index.html` and verifies that the public site matches the committed file.

The workflow uses the repository secret `AZURE_STATIC_WEB_APPS_API_TOKEN`,
containing the deployment token for `samfunnet-statutter`. If the token is
rotated in Azure, update this GitHub Actions secret as well.

To deploy manually from your computer, sign in to Azure CLI with an account
that can deploy to this resource. From the repository root, run:

```sh
mkdir -p dist
cp index.html dist/index.html
SWA_CLI_DEPLOYMENT_TOKEN="$(az staticwebapp secrets list \
  --name samfunnet-statutter \
  --resource-group rg-samfunnet-statutter \
  --query 'properties.apiKey' -o tsv)" \
  npx --yes @azure/static-web-apps-cli deploy dist --env production
```

The deployment token is retrieved from Azure for the deploy command. Do not
store it in this repository.

## Domeneshop DNS

In the `samfunnetibergen.no` DNS zone, configure:

| Host | Type | Value |
| --- | --- | --- |
| `statuttendring` | CNAME | `black-river-05d6a6903.6.azurestaticapps.net` |

If the Azure custom-domain binding needs to be registered again, create it and
get the current verification token:

```sh
az staticwebapp hostname set \
  --name samfunnet-statutter \
  --resource-group rg-samfunnet-statutter \
  --hostname statuttendring.samfunnetibergen.no \
  --validation-method dns-txt-token --no-wait

az staticwebapp hostname show \
  --name samfunnet-statutter \
  --resource-group rg-samfunnet-statutter \
  --hostname statuttendring.samfunnetibergen.no \
  --query validationToken -o tsv
```

Add the returned token in Domeneshop as a TXT record with host
`_dnsauth.statuttendring`. Keep the TXT record alongside the CNAME. The token is
specific to the Azure binding, so retrieve it from Azure when needed instead of
storing it here.
