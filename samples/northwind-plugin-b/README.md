# Northwind Traders Type A Plugin

## Prerequisites

  * [Visual Studio Code](https://code.visualstudio.com/Download)
  * [NodeJS 18.x](https://nodejs.org/en/download)
  * [Teams Toolkit extension for VS Code](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension)
  * [Teams Toolkit CLI](https://learn.microsoft.com/en-us/microsoftteams/platform/toolkit/teams-toolkit-cli?pivots=version-three) (`npm install -g @microsoft/teamsapp-cli`)
  * (optional) [Postman](https://www.postman.com/downloads/)

## Setup instructions (one-time setup)


1. Copy the `env/.env.local.user.sample` file to a new file called `env/.env.local.user`

~~~text
SECRET_STORAGE_ACCOUNT_CONNECTION_STRING=UseDevelopmentStorage=true
~~~

1. Browse to https://dev.teams.microsoft.com/apps and log into the tenant where you plan to test. Create a new app and copy the App ID to the `id` property in **\appPackage\manifest.json**.

1. Log into Teams Toollkit using any tenant for now, as we will be uploading manually.

## Running the solution (after each build)

1. Press F5 to start the application. Eventually a browser window should open up; this is from the Teams Toolkit API Message Extension we used to start the project. Please minimize the browser window - i.e. leave it running, but don't use it.

   a. (optional) In the Terminal tab, find the "Start local tunnel" terminal and copy the forwarding URL to test the web service in Postman with a request such as https://xxxxxxxx-7071.xxx.devtunnels.ms/api/products/ (using your forwarding URL)

1. Log into the target tenant with Teams Toolkit CLI:

    `teamsapp account login m365`

1. Upload the package using the Teams Toolkit CLI:

   `teamsapp m365 sideloading -file-path ./pluginPackage-Local/build/pluginPackage.zip`

1. Go to the Copilot app in Teams and enable your plugin in the plugin panel.

1. Try some of the sample prompts. Use `-developer on` and view the application log to try and understand what's going on. The application log can be viewed under the Debug Console tab by selecting "Attach to Backend" from the dropdown on the top right of the debug console window.


## Sample prompts
- Find product Chai from Northwind Traders
- Find product Chai from Northwind Traders Inventory Management
- Get supplier of product Chai from Northwind Inventory Management
- What is the price of Protein Shake in Northwind Inventory Management?
- For the product Chai who is the supplier?
- What are the beverages available in Northwind Traders Management System
- Find Condiments available in Northwind Traders Management System
- Write me a one paragraph summary detailing the number and types of condiments that are supplied by New Orleans Cajun Delights

## API Documentation

### Methods 
- GET: Retrieve information about products or a specific product using its name
- POST: Add a product or update an existing product's information. 

### Headers 
- Content-Type: application/json
- Authorization: Token  (if required) 

### Operations 

#### PRODUCTS

##### GET /products 

- Retrieves a list of all products.
- Optional Query Parameters:

  - productName: Filters products by product name, which may be partial
  - categoryName: Filters products by category name, which may be partial
  - supplierName: Filters products by supplier name, which may be partial
  - inventoryStatus: Filters products by inventory status where possible values are:
    - out of stock
    - low on stock
    - on order
    - in stock
  - supplierCity: Filters products by supplier city, which may be partial
  - inventoryRange: Filters products by the inventory (stock) level as a range in the form 0-300 to obtain products with 300 or less in stock, or 400- to obtain products with more than 400 in stock
  - discontinued: Filters products that have been discontinued (true or false)
  - revenueRange: Filters by the revenue during the last period in the form 0-100 or 50000-. Keywords “high” and “low” are also accepted by this query string

- Success Response:
  - Code: 200 OK
  - Content: JSON array of product objects.

##### GET products/{productName}

- Retrieves a single product with a specific name or partial name in the last path segment. If multiple products match the productName, only the first is returned.

- Success Response:
  - Code: 200 OK
  - Content: JSON array containing the first product that matches the specified name
 
##### GET /products/{productId}
 
- Retrieves detailed information about a specific product by ID.
- URL Parameters:
  - productId: The unique identifier for the product.
- Success Response:
  - Code: 200 OK
  - Content: JSON array containing the product
 
##### POST /products/{ProductID}
 
- Updates an existing product.
- Request Body:

  - JSON representation of the product object (as defined below). The product ID in the URL must match the product ID in the product object.

- Success Responses:
  - Code: 200 OK (for updates)
  - Content: JSON object of the created/updated product.

### Product Object

This is the product object; an array of these is returned in GET requests and a single instance is expected in the body of a POST request.

~~~typescript
 interface IProduct {
    // Mapped directly to database fields:
    productId: number;
    productName: string;
    supplierId: number;
    categoryId: number;
    quantityPerUnit: string;
    unitPrice: number;
    unitsInStock: number;
    unitsOnOrder: number;
    reorderLevel: number;
    discontinued: boolean;
    imageUrl: string;
    // Denormalized during augmentation
    categoryName: string,
    supplierName: string,
    supplierCity: string,
    // Generated from order details during augmentation
    inventoryStatus: string,
    valueOfInventory: number,
    unitSales: number,
    revenue: number,
    averageDiscount: number
}
~~~

<!--

# Overview of Custom Search Results app template

## Build a message extension from a new API with Azure Functions

This app template allows Teams to interact directly with third-party data, apps, and services, enhancing its capabilities and broadening its range of capabilities. It allows Teams to:

- Retrieve real-time information, for example, latest news coverage on a product launch.
- Retrieve knowledge-based information, for example, my team’s design files in Figma.

## Get started with the template

> **Prerequisites**
>
> To run this app template in your local dev machine, you will need:
>
> - [Node.js](https://nodejs.org/), supported versions: 16, 18
> - A [Microsoft 365 account for development](https://docs.microsoft.com/microsoftteams/platform/toolkit/accounts)
> - [Teams Toolkit Visual Studio Code Extension](https://aka.ms/teams-toolkit) version 5.0.0 and higher or [Teams Toolkit CLI](https://aka.ms/teamsfx-cli)

1. First, select the Teams Toolkit icon on the left in the VS Code toolbar.
2. In the Account section, sign in with your [Microsoft 365 account](https://docs.microsoft.com/microsoftteams/platform/toolkit/accounts) if you haven't already.
3. Select `Debug in Teams (Edge)` or `Debug in Teams (Chrome)` from the launch configuration dropdown.
4. When Teams launches in the browser, you can navigate to a chat message and [trigger your search commands from compose message area](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions?tabs=dotnet#search-commands).

## What's included in the template

| Folder       | Contents                                                                                                    |
| ------------ | ----------------------------------------------------------------------------------------------------------- |
| `.vscode`    | VSCode files for debugging                                                                                  |
| `appPackage` | Templates for the Teams application manifest, the API specification and response template for API responses |
| `env`        | Environment files                                                                                           |
| `infra`      | Templates for provisioning Azure resources                                                                  |
| `repair`     | The source code for the repair API                                                                          |

The following files can be customized and demonstrate an example implementation to get you started.

| File                                          | Contents                                                                     |
| --------------------------------------------- | ---------------------------------------------------------------------------- |
| `repair/function.json`                        | A configuration file that defines the function’s trigger and other settings. |
| `repair/index.ts`                             | The main file of a function in Azure Functions.                              |
| `appPackage/apiSpecificationFiles/repair.yml` | A file that describes the structure and behavior of the repair API.          |
| `appPackage/responseTemplates/repair.json`    | A generated Adaptive Card that used to render API response.                  |
| `repairsData.json`                            | The data source for the repair API                                           |

The following are Teams Toolkit specific project files. You can [visit a complete guide on Github](https://github.com/OfficeDev/TeamsFx/wiki/Teams-Toolkit-Visual-Studio-Code-v5-Guide#overview) to understand how Teams Toolkit works.

| File                 | Contents                                                                                                                                  |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `teamsapp.yml`       | This is the main Teams Toolkit project file. The project file defines two primary things: Properties and configuration Stage definitions. |
| `teamsapp.local.yml` | This overrides `teamsapp.yml` with actions that enable local execution and debugging.                                                     |

## Addition information and references

- [Extend Teams platform with APIs](https://aka.ms/teamsfx-api-plugin)

-->