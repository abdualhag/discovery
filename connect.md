---

copyright:
  years: 2015, 2018, 2019
lastupdated: "2019-01-02"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:tip: .tip}
{:note: .note}
{:pre: .pre}
{:important: .important}
{:deprecated: .deprecated}
{:codeblock: .codeblock}
{:screen: .screen}
{:download: .download}
{:hide-dashboard: .hide-dashboard}
{:apikey: data-credential-placeholder='apikey'} 
{:url: data-credential-placeholder='url'}
{:curl: #curl .ph data-hd-programlang='curl'}
{:javascript: .ph data-hd-programlang='javascript'}
{:java: .ph data-hd-programlang='java'}
{:python: .ph data-hd-programlang='python'}
{:ruby: .ph data-hd-programlang='ruby'}
{:swift: .ph data-hd-programlang='swift'}
{:go: .ph data-hd-programlang='go'}

# Connecting to Data Sources
{: #sources}

The {{site.data.keyword.discoveryshort}} service lets you connect to and crawl documents from remote sources.
{: shortdesc}

You can connect to a data source and pull documents on a schedule (if desired) into the {{site.data.keyword.discoveryshort}} service by configuring a collection to associate with that source. Each collection can be configured with one data source. The {{site.data.keyword.discoveryshort}} service pulls documents from the data source using a process called crawling. Crawling is the process of systematically browsing and retrieving documents from the specified start location. Only items explicitly specified by you are crawled by the {{site.data.keyword.discoveryshort}} service. The following types of data sources can be crawled:

-  [Box](/docs/services/discovery/connect.html#connectbox)
-  [Salesforce](/docs/services/discovery/connect.html#connectsf)
-  [Microsoft SharePoint Online](/docs/services/discovery/connect.html#connectsp)
-  [Microsoft SharePoint 2016 On-Premise](/docs/services/discovery/connect.html#connectsp_op)
-  [Web Crawl](/docs/services/discovery/connect.html#connectwebcrawl) (beta)

Connecting to a data source can be performed using the {{site.data.keyword.discoveryshort}} tooling, or the API. The {{site.data.keyword.discoveryshort}} tooling provides a simplified method of connection that requires less understanding of the source systems, and the API provides a more granular and highly configurable interface which required a greater understanding of the source that you are connecting to. The following process overview let you know which sections of this document to read next:

1.  Read the [General Source Requirements](/docs/services/discovery/connect.html#gen_req) to get a general understanding of what will be required.
2.  Read the requirements specific to your source system:
    -  [Box](/docs/services/discovery/connect.html#connectbox)
    -  [Salesforce](/docs/services/discovery/connect.html#connectsf)
    -  [Microsoft SharePoint Online](/docs/services/discovery/connect.html#connectsp)
    -  [Microsoft SharePoint 2016 On-Premise](/docs/services/discovery/connect.html#connectsp_op)
    -  [Web Crawl](/docs/services/discovery/connect.html#connectwebcrawl) (beta)
3.  Read the source configuration instructions based on your configuration choice:
    -  [Using the tooling](/docs/services/discovery/connect.html#source_tooling)
    -  [Using the API](/docs/services/discovery/connect.html#source_api)

If you select an on-premise data source, you must first install and configure IBM Secure Gateway. See [Installing IBM Secure Gateway for on-premise data](/docs/services/discovery/connect.html#gateway) for more information.

## General Source Requirements
{: #gen_req}

The following general requirements apply to all data sources:

-  The individual document file size limit for Box, Salesforce, SharePoint Online, and SharePoint 2016 is 10MB.
-  You will need the credentials and file locations (or URLs) for each data source - these are typically provided by a developer/system administrator of the data source.
-  You will need to know which resources of the data source to crawl. This can be provided by the source administrator. When crawling Box or Salesforce, a list of available resources is presented when configuring a source using the {{site.data.keyword.discoveryshort}} tooling.
-  Crawling a data source will use resources (API calls) of the data source. The number of calls depends on the number of documents crawled. An appropriate level of service (for example Enterprise) must be obtained for the data source, and the source system administrator consulted.
-  The following file types can be ingested by {{site.data.keyword.discoveryshort}}, all other types of document are ignored:
   -  Microsoft Word
   -  PDF
   -  HTML
   -  JSON
-  {{site.data.keyword.discoveryshort}} source crawls do not delete documents that are stored in a collection. When a source is re-crawled, new documents are added, updated document are modified to the current version, and deleted documents remain as the version last stored.

## Box
{: #connectbox}

You'll need to create a new Box custom application to connect to {{site.data.keyword.discoveryfull}}. The Box application you create requires either Enterprise level or Application level access.

-  If you are not the Box administrator for your organization, [**Application level** access](/docs/services/discovery/connect.html#applevelbox) is recommended. You will need an administrator to approve your application.

-  If you are the Box administrator for your organization, [**Enterprise level** access](/docs/services/discovery/connect.html#entlevelbox) is recommended.

**Note:** The steps to setup Box access may change if there is a Box update. Consult the [Box developer documentation ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://developer.box.com/){: new_window} for updates.

### Setting up Application level access
{: #applevelbox}

1.   Navigate to `https://app.box.com/developers/console` (use your company's Box URL) and click **Create New App**.
1.   From the **Create a New App** screen, select **Enterprise Integration** and click **Next**.
1.   On the **Authentication Method** screen, select **OAuth 2.0 with JWT (Server Authentication)** and click **Next**.
1.   Name your app and click the **Create App** button. Once your Box app has been created, click **View Your App**.
1.   While viewing your app, select **Application access** of **Application**. You can use your existing managed users as you continue; you are not required to create new ones.
1.   Scroll down to the **Application Scopes** section of the page and enable the following checkboxes: 
     - `Read and write all folders stored in Box`
     - `Manage Users`
     - `Manage Enterprise Properties` 
1.   Scroll down to the **Advanced Features** section and enable the following toggles:
     - `Perform Actions as Users`
     - `Generate User Access Tokens` 
1.  Click the **Save Changes** button.

The next few steps will require assistance from the Administrator of your organization's Box account. If you are not your organization's Box Administrator, you can identify the Administrator by opening the Box developer's console and looking under **Account settings** > **Account details** > **Settings**.

1.  [Administrator step] Authorize your application client id at `https://app.box.com/master/settings/openbox` by clicking the **Authorize New App** button.
1.  [Administrator step] Type the **client ID** from `https://app.box.com/developers/console` into the **API key** field and then click the **Authorize** button.
1.  [Administrator step] Retrieve a developer token for the application. To do this, navigate to `https://app.box.com/developers/console`, scroll to the **Developer Token** section and generate your token.
1.  Now that the application has been authorized by the Administrator, navigate to `https://developer.box.com/reference#page-create-an-enterprise-user` and create the Box user using the API reference page.

   Sample curl command [from the box API](https://api.box.com/2.0/users):

   ```bash
    curl -X POST -H "Authorization: Bearer <YOUR_DEVELOPER_TOKEN_GOES_HERE>" -d '{"login": "<YOUR_APP_USER_EMAIL>@<YOUR_COMPANY_DOMAIN>", "name": "<YOUR_APP_USER_NAME>"}' 
   ```
   {: pre}

1.  Copy the newly generated **id** field contents and provide them to the non-administrator connecting the Box application to {{site.data.keyword.discoveryfull}}.
1.  Return to the Box developer console `https://app.box.com/developers/console` and scroll to the **Add and Manage Public Keys** section. Click the **Generate the public/private keypair** and download your keypair file. Open the file.
1.  From the keypair file, cut and paste the following fields into the {{site.data.keyword.discoveryshort}} tooling.
    -  `client_id`
    -  `enterprise_id`
    -  `client_secret` 
    -  `public_key_id` 
    -  `private_key` 
    -  `passphrase` 

### Setting up Enterprise level access
{: #entlevelbox}

1.  Navigate to `https://app.box.com/developers/console` (use your company's Box URL) and click **Create New App**.
1.  From the **Create a New App** screen, select **Enterprise Integration** and click **Next**.
1.  On the **Authentication Method** screen, select **OAuth 2.0 with JWT (Server Authentication)** and click **Next**.
1.  Name your app and click the **Create App** button. Once your Box app has been created, click **View Your App**.
1.  While viewing your app, select **Application access** of **Application**. You can use your existing managed users as you continue; you are not required to create new ones.
1.  Scroll down to the **Application Scopes** section of the page and enable the following checkboxes: 
    - `Read and write all folders stored in Box`
    - `Manage Users`
1.  Scroll down to the **Advanced Features** section and enable the following toggles:
    - `Perform Actions as Users`
    - `Generate User Access Tokens` 
1.  Click the **Save Changes** button.

The next few steps will require assistance from the Administrator of your organization's Box account. If you are not your organization's Box Administrator, you can identify the Administrator by opening the Box developer's console and looking under **Account settings** > **Account details** > **Settings**.

1.  [Administrator step] Authorize your application client id at `https://app.box.com/master/settings/openbox` by clicking the **Authorize New App** button.
1.  [Administrator step] Type the **client ID** from `https://app.box.com/developers/console` into the **API key** field and then click the **Authorize** button.
1.  Return to the Box developer console `https://app.box.com/developers/console` and scroll to the **Add and Manage Public Keys** section. Click the **Generate the public/private keypair** and download your keypair file. Open the file.
1.  From the keypair file, cut and paste the following fields into the {{site.data.keyword.discoveryshort}} tooling.
    -  `client_id`
    -  `enterprise_id`
    -  `client_secret` 
    -  `public_key_id` 
    -  `private_key` 
    -  `passphrase` 

Other items to consider when crawling Box:

-  Box notes are stored in JSON format, so any Box notes in the specified folders will be ingested by {{site.data.keyword.discoveryshort}}.
-  When using the API, you will need to have a list of Folders IDs and the associated Owner ID for each folder that you want to crawl. The {{site.data.keyword.discoveryshort}} tooling lets you browse and select which content to crawl.

## Salesforce
{: #connectsf}

When connecting to a Salesforce source, ensure that the instance you plan to connect to is an Enterprise plan or higher.

The following credentials are required to connect to a Salesforce source, they should be obtained from your Salesforce administrator:
-  `url` - The `url` of the source that these credentials connect to.
-  `username` - The `username` of the source that these credentials connect to.
-  `password` - The `password` consists of the Salesforce password and a valid Salesforce security token concatenated. This value is never returned and is only used when creating or modifying credentials.

When identifying the credentials, it might be useful to consult the [Salesforce developer documentation ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://developer.salesforce.com/docs/){: new_window}.

Other items to note when crawling Salesforce:

-  Knowledge Articles are only crawled if their **version** is `published` and their languages is `en-us`.
-  When using the API, you will need to have a list of Salesforce objects that you want to crawl. The {{site.data.keyword.discoveryshort}} tooling lets you browse and select which content to crawl.


## SharePoint Online
{: #connectsp}

When connecting to a Microsoft SharePoint Online source, ensure that the instance you plan to connect to is an Enterprise (E1) plan or higher.

The following credentials are required to connect to a SharePoint Online source, they should be obtained from your SharePoint administrator:

-  `organization_url` - The `organization_url` of the source that these credentials connect to.
-  `site_collection.path` - The `site_collection.path` of the source that these credentials connect to.
-  `username` - The `client_id` of the source that these credentials connect to.
-  `password` - The `password` of the source that these credentials connect to. This value is never returned and is only used when creating or modifying credentials.

When identifying the credentials, it might be useful to consult the [Microsoft SharePoint developer documentation ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://docs.microsoft.com/en-us/sharepoint/dev/){: new_window}.

Other items to note when crawling Microsoft SharePoint Online:

-  When crawling SharePoint, you will need to have a list of SharePoint site collection paths that you want to crawl. The {{site.data.keyword.discoveryshort}} tooling lets you browse and select which content to crawl. To crawl your entire SharePoint Online site, do not select multiple paths (URLs) in this field. In that scenario, enter a `/` in the `site_collection.path` field.

## Web Crawl
{: #connectwebcrawl}

This beta feature can be used crawl public websites that don’t require a password. You can select how often you'd like {{site.data.keyword.discoveryshort}} to sync with the websites, the language, and the number of hops.

-  `Sync my data` - You can choose to sync daily, weekly, or monthly. Weekly syncs will occur every Sunday. Monthly syncs will occur on the first Sunday of every month.
-  `Select language` - Choose the language of the websites from the list of supported languages. See [Language support](/docs/services/discovery/language-support.html#supported-languages) for the list of languages supported by {{site.data.keyword.discoveryshort}}. It is recommended that you create a separate collection for each language.
-  `URL group to sync` - Enter your URL and click the plus sign to add it to the URL group. To specify the **Crawl settings** for this URL group, click the ![Cog](images/icon_settings.png) icon. You can set the **Maximum hops**, which is the number of consecutive links to follow from the starting URL (the starting URL is `0`). The default number of hops is `2` and the maximum is `20` (if using the API, the maximum is `1000`). 

For this beta release, the number of web pages crawled will be limited, so the web crawler may not crawl all the websites specified and reach the maximum number of hops.
{: note}

If you require different **Crawl settings** for other URLs, click **Add URL group** and create a new group. You can create as many URL groups as you need.
{: tip}

## Installing IBM Secure Gateway for on-premise data 
{: #gateway}

To connect to an on-premise data source, you first need to download, install, and configure IBM Secure Gateway. After you have installed IBM Secure Gateway for your first on-premise data source, you won't need to repeat this process.

1.  From the **Manage data** page of the {{site.data.keyword.discoveryshort}} tooling, select **Connect a data source**.
1.  Select the data source that you want to connect to. When you select an on-premise data source, go to the **Connect to your on-premise network** section and click the **Make connection** button.
1.  On the **Download and install the Secure Gateway Client** screen, download the appropriate version of IBM Secure Gateway.
1.  After you have completed the download, click the **Download Secure Gateway and Continue** button. During installation, you will need the **Gateway ID** and **Token** provided on the screen when prompted by the Secure Gateway Client. See [Installing the client ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://cloud.ibm.com/docs/services/SecureGateway/securegateway_install.html#installing-the-client){: new_window} in the Secure Gateway documentation for installation instructions.
1.  On the machine running the Secure Gateway Client, open the Secure Gateway dashboard at http://localhost:9003.
1.  Click **add ACL** on the dashboard. Add the endpoint URL of each SharePoint collection to the **Allow access** list. For example, Hostname: `mycompany.sharepoint.com` and port: `80`.
1.  Return to the {{site.data.keyword.discoveryshort}} tooling and click **Continue**. If the connection is successful you will see the `Connection successful` message. If the connection was not successful, open the Secure Gateway dashboard and verify that the endpoints on the **Allow access** list are correct.

After the connection is successful you can begin entering the credentials for your on-premise data source.

## SharePoint 2016 On-Premise
{: #connectsp_op}

Microsoft SharePoint 2016 (also known as SharePoint Server 2016) is an on-premise data source. To connect to it, you must first install and configure IBM Secure Gateway. See [Installing IBM Secure Gateway for on-premise data](/docs/services/discovery/connect.html#gateway) for more information.
{: note}

The following credentials are required to connect to a SharePoint 2016 data source, they should be obtained from your SharePoint administrator:

-  `username` - The `client_id` of the source that these credentials connect to.
-  `password` - The `password` of the source that these credentials connect to. This value is never returned and is only used when creating or modifying credentials.
-  `web_application_url` - The SharePoint 2016 `web_application_url`; for example, https://sharepointwebapp.com:8443. If the port is not supplied, it defaults to port `80` for http and port `443` for https.
-  `domain` - The `domain` of the SharePoint 2016 account.

When identifying the credentials, it might be useful to consult the [Microsoft SharePoint developer documentation ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://docs.microsoft.com/en-us/sharepoint/dev/){: new_window}.

Other items to note when crawling Microsoft SharePoint 2016 :

-  When crawling SharePoint 2016, you will need to have a list of SharePoint site collection paths that you want to crawl. The {{site.data.keyword.discoveryshort}} tooling lets you browse and select which content to crawl. To crawl your entire SharePoint 2016 site, do not select multiple paths (URLs) in this field. In that scenario, enter a `/` in the `site_collection.path` field.

## Using the tooling
{: #source_tooling}

Connecting to a data source using the {{site.data.keyword.discoveryshort}} tooling is performed by creating a collection specifically for a source. Perform the following steps to create a source collection and crawl it:

1.  From the **Manage data** page of the {{site.data.keyword.discoveryshort}} tooling, select **Connect a data source**.
2.  Select the data source that you want to connect to. If you have selected an on-premise data source, you will first need to install IBM Secure Gateway, see [Installing IBM Secure Gateway for on-premise data](/docs/services/discovery/connect.html#gateway) for more information. 
3.  Enter your source credentials and click connect. Your source credentials must be obtained from your source system administrator.
4.  Choose which data you want to the crawl and how often you want to sync it.
5.  Click **Save & Sync objects** to start crawling your data source. You are then redirected to the collection status screen which updates as documents are added to the collection.

The crawl will sync the data initially and then on frequency that you specified.
**Note:** If you modify anything in the **Sync settings** screen and then click **Save and Sync objects** a crawl will be started (or restarted if one is already running) at that time.

## Using the API
{: #source_api}

Use the following process to create a collection connected to a data source using the API.

If you plan to connect to an on-premise data source, you will first need to install IBM Secure Gateway, see [Installing IBM Secure Gateway for on-premise data](/docs/services/discovery/connect.html#gateway) for more information. 

1.  Create credentials for the source that you are connecting to using the [Source Credentials API ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}/apidocs/discovery#create-credentials){: new_window}. Record the returned **credential_id** of the newly created credentials.
2.  Create a new configuration using the [Configuration API ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}/apidocs/discovery#add-configuration){: new_window}. This configuration must contain a **source** object which defines what should be crawled. The **source** object must contain the **credential_id** that you recorded earlier.
    ```json
    "source" : {
      "type" : "salesforce",
      "credential_id" : "{credential_id}",
      "schedule" : {
        "enabled" : true,
        "time_zone" : "America/New_York",
        "frequency" : "weekly"
      },
      "options" : {
         "site_collections" : [ {
         "site_collection_path" : "/sites/TestSiteA",
         "limit" : 10
         } ]
      }
    }
    ```
    {: codeblock}
    Record the returned **configuration_id** of the newly created configuration.
3.  Create a new collection using the [Collections API ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://{DomainName}/apidocs/discovery#create-a-collection){: new_window}. The object defining the collection must contain the **configuration_id** that you recorded earlier.

The source crawl begins as soon as the collection is created, and then again on the frequency that you specified.
**Note:** If you modify anything in the **source** object of the configuration a new crawl will be started (or restarted if one is already running) at that time.

## Specifying a `customer_id`
{: #source_customer_id}

A `customer_id` field in an ingested {{site.data.keyword.discoveryshort}} document can be used to delete content based on the `customer_id` using the **user-data** method in the API. Incoming documents from a data source are not automatically assigned a `customer_id` when ingested. If your application requires a `customer_id` to be defined, you can specify one (or more) of the incoming fields from the source system to be copied and used as a `customer_id`. To do this you must modify the configuration that is being used to connect to the source.

1.  Perform a sample query and identify which field you want to use as a `customer_id`.
2.  Modify the configuration. You will need to add a **Normalization** section in order to create the `customer_id` field.
    -  In the tooling, navigate to your collection and click the **edit** link in the configuration section. Next, click the **Normalization** tab and add in a **copy** normalization to create the `customer_id` field. Then click **Apply & save**.
    ![Copy normalization](images/norm_copy.png)
    -  When using the API, add the following object to the **normalizations** array:
       ```json
       {
         "operation" : "copy",
         "source_field" : "{original_field_name}",
         "destination_field" : "customer_id"
       }
       ```
       {: codeblock}
3.  The next scheduled crawl will add the `customer_id` field to all documents. If you want to have a crawl start immediately, modify the source configuration (**Sync settings** in the tooling).

See [Information security ![External link icon](../../icons/launch-glyph.svg "External link icon")](https://cloud.ibm.com/docs/services/discovery/information-security.html){: new_window} for more information and information about deleting based on `customer_id`.
