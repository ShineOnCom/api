# Topics

- [API Integrations (admin notes)](https://github.com/ShineOnCom/integrations/blob/master/README.md#api-integrations-admin-notes)
- [Orders API](https://github.com/ShineOnCom/integrations/blob/master/README.md#orders-api)
- [Skus API](https://github.com/ShineOnCom/integrations/blob/master/README.md#skus-api)
- [Artwork API](https://github.com/ShineOnCom/integrations/blob/master/README.md#artwork-api)
- [ProductTemplates API](https://github.com/ShineOnCom/integrations/blob/master/README.md#producttemplates-api)
- [Transformations API](https://github.com/ShineOnCom/integrations/blob/master/README.md#transformations-api)

# API Integrations (admin notes)

- API Affiliate Admin account necessary for all ShineOn Partners
- API access and usage instructions
- Viewing product templates w/associative transformations
- Viewing orders that have synchronized
- Access to our CS tools
- Viewing payment due




# Orders API

## Example Endpoints

> Here is a description of endpoints ShineOn can leverage to synchronize your orders and send back tracking information.

### GET https://shineon-partner-domain.com/api/orders

#### GET `Orders` REQUEST (multiple)

... options

#### GET `Orders` __RESPONSE__ (multiple)

> Sample of a complete Orders Response.

```
{
  "orders": [
    {
     "id": 450789469,
      "source_id": "fhwdgads",
      "source_url": null,
      "email": "bob.norman@hostmail.com",
      "number": 1,
      "note": null,
      "test": false,
      "total_price": "409.94",
      "subtotal_price": "398.00",
      "total_weight": 0,
      "total_tax": "11.94",
      "currency": "USD",
      "name": "#1001",
      "reference": "fhwdgads",
      "phone": "+557734881234",
      "order_number": 1001,
      "created_at": "2008-01-10T11:00:00-05:00",
      "updated_at": "2008-01-10T11:00:00-05:00",
      "cancelled_at": null,
      "cancel_reason": null,
      "line_items": [
        {
          "id": 518995019,
          "variant_id": 49148385,
          "title": "IPod Nano - 8gb",
          "quantity": 1,
          "base_price": "19.00",
          "price": "199.00",
          "currency_code": "USD",
          "sku": "IPOD2008RED",
          "variant_title": "red",
          "product_id": 632910392,
          "name": "IPod Nano - 8gb - red",
          "properties": [],
          "grams": 200,
          "fulfillment_id": null
        }
      ],
      "billing_address": {
        "first_name": "Bob",
        "address1": "Chestnut Street 92",
        "phone": "555-625-1199",
        "city": "Louisville",
        "zip": "40202",
        "province": "Kentucky",
        "country": "United States",
        "last_name": "Norman",
        "address2": "",
        "company": null,
        "name": "Bob Norman",
        "country_code": "US",
        "province_code": "KY"
      },
      "shipping_address": {
        "first_name": "Bob",
        "address1": "Chestnut Street 92",
        "phone": "555-625-1199",
        "city": "Louisville",
        "zip": "40202",
        "province": "Kentucky",
        "country": "United States",
        "last_name": "Norman",
        "address2": "",
        "company": null,
        "name": "Bob Norman",
        "country_code": "US",
        "province_code": "KY"
      },
      "fulfillments": [
        {
          "id": 255858046,
          "created_at": "2018-11-27T12:38:21-05:00",
          "updated_at": "2018-11-27T12:38:21-05:00",
          "tracking_company": null,
          "tracking_number": "1Z2345",
          "tracking_url": "http://wwwapps.ups.com/etracking/tracking.cgi?InquiryNumber1=1Z2345&TypeOfInquiryNumber=T&AcceptUPSLicenseAgreement=yes&submit=Track",
          "line_items": [
            {
              "id": 466157049,
              "quantity": 1,
            }
          ]
        }
      ]
    }
  ]
}
```

### GET https://shineon-partner-domain.com/api/orders/{order_id}

> Fetching a single order (with your id) is also useful for us for adhoc, debugging, and informational requests.

```
// no query string
```

#### GET `Orders` __RESPONSE__

```
{
    // your orders array response, each with line items with `external_sku_id` property or equivalent
}
```



---




# Skus API

An sku object identfies the artwork captured when the seller created the design, and the product (variant) template used. Skus should not be generated until the artwork is FINAL (this means sending a cropped version of the desired FINAL artwork).

## Endpoints (draft) for `Sku` CRUD and optional rendering

### POST: 	https://api.shineon.com/v1/skus
### PUT:  	https://api.shineon.com/v1/skus/{sku_id}
### GET:  	https://api.shineon.com/v1/skus/{sku_id}
### DELETE: 	https://api.shineon.com/v1/skus/{sku_id}
and
### GET:    	https://api.shineon.com/v1/skus

## Examples

#### POST `Sku` REQUEST:

> If you already have an artwork_id, then transport will occur much faster by specifying it instead of artwork data. However, it may make sense to pass your artwork along when you create your Sku, creating both an `Artwork` object and a `Sku` object in one request. Furthermore, you may optionally specify a transformation_id to receive back a `Artwork` object, `Sku` object and a `transformation` (with render) all with one request.

```
{
    sku: {
        product_template_id: int required,
        (artwork_id|artwork|artwork_src): (int|string), either artwork_id or artwork (png or jpg) is required
        external_id: int or string required (gearbubble variant or campaign id),
        transformation_id: int optional 		// This is really a bonus if you want to accomplish your task with 1 API request.
    }
}
```

#### PUT `Sku` REQUEST:

> Must include sku id, may only update artwork (optional), and external_id

```
{
    sku: {
        id: int required,
        artwork: string optional,
        external_id: (int|string) optional
    }
}
```

#### GET `Sku` REQUEST: 

> Must include sku id only

```
// no payload
```

#### GET|PUT|POST `Sku` __RESPONSE__:

- SKU belongs to ProductTemplate(variant)
- SKU belongs to Artwork
- ProductTemplate has many Transformations
- ProductTemplate has many siblings (ProductTemplate)

```
{
    sku: {
        id: int,
        external_id: string,
        product_template: {
            id: int, 				// What is the variant id?
            siblings: [				// What are the compatible sibling variant ids?
                int, int, int
            ],
            engraving_variant_id: int (associative engraving variant id, can be self) | null (not available)
            buyer_uploads: boolean,
            base_cost: decimal (USD),
            artwork_mask_src: cdn-url.    	// What is the public cdn url for the white mask applied to artwork to show the printable area? 
            title: string, 			// What is the suggested variant title?
            position: int, 			// What is the suggested position with respect to the other sibling variants for this product?
            optimized_height: int,  		// What height the artwork will be constrained?
            optimized_width: int, 		// What width the artwork will be constained?
            upload_min_height: int, 		// What is the suggested min height (not enforced)
            upload_min_width: int,  		// What is the suggested min width (not enforced)
            metafields: {
                type: string,
                shape: string,
                metal: string,
                engravings: int,
            },
            transformations: [
	    	{
                    id: int,
                    label: string, 		// A short descriptor
                    renders: boolean, 		// Is it a render or just a plain image?
                    default: boolean, 		// Is this the considered a standard choice for the variant?
                    position: int, 		// Suggested position with respect to the other transformations for the product
                    dark_available: boolean, 
                    output_format: string, 	// What is the default output format?
                    output_quality: integer, 	// What is the default output quality?
                    published_at: ISO 8601 date,
                    created_at: ISO 8601 date,
                    updated_at: ISO 8601 date
		}
            ],
            published_at: ISO 8601 date
        },
        artwork: {
            id: int,
            src: 24hr-signed-s3-cdn-url, 	// This is signed to protect seller IP. You can store on your end if that makes sense.
            src_mimetype: string,
            src_bytes: string,
            original_src: 24hr-signed-s3-cdn-url, // This is signed to protect seller IP. You can store on your end if that makes sense.
            original_src_mimetype: string,
            original_src_bytes: int,
            alpha: boolean, // Were any alpha (transparent pixels) detected?
            created_at: ISO 8601 date,
            updated_at: ISO 8601 date
        },
        render: {
	    src: (string|null) base64_encoded optimized jpg,
	    src_mimetype: '(image/png|image/png)'
	    src_bytes: int,
	    transformation: {
	        product_template: {
		   ...
		},
	        ...
	    }
	},
        created_at: ISO 8601 date,
        updated_at: ISO 8601 date
    }
}
```

#### DELETE `Sku` REQUEST: 

```
// no query string
```

#### DELETE `Sku` __RESPONSE__:

> Will return 404 if Sku was already deleted.
> Will return 405 if Sku has previously sold.

```
{
    sku: {
        id: int
    }
}
```

#### GET `Sku` REQUEST (multiple): 

> No query string is also acceptable

```
?page=1&per_page=250
```

#### GET `Sku` __RESPONSE__ (multiple):

The `skus` array will be returned with the product_template_id and artwork_id instead of objects.

> Base

```
{
    skus: [
        {
	    // see above for example
        }
    ]
}
```

Params accepted: 

- `page` (default: 1), 
- `per_page` (default:15, max:250), 
- `product_template_id` (default: null), 
- `created_at_min` (default: null), 
- `updated_at_min` (default: null)




---




# Artwork API

## Endpoints (draft) for `Artwork` CRUD and optional rendering

### POST: 	https://api.shineon.com/v1/artwork
### PUT: 	https://api.shineon.com/v1/artwork/{artwork_id}
### GET: 	https://api.shineon.com/v1/artwork/{artwork_id}
### DELETE: 	https://api.shineon.com/v1/artwork/{artwork_id}

## Examples

#### POST `Artwork` REQUEST:

> The purpose of this endpoint is to generate an artwork object. Artwork supplied will automatically be constrained to the optimized dimensions specified by the respective product template. An artwork object should only be created when the seller is satisfied with how their design will look when printed.


```
{
    (artwork|artwork_src): base64encoded-png-or-jpg,
    product_template_id: int required,
    transformation_id: optional
}
```

#### PUT `Artwork` REQUEST:

> Artwork may be amended.

```
{
    artwork: base64encoded-png-or-jpg
}
```

#### GET `Artwork` REQUEST:

> Or get a single artwork object. Query string optional.

```
?base64=1
```

#### GET|PUT|POST `Artwork` __RESPONSE__:

> When true, the `base64` option will return a `*_src` with base64 encoded data.

```
{  
    artwork: {
        id: int,
        src: 24hr-signed-s3-cdn-url, 		// This is signed to protect seller IP. You can store on your end if that makes sense.
        src_mimetype: string,
        src_bytes: string,
        original_src: 24hr-signed-s3-cdn-url, 	// This is signed to protect seller IP. You can store on your end if that makes sense.
        original_src_mimetype: string,
        original_src_bytes: int,
        alpha: boolean, 			// Were any alpha (transparent pixels) detected?
        render: {
	    src: (string|null) base64_encoded optimized jpg,
	    src_mimetype: '(image/png|image/png)'
	    src_bytes: int,
	    transformation: {
	        product_template: {
		   ...
		},
	        ...
	    }
	},
        created_at: ISO 8601 date,
        updated_at: ISO 8601 date
    }
}
```

#### DELETE `Artwork` REQUEST: 

```
// no query string
```

#### DELETE `Artwork` __RESPONSE__:

> Will return 404 if Artwork was already deleted.

> Will return 405 if Artwork has previously been sold.

> Will return 406 if Artwork has Skus that still exist.

```
{
    sku: {
        id: int
    }
}
```




---




# ProductTemplates API

Contextually, it should be understood that `ProductTemplate` resources are always for a __variant__!

## Endpoints (draft) for Product Template data

### GET: 	https://api.shineon.com/v1/product_templates/{product_template_id}
### GET: 	https://api.shineon.com/v1/product_templates

## Examples

#### GET `ProductTemplate` REQUEST:

```
// no query string necessary
```

#### GET `ProductTemplate` __RESPONSE__: 

```
{
    product_template: {
        id: int, 			// What is the variant id?
        siblings: [			// What are the compatible sibling variant ids?
            int, int, int
        ],
        engraving_variant_id: int (associative engraving variant id, can be self) | null (not available)
        buyer_uploads: boolean,
        base_cost: decimal (USD),
        artwork_mask_src: cdn-url.    	// What is the public cdn url for the white mask applied to artwork to show the printable area? 
        title: string, 			// What is the suggested variant title?
        position: int, 			// What is the suggested position with respect to the other sibling variants for this product?
        optimized_height: int,  	// What height the artwork will be constrained?
        optimized_width: int, 		// What width the artwork will be constained?
        upload_min_height: int, 	// What is the suggested min height (not enforced)
        upload_min_width: int,  	// What is the suggested min width (not enforced)
            metafields: {
            type: string,
            shape: string,
            metal: string,
            engravings: int,
        },
        transformations: [
            {
                id: int,
                label: string, 		// A short descriptor
                renders: boolean, 	// Is it a render or just a plain image?
                default: boolean, 	// Is this the considered a standard choice for the variant?
                position: int, 		// Suggested position with respect to the other transformations for the product
                dark_available: boolean, 
                output_format: string, 	// What is the default output format?
                output_quality: integer, // What is the default output quality?
                published_at: ISO 8601 date,
                created_at: ISO 8601 date,
                updated_at: ISO 8601 date
            }
        ],
        published_at: ISO 8601 date
    }
}
```

#### GET `ProductTemplate` REQUEST (multiple):

```
// no query string necessary
```

#### GET `ProductTemplate` __RESPONSE__ (multiple): 

```
{
    product_templates: [
    	{
    		// see above ...
	}
    ]
}
```

# Transformations API

Wanting to retrieve even more incredible images? 

The `Skus`, `Artwork` and `ProductTemplate` APIs are somewhat the MVP for building skus, remitting artwork and fetching a single render for your sku when you do so. 

We intend on making additional endpoints explicitly for fetching additional jewelry renders for your variants. If this is of interest, please let us know more about the nature of your implementation, [dan@shineon.com](mailto:dan@shineon.com).
