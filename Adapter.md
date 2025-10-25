> The Adapter Pattern is a structural design pattern that allows two incompatible systems to work together.
> It acts as a translator or bridge between two systems that have different interfaces — so one can understand the other without modifying existing code.

## Real-World Analogy: Hospital Management App & Insurance System

Imagine we have a **Hospital Management App** that handles patient records, billing, and treatment details.

Now, the hospital decides to **integrate with various Insurance Providers** (e.g., “MediCare”, “HealthPlus”, “SecureLife”) so patients can claim their insurance directly from the hospital’s system.

But there’s a problem.

Each insurance provider has a **different API format**:

| Provider   | API Request Format                           | Response Format                     |
| ---------- | -------------------------------------------- | ----------------------------------- |
| MediCare   | JSON fields: `firstName`, `lastName`, `dob`  | Returns approval as `isApproved`    |
| HealthPlus | XML data with tags `<patient>` `<policy>`    | Returns `<status>approved</status>` |
| SecureLife | JSON but uses `fname`, `lname`, `birth_date` | Returns `success: true/false`       |

The hospital’s app expects a **standard format**:

```json
{
  "name": "John Doe",
  "date_of_birth": "1990-01-01",
  "amount": 5000
}
```

and expects a standard response like:

```json
{
  "approved": true
}
```

---

## The Problem

If we directly integrate the hospital app with each insurance API, we will end up writing **different code for every provider**.

That’s messy and violates the **Open-Closed Principle** (open for extension, closed for modification).

---

## The Adapter Solution

We introduce **an Adapter** for each insurance provider.

Each adapter **converts the hospital’s standard data** into the specific format the insurance provider expects and **translates the provider’s response** back into the hospital’s expected format.


## Example

### Step 1: Common Interface (Target)

This is what the Hospital App expects:

```python
class InsuranceProvider:
    def request_approval(self, patient_data):
        pass
```

### Step 2: Adaptees (Incompatible APIs)

```python
class MediCareAPI:
    def send_claim(self, firstName, lastName, dob, amount):
        # Logic for MediCare API
        return {"isApproved": True}
```

```python
class HealthPlusAPI:
    def submit_insurance_request(self, xml_data):
        # XML processing...
        return "<status>approved</status>"
```

---

### Step 3: Adapters

Each adapter wraps an external API and adapts it to the hospital system.

#### MediCare Adapter

```python
class MediCareAdapter(InsuranceProvider):
    def __init__(self):
        self.medicare = MediCareAPI()

    def request_approval(self, patient_data):
        name_parts = patient_data["name"].split(" ")
        response = self.medicare.send_claim(
            firstName=name_parts[0],
            lastName=name_parts[1],
            dob=patient_data["date_of_birth"],
            amount=patient_data["amount"]
        )
        return {"approved": response["isApproved"]}
```

#### HealthPlus Adapter

```python
class HealthPlusAdapter(InsuranceProvider):
    def __init__(self):
        self.healthplus = HealthPlusAPI()

    def request_approval(self, patient_data):
        xml_data = f"""
        <patient>
            <name>{patient_data['name']}</name>
            <dob>{patient_data['date_of_birth']}</dob>
            <amount>{patient_data['amount']}</amount>
        </patient>
        """
        response_xml = self.healthplus.submit_insurance_request(xml_data)
        approved = "approved" in response_xml
        return {"approved": approved}
```

---

### Step 4: Usage in Hospital System

```python
def process_insurance(provider_adapter, patient_data):
    result = provider_adapter.request_approval(patient_data)
    print("Insurance Approved?" , result["approved"])

# Choose adapter dynamically
patient_data = {
    "name": "John Doe",
    "date_of_birth": "1990-01-01",
    "amount": 5000
}

adapter = MediCareAdapter()
process_insurance(adapter, patient_data)
```
---

## Why not call the APIs directly?

| Problem                      | Explanation                                                                                                                                                                   |
| ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Tight Coupling**        | Our hospital app now *knows about every insurance provider’s API details*. If any API changes (field names, endpoints, auth), our main code must change too.                |
| **2. Code Duplication**      | Every time we process insurance (e.g., billing, discharge, refund), we will repeat this big `if-else` logic.                                                                  |
| **3. Hard to Extend**        | Tomorrow, a new provider "CareX" joins — we must edit multiple parts of our app, violating **Open-Closed Principle**. |
| **4. Maintenance Nightmare** | If MediCare changes its field from `firstName` → `fname`, we must hunt through the entire codebase to fix it.                                                                |
| **5. Testing Difficulty**    | Each provider’s logic is mixed into the main app, making it hard to unit test them separately.                                                                                |
