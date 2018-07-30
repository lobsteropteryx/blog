+++
date = "2018-07-30T00:00:00+00:00"
draft = false
title = "Automating Azure Functions with Letsencrypt"
+++

[Azure Functions](https://azure.microsoft.com/en-us/services/functions/) are Microsoft's serverless offering, and they can be a great option for building simple, scalable applications and APIs.  A common practice is to apply a CNAME with your custom domain to the Azure DNS entry, and have users access your application from that domain, "hiding" the implementation.  Today HTTPS is [necessary](https://www.theverge.com/2018/2/8/16991254/chrome-not-secure-marked-http-encryption-ssl), and Azure has built-in capability to apply a custom domain and certificate to your Function App.

If you want to get your certificates from [Letsencrypt](https://letsencrypt.org/), there is a [site extension](https://github.com/sjkp/letsencrypt-siteextension/) by [Simon J.K. Pedersen](http://wp.sjkp.dk/) that will handle that for you; there are many [great](https://www.troyhunt.com/everything-you-need-to-know-about-loading-a-free-lets-encrypt-certificate-into-an-azure-website/) [posts](https://www.hanselman.com/blog/SecuringAnAzureAppServiceWebsiteUnderSSLInMinutesWithLetsEncrypt.aspx) that show how to set this up for App Services, and the same approach works for Functions as well, with some [additional steps](https://github.com/sjkp/letsencrypt-siteextension/wiki/Azure-Functions-Support).

All of the existing posts are basically how-to guides, walking you through a manual configuration.  This is fine for securing a few sites, but in production we'd ideally automate the process, especially if we're going to build many small services on top of Azure Functions.

To help with this, I've created a sample GitHub [repository](https://github.com/lobsteropteryx/azure-function-letsencrypt) that contains scripts to automate the setup of an Azure Function App as much as possible.  Using [Terraform](https://www.terraform.io/) and the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest), the scripts will:

* Create the basic scaffolding for a Function App (Resource Group, App Service, Application Insights, etc.)
* Configure the App Service with the necessary settings for the extension to work
* Publish two functions--one to manage the Letsencrypt certs, and another test function to verify the cert is working
* Configure the required proxy and route template to handle letsencrypt requests

You will still need to request a CNAME for your Function App, and manually install the extension into your Function app, but the rest of the configuration and creation is taken care of via the scripts, and can be customized as needed!



