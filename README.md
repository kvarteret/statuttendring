# Statuttendring

Standalone Svelte app for reviewing Samfunnet i Bergen's statutes and preparing
amendment proposals. The original author was Morten Stene.

The app is hosted at [statuttendring.samfunnetibergen.no](https://statuttendring.samfunnetibergen.no).

## Local development

Requires Node.js 22.12 or later.

```sh
npm ci
npm run dev
```

Create the production files with `npm run build`; Vite writes them to `dist/`.

The statutes are stored in `src/lib/statuttar.json`, and the Word form template
is in `public/statuttendringsskjema.docx`. Drafts are stored in the visitor's
browser and can be exported or imported as JSON files.

## Deploy to Azure Static Web Apps

The Azure resource is `samfunnet-statutter`, in resource group
`rg-samfunnet-statutter` (West Europe, Free tier). Its default hostname is
`black-river-05d6a6903.6.azurestaticapps.net`; the custom hostname is
`statuttendring.samfunnetibergen.no`.

Sign in to Azure CLI with an account that can deploy to this resource. From the
repository root, run:

```sh
npm run build
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
