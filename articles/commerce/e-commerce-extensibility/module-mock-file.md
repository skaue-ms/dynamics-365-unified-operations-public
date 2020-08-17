---
# required metadata

title: Module mock file
description: This topic covers the module mock file in Microsoft Dynamics 365 Commerce.
author: samjarawan
manager: annbe
ms.date: 01/31/2020
ms.topic: article
ms.prod: 
ms.service: dynamics-365-commerce
ms.technology: 

# optional metadata

# ms.search.form: 
audience: Developer
# ms.devlang: 
ms.reviewer: v-chgri
ms.search.scope: Retail, Core, Operations
# ms.tgt_pltfrm: 
ms.custom: 
ms.assetid: 
ms.search.region: Global
# ms.search.industry: 
ms.author: samjar
ms.search.validFrom: 2019-10-31
ms.dyn365.ops.version: Release 10.0.5

---
# Module mock file

[!include [banner](../includes/banner.md)]

This topic covers the module mock file in Microsoft Dynamics 365 Commerce.

## Overview

Module mock files are used to configure mock data for modules. Therefore, they can help you do local testing. The default module mock file is located at \\src\\MODULE\_NAME\\MOCKS\\MODULE\_NAME.json.

### Example of a mock file

Here is an example of a mock file.

```json
{
	"id": "R1Module1",
	"config": {
	    "imageAlignment": "left",
	    "productTitle": "Retro Horn Rimmed Keyhole Nose Bridge Round Sunglasses",
	    "productDetails": "High-quality and pioneered with the perfect blend of timeless classic and modern technology with hint of old school glamor.",
	    "productImage": {
		    "src": "https://bit.ly/33cMGxr",
		    "altText": "Retro Horn Rimmed Keyhole Nose Bridge Round Sunglasses"
	    },
	    "buttonText": "Buy Now",
	    "productIds": "68719498121"
	},
	"data": {
	    "actionResponse": {
		    "text": "Sample Action Response"
	    }
	},
	"typeName": "product-feature"
} 
```

Several configuration properties are used to set values for configuration fields that are set in the module definition file. When modules are run on a production server, this data comes from the Microsoft Dynamics content management system (CMS) database, as configured by the page authors, instead of a mock file.

## Render a module by using mock file data

To render a module by using data from a mock file, run the **yarn start** command, and then open the following URL in a web browser: `https://localhost:4000/modules?type=MODULE_NAME`.

## Add mock files

In addition to the default mock file, you can add multiple other mock files. In this way, you can view and test various scenarios. 

To add a mock file, create a new JavaScript Object Notation (JSON) file in the mocks folder, and give it a unique name (for example, **\\src\\MODULE\_NAME\\MOCKS\\MODULE\_NAME\_TEST.json**).

To render a module by using a different mock file, run the **yarn start** command, and then open the following URL in a web browser: `https://localhost:4000/modules?type=MODULE_NAME:MODULE_NAME_TEST`.

## Additional resources

[Module definition file](module-definition-file.md)

[Module React component file](module-react-file.md)

[Module view file](module-view-file.md)

[Module data file](module-data-file.md)

[Module test file](module-test-file.md)

[Module props.autogenerated.ts file](module-props-autogenerated-ts-file.md)