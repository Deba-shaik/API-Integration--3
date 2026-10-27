# Topic 3 - Working with JSON Data
## N8N JSON Traversal Workflow

### Objective
Demonstrate handling nested JSON objects and arrays in n8n, extracting selected values with expressions, and sending the extracted data in a JSON HTTP POST request.

### Workflow architecture
`Manual Trigger -> Mock Invoice Data -> Extract Customer Data -> HTTP_POST_SendCustomerData`

### Node-by-node explanation
1. **Manual Trigger** - Starts the workflow on demand for testing.
2. **Mock Invoice Data** (Edit Fields, JSON mode) - Creates one mock invoice object with `receiptId`, a nested `customer` object, `purchasedItems` array, and a null `discountCode`.
3. **Extract Customer Data** (Edit Fields, Manual Mapping) - Extracts four fields from the input using n8n expressions:
   - `customerEmail` (String): `{{ $json.customer.contact.email }}`
   - `fullName` (String): `{{ $json.customer.firstName + ' ' + $json.customer.lastName }}`
   - `firstItemName` (String): `{{ $json.purchasedItems[0].name }}`
   - `firstItemPrice` (Number): `{{ $json.purchasedItems[0].price }}`
4. **HTTP_POST_SendCustomerData** (HTTP Request) - Sends the extracted data to `https://jsonplaceholder.typicode.com/posts` with POST and a JSON request body. The API returns a simulated created record with an ID.

### Example source JSON
```json
{
  "receiptId": "REC-9988",
  "customer": {
    "firstName": "John",
    "lastName": "Doe",
    "contact": {"email": "johndoe@example.com"}
  },
  "purchasedItems": [
    {"name": "Wireless Mouse", "price": 25.99},
    {"name": "Mechanical Keyboard", "price": 89.99}
  ],
  "discountCode": null
}
```

### POST configuration
- Method: `POST`
- URL: `https://jsonplaceholder.typicode.com/posts`
- Authentication: `None`
- Send Body: `ON`
- Body Content Type: `JSON`
- Specify Body: `Using JSON`
- JSON request body (with n8n expressions):
```json
{
  "customerEmail": "{{ $json.customerEmail }}",
  "fullName": "{{ $json.fullName }}",
  "firstItemName": "{{ $json.firstItemName }}",
  "firstItemPrice": {{ $json.firstItemPrice }}
}
```

### Observed successful output
```json
{
  "customerEmail": "johndoe@example.com",
  "fullName": "John Doe",
  "firstItemName": "Wireless Mouse",
  "firstItemPrice": 25.99,
  "id": 101
}
```

### Test and verification
1. Execute the Manual Trigger.
2. Check that Mock Invoice Data emits one item containing nested customer and purchased-items structures.
3. Check that Extract Customer Data emits the four expected fields and numeric price `25.99`.
4. Check that HTTP_POST_SendCustomerData returns the extracted fields and response `id: 101`.
5. Confirm that the final canvas shows four green node checkmarks and one item flowing to the HTTP node.

### Evidence
See `Topic3_JSON_Traversal_Evidence.pdf` for the actual node and final execution screenshots.

### Notes
- JSONPlaceholder is a public mock API. A successful POST demonstrates request/response handling but does not permanently store a record.
- The HTTP node initially rejected an object expression as invalid JSON; it was corrected by supplying a valid JSON body with field expressions.
- The supplied screenshots demonstrate a successful happy-path run. They do not establish behavior for missing customer fields, empty item arrays, or API outages.

### Submission
Include the exported n8n workflow JSON, this README, and the evidence PDF.
