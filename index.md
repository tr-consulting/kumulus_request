# Instruction Page: Upload File and Run Intric Service

This page explains how we currently make **dynamic API calls** to the Intric platform.  
We do **not** rely on static input/output definitions in Azure – everything is sent dynamically with each request.  
A consultant should build the same flow in Azure.

---

## Quick Overview
1. **Upload a file** to `POST /api/v1/files` using `form-data`. The response includes a `file id`.
2. **Run the service** with `POST /api/v1/services/{serviceId}/run`, sending:
   - Your custom **Prompt** (input).
   - The **file id** from step 1 inside `files[]`.

> Example service ID in this guide: `4275d5f6-3982-446a-a5dc-149916650b7d`

---

## Requirements
- An **Intric API key** (never expose this in client-side code).
- An HTTP client (e.g. `curl`, Postman, or server-side code).
- A test document (e.g. PDF hosted in GitHub).

---

## Step 1 – Upload File

**Endpoint**  
`POST https://api.intric.ai/api/v1/files/`

**Headers**

**Body (multipart/form-data)**
- `upload_file`: the document you want to analyze

**Example (curl)**
```bash
curl -X POST \
  https://api.intric.ai/api/v1/files/ \
  -H "Authorization: Bearer $INTRIC_API_KEY" \
  -F "upload_file=@./path/to/your/testdocument.pdf"
**Example JSON response**
{
  "id": "4511bc14-a32e-4087-bc96-23154eff5c8c",
  "filename": "testdocument.pdf",
  "mime_type": "application/pdf",
  "size": 123456
}
