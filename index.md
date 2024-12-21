
# Description of the Web Service CHYTRÁ DISTRIBUCE  
### Version 1.1 (April 26, 2024)  

**Purpose:** The API service is designed for processing CHYTRÁ DISTRIBUCE orders.  
**Author:** Karel Větrovský (it@chytradistribuce.cz)  
**URL Endpoint:**  
`https://ws.chytradistribuce.cz`  

Include the token obtained from the profile in the ordering system in the `Authorization` header of your request.  

---

## Submitting Orders (Returns)  
**URL Structure:**  
`https://ws.chytradistribuce.cz/api/objednavka`  
**Method:** POST  

**Request for Creating an Order**  
```json
{
  "order_number (string)": {
    "id": "*Customer ID",
    "nazev": "*Customer Name / Company Name (string 100)",
    "dobirka": "Cash on delivery amount (float)",
    "varsym": "Variable symbol (string 10)",
    "ulice": "*Street + house number (string 39)",
    "psc": "*Postal code without spaces (string 5)",
    "obec": "*City (string 29)",
    "osoba": "Contact person (string 37)", 
    "email": "Contact email (string 100)",
    "telefon": "Contact phone (string 37)",
    "poznamka": "Courier note (string 60)",
    "vaha": "Package weight in kg (default is 1 kg)",
    "c_baliku": "Parcel sequence number in multi-package (if omitted, auto-numbered; if provided, all parcels in multi-package must be included individually) (int)",            
    "poc_bal": "Total number of parcels in multi-package (default is 1) (int)",       
    "vratka": "0 = standard package (default), 1 = return package (int)",
    "rating": "Empty string or 'VIP' (string 3)",
    "vlastni_kod": "**Custom barcode from assigned series (11 digits)",
    "chlazene": "0 = non-cooled (default), 1 = cooled (int)",
    "otv_doba": "Availability time range on delivery day (e.g., 'Tue: 08:00-12:00, 13:00-14:30') (string 200)"  
  }
}
```  

Fields marked in * are mandatory.  
Fields marked in ** are mandatory if you have an assigned numerical range. For orders with multiple packages, include the appropriate number of codes separated by commas (see example).  
The order number (accepts string) is a unique string within a batch and is used only as a reference in the response.  

---

## Response for Order Creation  
**Description of the Response:**  
```json
{
  "order_number": {
    "trasa": "Route number",
    "barcode": ["List of order barcodes", ...],
    "status": "OK - successfully added to the roster, ERROR - missing mandatory field or postal code not found"
  }
}
```  

For orders distributed by GLS to Slovakia, the response includes additional parameters such as depot number, hub sorting, and driver details.  

---

## Example Usage  
**Request Example:**  
Creating three packages for two customers. Order "1" uses all possible parameters, and order "2" uses only mandatory fields.  
```json
{
  "1": {
    "id": 100,
    "nazev": "Jan Novák",
    "dobirka": 100.1,
    "varsym": "1234567890",
    "ulice": "Leitnerova 32",
    "psc": "60200",
    "obec": "Brno",
    "rating": "VIP",
    "vlastni_kod": "10020030040,10020030041",
    "email": "test@chytradistribuce.cz",
    "telefon": "+420730145678",
    "poznamka": "Test order",
    "vaha": 10.77,
    "poc_bal": 2,
    "otv_doba": "Wed: 07:00-10:00,12:00-14:00"
  },
  "2": {
    "id": 124,
    "nazev": "Exotic Garden",
    "ulice": "Přibice 500",
    "psc": "69124",
    "obec": "Přibice"
  }
}
```  

**Response Example (Success):**  
```json
[
  {
    "1": {
      "trasa": "1297",
      "barcodes": ["60000000215", "60000000216"],
      "status": "OK"
    }
  },
  {
    "2": {
      "trasa": "2297",
      "barcodes": ["60000000217"],
      "status": "OK"
    }
  }
]
```  

**Response Example (Error):**  
```json
[
  {
    "1": {
      "status": "error",
      "desc": "missing mandatory data"
    }
  },
  {
    "2": {
      "status": "error",
      "desc": "missing mandatory data"
    }
  }
]
```  

---

## Example Using CURL  
```bash
curl -d '{"1": {"id": 124,"nazev": "Exotic Garden","psc": "69124","obec": "Přibice", "ulice": "Přibice 500"}}' -H 'Accept: application/json' -H "Authorization: TOKEN" https://ws.chytradistribuce.cz/api/objednavka
```  

---

## Cancelling an Order (Return)  
**URL Structure:**  
`https://ws.chytradistribuce.cz/api/objednavka/{parcel_code}`  
**Method:** DELETE  

`{parcel_code}` is an 11-digit package number assigned by the system.  
Packages can be canceled until the delivery protocol is generated. After that, requests must be submitted to the CHYTRÁ DISTRIBUCE dispatch center.  

**Response for Order Cancellation:**  
```json
[
  {
    "parcel_code": { 
      "status": "OK - successfully canceled, error - error with a short description in the desc field",
      "desc": "error description"
    }
  }
]
```


## Checking the Latest Status of an Order (Returns)  
**URL Structure:**  
`https://ws.chytradistribuce.cz/api/objednavka/{parcel_code,...}`  
**Method:** GET  

`{parcel_code}` is an 11-digit package number assigned by the system. Multiple codes can be submitted, separated by commas. If the code is not found, the status will be `null`.  

**Response for Checking Order Status:**  
```json
[
  {
    "parcel_code": { 
      "status": "Parcel status code",
      "desc": "Status description",
      "updated_at": "Date and time of the last update"
    }
  }
]
```

**Response Example:**  
```json
[
  {
    "60000000001": { 
      "status": 400,
      "desc": "Successfully delivered",
      "updated_at": "2023-12-08T15:56:25+01:00"
    }
  }
]
```

---

## Downloading Labels  

### Download All Created Labels:  
`https://moje.chytradistribuce.cz/stitky.php?token=TOKEN`  

### Download Only New (Unprinted) Labels:  
`https://moje.chytradistribuce.cz/stitky.php?newonly&token=TOKEN`  

### Download Label for a Specific Customer by ID:  
`https://moje.chytradistribuce.cz/stitky.php?zprint=ID&token=TOKEN`  

### Download Label for a Specific Parcel (Barcode):  
`https://moje.chytradistribuce.cz/stitky.php?bprint=BARCODE&token=TOKEN`  

### Download Delivery Protocol:  
`https://moje.chytradistribuce.cz/soupiska.php?op=protokol&token=TOKEN`  

---

## Downloading the Customer List  
**Export all customers, including routes:**  

**URL Structure:**  
`https://ws.chytradistribuce.cz/api/zakaznici`  
**Method:** GET  

**Response Example:**  
```json
[
  {
    "cislo": "2",
    "trasa": "220",
    "nazev": "Test2",
    "ulice": "Testová 32/2",
    "mesto": "Brno",
    "psc": "60201",
    "telefon": "730104398",
    "email": "vet@email.cz",
    "osoba": "",
    "poznamka": "test2",
    "poznamka_admin": null,
    "rating": "",
    "ind_cislo": null
  }
]
```

---

## Downloading Postal Codes  
**Functionality:** Used for initial route assignments based on postal codes.  

**URL Structure:**  
`https://ws.chytradistribuce.cz/api/psc`  
**Method:** GET  

**Response Example:**  
```json
[
  {
    "psc": 37371,
    "posta": "Rudolfov",
    "obec": "Adamov",
    "trasa": 400
  },
  {
    "psc": 37341,
    "posta": "Hluboká nad Vltavou",
    "obec": "Bavorovice",
    "trasa": 420
  }
]
```

---

## Obtaining Track & Trace Information for an Order  
The order status can be retrieved as an HTML page from the following URL format:  

`https://moje.chytradistribuce.cz/statistika.php?op=showbarcode&token=TOKEN&barcode=CODE`  

Where:  
- `TOKEN` is the authorization code obtained from the profile in the ordering system.  
- `CODE` is the 11-digit parcel number assigned to the shipment.  

---

End of Document

