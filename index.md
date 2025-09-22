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
```
Authorization: Bearer <API_KEY>
```

**Body (multipart/form-data)**
- `upload_file`: the document you want to analyze

**Example (curl)**
```bash
curl -X POST   https://api.intric.ai/api/v1/files/   -H "Authorization: Bearer $INTRIC_API_KEY"   -F "upload_file=@./path/to/your/testdocument.pdf"
```

**Example Response**
```json
{
  "id": "4511bc14-a32e-4087-bc96-23154eff5c8c",
  "filename": "testdocument.pdf",
  "mime_type": "application/pdf",
  "size": 123456
}
```

> Save the `id` – it will be used in step 2.

---

## Step 2 – Run the Service

**Endpoint**  
`POST https://api.intric.ai/api/v1/services/4275d5f6-3982-446a-a5dc-149916650b7d/run/`

**Headers**
```
Authorization: Bearer <API_KEY>
Content-Type: application/json
```

**Body (JSON)**
```json
{
  "input": "<DIN PROMPT HÄR>",
  "files": [
    { "id": "4511bc14-a32e-4087-bc96-23154eff5c8c" }
  ]
}
```

**Example (curl)**
```bash
curl -X POST   https://api.intric.ai/api/v1/services/4275d5f6-3982-446a-a5dc-149916650b7d/run/   -H "Authorization: Bearer $INTRIC_API_KEY"   -H "Content-Type: application/json"   -d '{
    "input": "<DIN PROMPT HÄR>",
    "files": [{ "id": "4511bc14-a32e-4087-bc96-23154eff5c8c" }]
  }'
```

---

## Prompt (in Swedish)

```text
Du är en dokumentanalytiker som ska gå igenom den bifogade filen och kontrollera nedan stående punkter, om den bifogade filen inte är OCR tolkad gör du det först och sedan fortsätter du att följa instruktionerna. 
Du ska analysera dokumentet och gå igenom att dessa nedanstående punkter finns med, ytterst viktigt att svaret även är på svenska. Gå igenom punkt för punkt och vikta ditt beslut, samt notera hur säker din bedömning är. För varje punkt ska även ett textutdrag (snippet) och sidnummer (pageNumber) anges så att användaren kan dubbelkolla informationen.

Organisationsuppgifter

(Kontrollera att formatet av organisationsnummer stämmer, att ort finns med, och att ett revisionsdatum anges.)
	•	Organisationsnummer: NNNNNN-NNN
	•	Hemort: T.ex. Stockholm
	•	Reviderade: 24 april 2021, 2021-04-24 eller annat svenskt datumformat.

Om detta stämmer:
	•	Sätt "OrgDetails.match" till true, annars false.
	•	Ange även din säkerhet utav din bedömning som ett decimalvärde mellan 0 och 1 på "OrgDetails.confidence".
	•	Inkludera textutdraget under "OrgDetails.snippet".
	•	Ange sidnummer där informationen hittades under "OrgDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "OrgDetails.match" till false.
	•	Ange säkerhet till 0 % "OrgDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "OrgDetails.snippet".
	•	Ange sidnummer 0 på "OrgDetails.pageNumber".

Ändamål

Hitta ett stycke som handlar om organisationens ändamål eller syfte.

Om detta stämmer:
	•	Sätt "SyfteDetails.match" till true, annars false.
	•	Ange säkerhet på "SyfteDetails.confidence".
	•	Inkludera textutdraget i "SyfteDetails.snippet".
	•	Ange sidnummer "SyfteDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "SyfteDetails.match" till false.
	•	Ange säkerhet till 0 % "SyfteDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "SyfteDetails.snippet".
	•	Ange sidnummer 0 på "SyfteDetails.pageNumber".

Beslutande organ
	•	Årsmöte
	•	Extra årsmöte
	•	Styrelse

Om detta stämmer:
	•	Sätt "OrganDetails.match" till true, annars false.
	•	Ange säkerhet på "OrganDetails.confidence".
	•	Inkludera textutdraget i "OrganDetails.snippet".
	•	Ange sidnummer "OrganDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "OrganDetails.match" till false.
	•	Ange säkerhet till 0 % "OrganDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "OrganDetails.snippet".
	•	Ange sidnummer 0 på "OrganDetails.pageNumber".

Firmateckning
	•	Ordförande och kassör gemensamt.
	•	Styrelsen kan ge verksamhetsledaren fullmakt att teckna firman.

Om detta stämmer:
	•	Sätt "CompanySignDetails.match" till true, annars false.
	•	Ange säkerhet på "CompanySignDetails.confidence".
	•	Inkludera textutdraget i "CompanySignDetails.snippet".
	•	Ange sidnummer "CompanySignDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "CompanySignDetails.match" till false.
	•	Ange säkerhet till 0 % "CompanySignDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "CompanySignDetails.snippet".
	•	Ange sidnummer 0 på "CompanySignDetails.pageNumber".

Medlemskap
Får inte utesluta grupper av etnicitet eller kön.

Om detta stämmer:
	•	Sätt "MemberDetails.match" till true, annars false.
	•	Ange säkerhet på "MemberDetails.confidence".
	•	Inkludera textutdraget i "MemberDetails.snippet".
	•	Ange sidnummer "MemberDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "MemberDetails.match" till false.
	•	Ange säkerhet till 0 % "MemberDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "MemberDetails.snippet".
	•	Ange sidnummer 0 på "MemberDetails.pageNumber".

Medlemmars rättigheter och skyldigheter
	•	Rätt att delta i sammankomster och få information om verksamheten.
	•	Skyldighet att följa stadgar och beslut.
	•	Ingen rätt till del av föreningens tillgångar vid upplösning.

Om detta stämmer:
	•	Sätt "MemberRightsDetails.match" till true, annars false.
	•	Ange säkerhet på "MemberRightsDetails.confidence".
	•	Inkludera textutdraget i "MemberRightsDetails.snippet".
	•	Ange sidnummer "MemberRightsDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "MemberRightsDetails.match" till false.
	•	Ange säkerhet till 0 % "MemberRightsDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "MemberRightsDetails.snippet".
	•	Ange sidnummer 0 på "MemberRightsDetails.pageNumber".

Årsmöte
	•	Högsta beslutande organ.
	•	Hålls före april månads slut.
	•	Medlemmar som betalat medlemsavgift 30 dagar innan mötet har rösträtt.
	•	Kallelse skickas minst fyra veckor innan.

Om detta stämmer:
	•	Sätt "MeetingDetails.match" till true, annars false.
	•	Ange säkerhet på "MeetingDetails.confidence".
	•	Inkludera textutdraget i "MeetingDetails.snippet".
	•	Ange sidnummer "MeetingDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "MeetingDetails.match" till false.
	•	Ange säkerhet till 0 % "MeetingDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "MeetingDetails.snippet".
	•	Ange sidnummer 0 på "MeetingDetails.pageNumber".

Styrelsen
	•	Består av ordförande, vice ordförande, kassör och ledamöter.
	•	Beslutar genom majoritet och dokumenterar alla möten i protokoll.

Om detta stämmer:
	•	Sätt "BoardDetails.match" till true, annars false.
	•	Ange säkerhet på "BoardDetails.confidence".
	•	Inkludera textutdraget i "BoardDetails.snippet".
	•	Ange sidnummer "BoardDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "BoardDetails.match" till false.
	•	Ange säkerhet till 0 % "BoardDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "BoardDetails.snippet".
	•	Ange sidnummer 0 på "BoardDetails.pageNumber".

Stadgeändringar och upplösning
	•	Kräver beslut på två möten med minst en månads mellanrum och 2/3 majoritet.
	•	Vid upplösning ska tillgångar skänkas till ändamål som överensstämmer med organisationens syfte.

Om detta stämmer:
	•	Sätt "ChangeDetails.match" till true.
	•	Ange säkerhet på "ChangeDetails.confidence".
	•	Inkludera textutdraget i "ChangeDetails.snippet".
	•	Ange sidnummer "ChangeDetails.pageNumber".

Om detta INTE stämmer:
	•	Sätt "ChangeDetails.match" till false.
	•	Ange säkerhet till 0 % "ChangeDetails.confidence".
	•	Sätt text 'Kunde inte hitta detta avsnitt i dokumentet' i "ChangeDetails.snippet".
	•	Ange sidnummer 0 på "ChangeDetails.pageNumber".

Sammanfattning

Vid granskningen av dokumentet ska en kortfattad sammanfattning av analysen anges i "summary" som beskriver de viktigaste observationerna.

```

---

## JSON Schema (in Swedish)

```json
{"type":"object","$schema":"http://json-schema.org/draft-07/schema#","required":["summary","OrgDetails","SyfteDetails","OrganDetails","CompanySignDetails","MemberDetails","MemberRightsDetails","MeetingDetails","BoardDetails","ChangeDetails"],"properties":{"summary":{"type":"string","description":"En sammanfattande text över analysen av dokumentet."},"OrgDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean","description":"true om organisationsuppgifter är korrekta, annars false"},"snippet":{"type":"string","description":"Textutdrag från dokumentet som matchades för denna sektion."},"confidence":{"type":"number","maximum":1,"minimum":0,"description":"Säkerhet för matchning, 0 = ingen match, 1 = perfekt match"},"pageNumber":{"type":"integer","minimum":1,"description":"Sidnummer där texten hittades."}}},"BoardDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"OrganDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"SyfteDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"ChangeDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"MemberDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"MeetingDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"CompanySignDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}},"MemberRightsDetails":{"type":"object","required":["match","confidence","snippet","pageNumber"],"properties":{"match":{"type":"boolean"},"snippet":{"type":"string"},"confidence":{"type":"number","maximum":1,"minimum":0},"pageNumber":{"type":"integer","minimum":1}}}}}
```

---

## Notes
- This example shows **how we do it today**.  
- The consultant’s task is to **rebuild the same integration inside Azure**.  
- All values (`input`, `files[].id`) are **dynamic per request**.  


## Link to example file: 
[Ladda ner instruktionerna](intric_instructions.md)
