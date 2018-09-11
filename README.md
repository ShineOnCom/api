# ShineOn API Integrations (admin notes)

- API Affiliate Admin account necessary for all ShineOn Partners
- API access and usage instructions
- Viewing product templates w/associative transformations
- Viewing orders that have synchronized
- Access to our CS tools
- Viewing payment due

# Skus API

## Endpoints (draft) for `Sku` CRUD and optional rendering

### POST: 	https://api.shineon.com/v1/skus
### PUT:  	https://api.shineon.com/v1/skus/{sku_id}
### GET:  	https://api.shineon.com/v1/skus/{sku_id}
### DELETE: https://api.shineon.com/v1/skus/{sku_id}
### GET:    https://api.shineon.com/v1/skus not described below, `skus` array returned w/product_template_id and artwork_id instead of object, accepts `page` (default: 1), `per_page` (default:15, max:250), `product_template_id` (default: null), `created_at_min` (default: null), `updated_at_min` (default: null)

> An sku object identfies the artwork captured when the seller created the design, and the product (variant) template used. Skus should not be generated until the artwork is FINAL (this means sending a cropped version of the desired FINAL artwork).

#### POST `Sku` REQUEST:

> If you already have an artwork_id, then transport will occur much faster by specifying it instead of artwork data. However, it may make sense to pass your artwork along when you create your Sku, creating both an artwork object and a sku object in one request. Furthermore, you may optionally specify a transformation_id to receive back a Sku object, Artwork object and a render all with one request.

```
{
    sku: {
        product_template_id: int required,
        (artwork_id|artwork): (int|string), either artwork_id or artwork (png or jpg) is required
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

## GET `Sku` REQUEST: 

> Must include sku id only

```
{
    sku: {
        id: int required
    }
}
```

## DELETE `Sku` REQUEST: 

> Must include sku id only

```
{
    sku: {
        id: int required
    }
}
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
        transformation_src: (string|null) base64_encoded optimized jpg,
        created_at: ISO 8601 date,
        updated_at: ISO 8601 date
    }
}
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




---




# Artwork API

## Endpoints (draft) for `Artwork` CRUD and optional rendering

### POST: 	https://api.shineon.com/v1/artwork
### PUT: 	https://api.shineon.com/v1/artwork/{artwork_id}
### GET: 	https://api.shineon.com/v1/artwork/{artwork_id}
### DELETE: 	https://api.shineon.com/v1/artwork/{artwork_id}

#### POST `Artwork` REQUEST:

> The purpose of this endpoint is to generate an artwork object. Artwork supplied will automatically be constrained to the optimized dimensions specified by the respective product template. An artwork object should only be created when the seller is satisfied with how their design will look when printed.


```
{
    artwork: base64encoded-png-or-jpg,
    product_template_id: int required,
    transformation_id: optional
}
```

#### PUT `Artwork` REQUEST:

> Artwork may be amended.

```
{
    artwork_id: base64encoded-png-or-jpg,
    artwork: string required
}
```

#### GET `Artwork` REQUEST:

> Or fetched.

```
{
    artwork_id: int required,
    base64: boolean optional, default false (cdn url)
}
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
        transformation_src: (string|null) base64_encoded optimized jpg,
        created_at: ISO 8601 date,
        updated_at: ISO 8601 date
    }
}
```

#### DELETE `Artwork` __RESPONSE__:

> Will return 404 if Artwork was already deleted.

> Will return 405 if Artwork has previously sold.

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

## Endpoints (draft) for Product Template data

> Contextually, it should be understood that `ProductTemplate` resources are always for a __variant__!

### GET: 	https://api.shineon.com/v1/product_templates/{id}

#### GET `ProductTemplate` REQUEST:

```
{
    product_template_id: int
}
```

#### GET `ProductTemplate` RESPONSE: 

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

### GET: 	https://api.shineon.com/v1/product_templates

#### GET `ProductTemplate` REQUEST (multiple):

```
// no payload
```

#### GET `ProductTemplate` RESPONSE (multiple): 

```
{
    product_templates: [
    	{
    		// see above ...
	}
    ]
}
```




---




# Order Synchronization / Fulfillment Workflow

> The srder synchronization docs describes proposed path towards ShineOn automatically synchronizing orders with our fulfillment system.

- ShineOn receives api token from partner
- ShineOn makes hourly GET requests on partner's API Orders resource
- ShineOn synchronizes any new jewelry orders from partner
- Internally at ShineOn
    - Invoices are generated for new orders
    - Notifications are sent for any problematic orders
    - Work orders are generated for invoiced orders
    - Jewelry is manufacturere and shipped.
- ShineOn makes daily fulfillment POST requests on partner's API Fulfillments resource (tracking numbers are sent)

## Example Endpoints

> Here is a description of endpoints ShineOn can leverage to synchronize your orders and send back tracking information.

### GET https://shineon-partner-domain.com/api/orders

```
{
    created_at_min: '2018-09-10 08:00:00' // e.g. the last time we synced
    partner: 'ShineOn'
}
```

`created_at_min` param or equivalent is necessary.
`partner` param or equivalent for filtering only jewelry orders is suggested.




