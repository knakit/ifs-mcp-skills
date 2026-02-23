# IFS Customer Handling API Guide

Use the `call_protected_api` tool with the endpoints below to create, query, and manage customers and their addresses in IFS Cloud.

---

## Projection

All endpoints are under:
```
/main/ifsapplications/projection/v1/CustomerHandling.svc
```

---

## Create a Customer

**Endpoint:** `POST /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet`

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `Name` | string | Customer name |
| `CreationDate` | date (YYYY-MM-DD) | Today's date or specified date |
| `Country` | string | ISO 2-letter country code e.g. `"SE"`, `"GB"`, `"US"` |
| `CountryCode` | string | Same as Country |
| `DefaultLanguage` | string | ISO language code e.g. `"en"`, `"sv"` |
| `PartyType` | string | Always `"Customer"` |
| `DefaultDomain` | boolean | Always `true` |

**Optional fields:**

| Field | Type | Notes |
|-------|------|-------|
| `CustomerCategory` | string | `"Customer"` (default) or `"Prospect"` |
| `OneTime` | boolean | `false` (default) — set `true` for one-time customers |
| `B2bCustomer` | boolean | `false` (default) |
| `ValidDataProcessingPurpose` | boolean | `false` (default) |
| `IdentifierRefValidation` | string | `"None"` (default) |
| `AssociationNo` | string | Organisation/VAT registration number (digits only, no hyphens) |
| `CorporateForm` | string | Legal form of the company |
| `MainRepresentative` | string | Sales representative ID |
| `CustomerTaxUsageType` | string | Tax usage type code |
| `DateOfRegistration` | date | Registration date (YYYY-MM-DD) |
| `BusinessClassification` | string | Business classification code |

**Example request body:**
```json
{
  "Name": "Acme Corporation",
  "CreationDate": "2026-02-21",
  "Country": "SE",
  "CountryCode": "SE",
  "DefaultLanguage": "en",
  "PartyType": "Customer",
  "DefaultDomain": true,
  "CustomerCategory": "Customer",
  "OneTime": false,
  "B2bCustomer": false,
  "ValidDataProcessingPurpose": false,
  "IdentifierRefValidation": "None"
}
```

**Success response:** HTTP 201 with the created customer record, including the auto-assigned `CustomerId`.

---

## Get a Customer by ID

**Endpoint:** `GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')",
  method="GET"
)
```

Useful `$select` fields: `CustomerId, Name, AssociationNo, CustomerCategory, OneTime, B2bCustomer, Country, CountryCode, DefaultLanguage, CreationDate, IdentifierRefValidation, MainRepresentative, CorporateForm, BusinessClassification`

---

## Search / List Customers

**Endpoint:** `GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet`

**OData filters:**

| Goal | Query parameter |
|------|----------------|
| Filter by name | `$filter=contains(Name,'Acme')` |
| Filter by country | `$filter=Country eq 'SE'` |
| Filter by category | `$filter=CustomerCategory eq 'Customer'` |
| Limit results | `$top=10` |
| Select fields | `$select=CustomerId,Name,Country,CreationDate` |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet?$filter=contains(Name,'Acme')&$select=CustomerId,Name,Country,CreationDate&$top=10",
  method="GET"
)
```

---

## Update a Customer

Use PATCH to update one or more fields on an existing customer header. Only supply the fields you want to change.

**Endpoint:** `PATCH /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')`

**Updatable fields:**

| Field | Type | Notes |
|-------|------|-------|
| `Name` | string | Customer name |
| `AssociationNo` | string | Org/VAT number (digits only, no hyphens) |
| `DefaultLanguage` | string | ISO language code |
| `Country` | string | ISO 2-letter country code |
| `CountryCode` | string | Same as Country |
| `CorporateForm` | string | Legal form |
| `MainRepresentative` | string | Sales representative ID |
| `CustomerTaxUsageType` | string | Tax usage type |
| `B2bCustomer` | boolean | B2B flag |
| `OneTime` | boolean | One-time customer flag |
| `ValidDataProcessingPurpose` | boolean | GDPR flag |
| `BusinessClassification` | string | Business classification code |
| `DateOfRegistration` | date | Registration date (YYYY-MM-DD) |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')",
  method="PATCH",
  body={"Name": "Acme Corporation Updated", "MainRepresentative": "REP01"}
)
```

**Success response:** HTTP 200 with the full updated customer record.

---

## Copy an Existing Customer

Creates a new customer by copying settings from an existing one.

**Endpoint:** `POST /main/ifsapplications/projection/v1/CustomerHandling.svc/CopyExistingCustomer`

**Parameters:**

| Field | Type | Notes |
|-------|------|-------|
| `CustomerId` | string | Source customer ID to copy from |
| `NewId` | string | New customer ID (leave blank to auto-assign) |
| `NewName` | string | Name for the new customer |
| `NewCategory` | string | `"Customer"` or `"Prospect"` |
| `AssociationNo` | string | Org/VAT number for the new customer |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CopyExistingCustomer",
  method="POST",
  body={
    "CustomerId": "10041988",
    "NewName": "Acme Sweden AB",
    "NewCategory": "Customer",
    "AssociationNo": "5567890123"
  }
)
```

**Success response:** Returns the new `CustomerId` string.

---

## Change Customer Category

Converts a customer between categories (e.g. Prospect → Customer).

**Endpoint:** `POST /main/ifsapplications/projection/v1/CustomerHandling.svc/ChangeCustomerCategory`

**Parameters:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `CustomerId` | string | ✓ | Customer to change |
| `CustomerName` | string | ✓ | Customer name |
| `CustomerCategory` | string | ✓ | New category: `"Customer"` or `"Prospect"` |
| `PrevAssociationNo` | string | | Previous org number |
| `AssociationNo` | string | | New org number |
| `TemplateCustomerId` | string | | Template customer to apply |
| `TemplateCompany` | string | | Company for the template |
| `OverwriteOrderData` | boolean | | Whether to overwrite order data |
| `TransferAddressRelatedInfo` | boolean | | Transfer address-related info |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/ChangeCustomerCategory",
  method="POST",
  body={
    "CustomerId": "10041988",
    "CustomerName": "Acme Corporation",
    "CustomerCategory": "Customer",
    "OverwriteOrderData": false,
    "TransferAddressRelatedInfo": true
  }
)
```

---

## Validate Country Code

Before creating a customer, you can validate a country code:

**Endpoint:** `GET /main/ifsapplications/projection/v1/CustomerHandling.svc/GetCountryCode(CountryCode=IfsApp.CustomerHandling.Lookup_IsoCountry'{code}')`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/GetCountryCode(CountryCode=IfsApp.CustomerHandling.Lookup_IsoCountry'SE')",
  method="GET"
)
```

---

## Check if Association Number Already Exists

Before creating a customer, verify the org/VAT number is not already in use.

**Endpoint:** `GET /main/ifsapplications/projection/v1/CustomerHandling.svc/AssociationNoExist(AssociationNo='{number}')`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/AssociationNoExist(AssociationNo='5567890123')",
  method="GET"
)
```

Returns a string — non-empty means a customer already exists with that number.

---

## Get Company Defaults (for new customer setup)

Fetches default values from a company that should be applied when creating a customer.

**Endpoint:** `GET /main/ifsapplications/projection/v1/CustomerHandling.svc/GetCompanyDefaults(Company='{company}')`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/GetCompanyDefaults(Company='10')",
  method="GET"
)
```

---

## Create a Customer Address

Add a new address to an existing customer via the `CustomerInfoAddresses` navigation property.

**Endpoint:**
```
POST /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses
```

**Required fields:**

| Field | Type | Notes |
|-------|------|-------|
| `AddressId` | string | Address identifier, e.g. `"01"` |
| `CustomerId` | string | Must match the customer in the URL |
| `Country` | string | ISO 2-letter country code |
| `PartyType` | string | Always `"Customer"` |
| `DefaultDomain` | boolean | Always `true` — **required or the call will fail** |

**Optional fields:** `Address1`, `Address2`, `ZipCode`, `City`, `State`, `County`, `Name`, `EanLocation`, `ValidFrom`, `ValidTo`, `PrimaryContact`, `SecondaryContact`, `CustomerBranch`, `JurisdictionCode`

**Example request body:**
```json
{
  "AddressId": "01",
  "CustomerId": "10041990",
  "Address1": "Klarabergsviadukten 63",
  "ZipCode": "111 64",
  "City": "Stockholm",
  "Country": "SE",
  "PartyType": "Customer",
  "DefaultDomain": true
}
```

**Success response:** HTTP 201 with the full address record.

---

## Get Customer Addresses

Fetch all addresses for a customer.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses
```

**Recommended `$select` fields:**
`AddressId, CustomerId, Name, Address1, Address2, Address3, Address4, Address5, Address6, ZipCode, City, State, County, Country, CountryDesc, JurisdictionCode, EanLocation, ValidFrom, ValidTo, PrimaryContact, SecondaryContact, CustomerBranch, EndCustomerId, EndCustAddrId, DeliveryTypeExist, OneTimeDb, PartyType, Address`

**Get a specific address by AddressId:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses(CustomerId='{id}',AddressId='{addressId}')
```

**Get address types for an address:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses(CustomerId='{id}',AddressId='{addressId}')/AddressTypes?$select=AddressTypeCode,DefAddress
```
Address type codes include: `Delivery`, `Document`, `Pay`, `PrimaryContact`, `Visit`.

**Get communication methods for an address:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses(CustomerId='{id}',AddressId='{addressId}')/AddressCommunicationMethods?$select=CommId,Name,Description,MethodId,Value,MethodDefault,AddressDefault,ValidFrom,ValidTo
```

---

## Update a Customer Address

Use PATCH to update one or more fields on an existing address. Only supply the fields you want to change.

**Endpoint:**
```
PATCH /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoAddresses(CustomerId='{id}',AddressId='{addressId}')
```

**Updatable fields:**

| Field | Type | Notes |
|-------|------|-------|
| `Address1` | string | Street line 1 |
| `Address2` | string | Street line 2 |
| `Address3`–`Address6` | string | Additional lines (often hidden for SE) |
| `ZipCode` | string | Postal/ZIP code |
| `City` | string | City name |
| `State` | string | State/region code |
| `County` | string | County |
| `Country` | string | ISO 2-letter country code |
| `Name` | string | Address name/label |
| `EanLocation` | string | EAN location number |
| `JurisdictionCode` | string | Tax jurisdiction code |
| `CustomerBranch` | string | Branch reference |
| `PrimaryContact` | string | Primary contact person ID |
| `SecondaryContact` | string | Secondary contact person ID |
| `ValidFrom` | date | Validity start date (YYYY-MM-DD) |
| `ValidTo` | date | Validity end date (YYYY-MM-DD) |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10009430')/CustomerInfoAddresses(CustomerId='10009430',AddressId='01')",
  method="PATCH",
  body={"City": "STOCKHOLM", "ZipCode": "111 22"}
)
```

**Success response:** HTTP 200 with the full updated address record.

---

## Get Customer Contacts

Fetch all contacts linked to a customer.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInfoContacts
```

**Key fields returned:** `PersonId, CustomerId, Role, CustomerPrimary, CustomerSecondary, CustomerAddress, Phone, Mobile, Email, Fax, Created, Changed, NoteText, BlockedForCrmObjects`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/CustomerInfoContacts?$select=PersonId,Role,CustomerPrimary,Phone,Mobile,Email",
  method="GET"
)
```

---

## Create a Customer Contact (New Person)

Creates a new person and links them as a contact to the customer in one step.

**Endpoint:** `POST /main/ifsapplications/projection/v1/CustomerHandling.svc/CreateCustomerContact`

**Parameters:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `CustomerId` | string | ✓ | Customer to link contact to |
| `PersonId` | string | | Leave blank to auto-assign |
| `CustomerAddress` | string | | Address ID to link contact to |
| `CopyAddress` | boolean | | Copy address to person record |
| `Role` | string | | Contact role code |
| `FullName` | string | | Full name (alternative to First/Last) |
| `FirstName` | string | | First name |
| `MiddleName` | string | | Middle name |
| `LastName` | string | | Last name |
| `Title` | string | | Title (Mr, Mrs, etc.) |
| `JobTitle` | string | | Job title |
| `Initials` | string | | Initials |
| `Phone` | string | | Phone number |
| `Mobile` | string | | Mobile number |
| `Email` | string | | Email address |
| `Fax` | string | | Fax number |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CreateCustomerContact",
  method="POST",
  body={
    "CustomerId": "10041988",
    "FirstName": "Anna",
    "LastName": "Svensson",
    "JobTitle": "Purchasing Manager",
    "Phone": "+46 8 123 456",
    "Email": "anna.svensson@acme.se",
    "Role": "PURCHASE",
    "CopyAddress": false
  }
)
```

---

## Create a Customer Contact (Existing Person)

Links an already-existing person record as a contact to a customer.

**Endpoint:** `POST /main/ifsapplications/projection/v1/CustomerHandling.svc/CreateCustomerContactForExistPerson`

**Parameters:**

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `CustomerId` | string | ✓ | Customer to link contact to |
| `PersonId` | string | ✓ | Existing person ID |
| `CustomerAddress` | string | | Address ID to link contact to |
| `CopyAddress` | boolean | | Copy address to person record |
| `Role` | string | | Contact role code |
| `NoteText` | string | | Notes about this contact relationship |

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CreateCustomerContactForExistPerson",
  method="POST",
  body={
    "CustomerId": "10041988",
    "PersonId": "PER00123",
    "Role": "PURCHASE",
    "CopyAddress": false
  }
)
```

---

## Get Customer Communication Methods

Fetch all communication methods (phone, email, fax, web, etc.) at the customer level.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CommunicationMethods
```

**Key fields:** `CommId, Name, Description, MethodId, Value, MethodDefault, ValidFrom, ValidTo`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/CommunicationMethods?$select=CommId,MethodId,Value,MethodDefault",
  method="GET"
)
```

---

## Get Customer Sales Order Info

Fetches sales order defaults and settings for a customer (pricing, delivery, back-order options, etc.).

**Via navigation property (for one customer):**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustOrdCustomers
```

**Via direct entity set (filter/search across customers):**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustOrdCustomerSet?$filter=CustomerId eq '{id}'
```

**Key fields:** `CustomerNo, CustomerId, CurrencyCode, CustGrp, CustPriceGroupId, MarketCode, Discount, DiscountType, BackorderOption, CrStop, SalesmanCode, OrderId, ForwardAgentId, TemplateId, Priority, CyclePeriod, ConfirmDeliveries, PrintAmountsInclTax, B2bCustomer, NoteText`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/CustOrdCustomers?$select=CustomerNo,CurrencyCode,CustGrp,CustPriceGroupId,Discount,BackorderOption,SalesmanCode",
  method="GET"
)
```

---

## Update Customer Sales Order Info

Use PATCH to update sales order defaults for a customer.

**Endpoint:**
```
PATCH /main/ifsapplications/projection/v1/CustomerHandling.svc/CustOrdCustomerSet(CustomerNo='{id}')
```

**Key updatable fields:** `CurrencyCode, CustGrp, CustPriceGroupId, MarketCode, Discount, DiscountType, BackorderOption, SalesmanCode, OrderId, ForwardAgentId, TemplateId, Priority, CyclePeriod, ConfirmDeliveries, PrintAmountsInclTax, CrStop, NoteText`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustOrdCustomerSet(CustomerNo='10041988')",
  method="PATCH",
  body={"CurrencyCode": "SEK", "SalesmanCode": "SALES01", "BackorderOption": "AllowWithBackorderDate"}
)
```

---

## Get Customer Invoice Info (per Company)

Fetches invoice/payment settings for a customer within a specific company.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerInvoiceCompanies
```

**Key fields:** `Company, Identity, PayTermId, DefCurrency, TaxLiability, DefVatCode, NumerationGroup, TaxBookId, NoInvoiceCopies, TaxExempt, TaxExemptValidFrom, TaxExemptValidTo, CurrencyCode, EinvoiceReceiverCode, DigitalInvoice`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/CustomerInvoiceCompanies?$select=Company,PayTermId,DefCurrency,TaxLiability,DefVatCode,NoInvoiceCopies",
  method="GET"
)
```

---

## Get Customer Credit Info (per Company)

Fetches credit limit and credit control settings for a customer within a company.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerCreditInfoArray
```

**Key fields:** `Company, Identity, CreditLimit, CreditBlock, CreditRating, CreditNumber, AllowedDueDays, AllowedDueAmount, NextReviewDate, CreditAnalystCode, AvgDaysForPayment, NoteText, CreditRelationshipType`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/CustomerCreditInfoArray?$select=Company,CreditLimit,CreditBlock,CreditRating,NextReviewDate,CreditAnalystCode",
  method="GET"
)
```

---

## Get Customer CRM Info

Fetches CRM-specific information for a customer (account type, potential, turnover, etc.).

**Via navigation property:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/CustomerCrmInfoArray
```

**Via direct entity set:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CrmCustInfoSet?$filter=CustomerId eq '{id}'
```

**Key fields:** `CustomerId, Turnover, CurrencyCode, KeyAccount, SourceId, PotentialId, LoyaltyId, EmployeeCountId, MainRepresentativeId, Note, AccountType, ParentCompany`

---

## Get Customer Our-IDs (per Company)

Fetches the ID this company uses to identify itself to the customer (used in EDI and messaging).

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/OurIds
```

**Key fields:** `CustomerId, Company, OurId`

**Example:**
```
call_protected_api(
  endpoint="/main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='10041988')/OurIds",
  method="GET"
)
```

---

## Get Customer Message Setups

Fetches EDI/messaging configuration for a customer.

**Endpoint:**
```
GET /main/ifsapplications/projection/v1/CustomerHandling.svc/CustomerInfoSet(CustomerId='{id}')/MessageSetups
```

**Key fields:** `CustomerId, MediaCode, MessageClass, Address, MethodDefault, SequenceNo, Locale`

---

## Workflows

### Workflow: Create a New Customer (with Address)

1. **Collect required info**: customer name, country, language, optionally category, VAT/org number, address details.
2. Optionally call `AssociationNoExist` to check if the org number is already registered.
3. **POST to CustomerInfoSet** with the required fields. Note: `AssociationNo` must be digits only — strip any hyphens.
4. **Note the auto-assigned `CustomerId`** from the 201 response.
5. **POST to CustomerInfoAddresses** with `DefaultDomain: true` (mandatory) plus address fields.
6. **Confirm** both the customer ID and address to the user.

### Workflow: Fetch and Update a Customer Address

1. **GET** the customer's addresses using the `CustomerInfoAddresses` navigation property.
2. Identify the target address by `AddressId`.
3. **PATCH** only the fields that need to change.
4. Confirm the update to the user using the returned record.

### Workflow: Copy an Existing Customer

1. Identify the source `CustomerId` to copy from.
2. Call `CopyExistingCustomer` with the new name and optionally a new category and org number.
3. The action returns the new `CustomerId`.
4. Optionally add addresses or update fields on the new customer.

### Workflow: Add a Contact to a Customer

1. For a **new person**: call `CreateCustomerContact` with name, job title, phone/email, role, and customer ID.
2. For an **existing person**: call `CreateCustomerContactForExistPerson` with the `PersonId` and customer ID.
3. Verify with `GET .../CustomerInfoContacts` that the contact was linked.

---

## Common Country Codes

| Country | Code |
|---------|------|
| Sweden | SE |
| United Kingdom | GB |
| United States | US |
| Germany | DE |
| France | FR |
| Norway | NO |
| Denmark | DK |
| Finland | FI |
| Netherlands | NL |
| Spain | ES |

## Common Language Codes

| Language | Code |
|----------|------|
| English | en |
| Swedish | sv |
| German | de |
| French | fr |
| Norwegian | no |
| Danish | da |
