# Dataset - Prospecte Medicamentoase

## 1. De ce acest dataset? La ce e bun și cine l-ar folosi?

### 1.1. Scopul și utilitatea
Vrem ca datasetul nostru să organizeze informațiile importante despre medicamente într-un format structurat, care să poată fi folosit de un model AI pentru a răspunde pacienților la întrebări despre tratamente, în special în cazul persoanelor care iau mai multe medicamente simultan.

Datasetul furnizează secțiunile cheie din prospectele oficiale necesare pentru:
- **Orarul și modul corect de administrare** (`dosage_and_administration`, instrucțiuni raportate la masă/ore).
- **Detecția interacțiunilor medicamentoase** (`drug_interactions`, `warnings_and_cautions`).
- **Verificarea contraindicațiilor raportate la afecțiuni existente** (`contraindications`, `do_not_use`, `precautions`).
- **Avertismente speciale** (`boxed_warning`, `stop_use`, `pregnancy_or_breast_feeding`).

### 1.2. Cine l-ar folosi?
1. **Pacienți și aparținători:** Pentru a primi un plan de tratament coerent, ghid de administrare clar și avertismente („Nu luați Suplimentul X concomitent cu Medicamentul Y”).
2. **Farmaciști / Medici:** Ca instrument de asistență preliminară pentru screening-ul rapid al posibilelor incompatibilități medicamentoase.

### 1.3. Ce aduce în plus și de ce nu a fost suficient un set gata existent?
- **Normalizarea numelui substanței active (`drug`):** Simplificăm denumirea generică prin eliminarea unor termeni precum hydrochloride, sodium, maleate sau USP, pentru a facilita căutarea medicamentelor în dataset.
- **Selectivitate pe câmpuri critice:** Se extrag strict secțiunile relevante pentru ghidarea pacientului, eliminând datele chimice, biologice sau de laborator care ar crește costul de indexare fără valoare directă pentru utilizatorul final.
- **Informații insuficiente:** Seturile deja existente nu au foarte multe date.
---

## 2. Surse de date

| Parametru | Detalii |
| :--- | :--- |
| **Sursă structură** | **[tirandazdylan/drug-labels-snapshot](https://huggingface.co/datasets/tirandazdylan/drug-labels-snapshot) – Hugging Face dataset** |
| **Sursă oficială** | **U.S. Food and Drug Administration (FDA) – openFDA Drug Product Labeling** |
| **Link API / Manifest** | [https://open.fda.gov/apis/drug/label/](https://open.fda.gov/apis/drug/label/) / `https://api.fda.gov/download.json` |
| **Licență** | **Domeniu Public (CC0 / U.S. Government Work)**. Datele furnizate de openFDA sunt publice și pot fi descărcate, procesate și redistribuite liber, atât în scopuri comerciale cât și necomerciale. |
| **Dimensiune după filtrare** | Aproximativ **300 MB**, în format **Parquet (`.parquet`)** |

---

## 3. Date sensibile și confidențialitate

**Prezența datelor personale (PII / PHI):** **Nu există.** Datele brute FDA Structured Product Labeling (SPL) conțin exclusiv informații publice și autorizate despre produse medicamentoase, producători și companii farmaceutice. Nu există date medicale despre pacienți individuali, prescripții reale, CNP-uri, adrese sau date de contact individuale.

---

## 4. Schema datelor

Fiecare rând din datasetul procesat respectă următoarea structură:

### 4.1. Tabel descriptiv al câmpurilor

| Câmp | Tip | Descriere |
| :--- | :--- | :--- |
| `drug` | `string` | Numele de bază al substanței active, curățat de săruri (ex: `"ibuprofen"`). |
| `query` | `string` | Interogare openFDA folosită pentru trasabilitate/re-verificare. |
| `set_id` | `string` | UUID-ul unic al etichetei SPL FDA (identifică versiunile aceluiași prospect). |
| `id` | `string` | Identificatorul unic intern al înregistrării openFDA. |
| `version` | `string` | Versiunea prospectului. |
| `effective_time` | `string` | Data intrării în vigoare a etichetei (`YYYYMMDD`). |
| `fetched_on` | `string` | Data la care a fost rulat scraper-ul (`YYYY-MM-DD`). |
| `openfda_last_updated` | `string` | Data de export a bazei de date de către FDA. |
| `brand_name` | `string` | Numele comercial al medicamentului (ex: `"Advil"`, `"Tylenol"`). |
| `generic_name` | `string` | Denumirea comună internațională completă (ex: `"IBUPROFEN"`). |
| `manufacturer_name` | `string` | Compania deținătoare a autorizației / producătorul. |
| `product_type` | `string` | `"HUMAN PRESCRIPTION DRUG"` sau `"HUMAN OTC DRUG"`. |
| `rxcui` | `string` | Cod RxNorm / RxCUI asociat (util pentru corelări ontologice). |
| `boxed_warning` | `string` / `null` | Avertisment „Black Box” (riscuri letale / critice). |
| `indications_and_usage` | `string` / `null` | Pentru ce afecțiuni este indicat medicamentul. |
| `dosage_and_administration` | `string` / `null` | Dozaj recomandat, orar, relație cu mesele. |
| `contraindications` | `string` / `null` | Afecțiuni/condiții în care administrarea e strict interzisă. |
| `warnings_and_cautions` | `string` / `null` | Avertizări generale de siguranță. |
| `warnings` / `precautions` | `string` / `null` | Precauții suplimentare (dese la medicamentele OTC). |
| `drug_interactions` | `string` / `null` | Interacțiuni cu alte medicamente, alcool, alimente etc. |
| `use_in_specific_populations` | `string` / `null` | Utilizare la vârstnici, copii, insuficiență hepatică/renală. |
| `pregnancy` / `pregnancy_or_breast_feeding` | `string` / `null` | Recomandări legate de sarcină și alăptare. |
| `do_not_use` / `stop_use` | `string` / `null` | Câmpuri OTC specifice privind oprirea administrării. |
| `ask_doctor` / `ask_doctor_or_pharmacist` | `string` / `null` | Recomandări OTC de consult prealabil. |

---

### 4.2. Exemple de înregistrări (reprezentare JSON)

<details>
  <summary>Exemplul 1: Medicament Metformin Hydrochloride</summary>

  ```json
{
  "drug": "metformin",
  "query": "openfda.generic_name.exact:\"METFORMIN HYDROCHLORIDE\"+AND+openfda.product_type:\"HUMAN PRESCRIPTION DRUG\"+AND+openfda.route:\"ORAL\"",
  "set_id": "14f3091c-f243-4364-b433-727c7d4ef8d5",
  "id": "560798a8-34ca-25ac-e063-6294a90ad854",
  "version": "19",
  "effective_time": "20260707",
  "fetched_on": "2026-10-27",
  "openfda_last_updated": "2026-10-27",
  "brand_name": "METFORMIN HYDROCHLORIDE EXTENDED RELEASE",
  "generic_name": "METFORMIN HYDROCHLORIDE",
  "manufacturer_name": "PD-Rx Pharmaceuticals, Inc.",
  "product_type": "HUMAN PRESCRIPTION DRUG",
  "rxcui": "860975",
  "boxed_warning": "lactic acidosis; renal impairment; discontinue treatment; hemodialysis",
  "indications_and_usage": "adjunct to diet and exercise; glycemic control; adults with type 2 diabetes mellitus",
  "dosage_and_administration": "500 mg once daily with the evening meal; increase by 500 mg weekly; maximum 2000 mg once daily",
  "contraindications": "eGFR below 30 mL/min/1.73 m²; hypersensitivity to metformin; acute or chronic metabolic acidosis",
  "warnings_and_cautions": "lactic acidosis; vitamin B12 deficiency",
  "warnings": "...",
  "precautions": "...",
  "drug_interactions": "...",
  "use_in_specific_populations": "...",
  "pregnancy": "...",
  "do_not_use": "...",
  "ask_doctor": "...",
  "ask_doctor_or_pharmacist": "...",
  "stop_use": "...",
  "pregnancy_or_breast_feeding": "..."
}
```
</details>
<details>
  <summary>Exemplul 2: Medicament Ibuprofen</summary>
  
  ```json
{
  "drug": "ibuprofen",
  "query": "openfda.generic_name.exact:\"IBUPROFEN\"+AND+openfda.product_type:\"HUMAN OTC DRUG\"+AND+openfda.route:\"ORAL\"",
  "set_id": "a7789f7f-3356-463b-9fd7-2d7f7eba2799",
  "id": "b0cdee36-49ad-4d25-97d9-3fb750682cfa",
  "version": "9",
  "effective_time": "20260908",
  "fetched_on": "2026-10-27",
  "openfda_last_updated": "2026-10-27",
  "brand_name": "Ibuprofen",
  "generic_name": "IBUPROFEN",
  "manufacturer_name": "The Kroger Co.",
  "product_type": "HUMAN OTC DRUG",
  "rxcui": "310964",
  "boxed_warning": null,
  "indications_and_usage": "minor aches and pains; temporarily reduces fever",
  "dosage_and_administration": "1 capsule every 4 to 6 hours; maximum 6 capsules in 24 hours",
  "contraindications": null,
  "warnings_and_cautions": null,
  "warnings": "allergic reaction; stomach bleeding; heart attack and stroke",
  "precautions": null,
  "drug_interactions": null,
  "use_in_specific_populations": null,
  "pregnancy": null,
  "do_not_use": "allergic reaction to another pain reliever/fever reducer; right before or after heart surgery",
  "ask_doctor": "stomach bleeding risk; heart disease; liver or kidney disease; asthma; stroke; diuretic use",
  "ask_doctor_or_pharmacist": "under a doctor's care; aspirin use; other drugs",
  "stop_use": "stomach bleeding; heart or stroke symptoms; pain lasting more than 10 days; fever lasting more than 3 days",
  "pregnancy_or_breast_feeding": "ask a health professional; avoid use at 20 weeks or later unless directed by a doctor"
}
  ```
</details>

## 5. Utilitatea datasetului:
**Pasul 1.** **Scenariul de test:** Definim profilul unui pacient (ex.: Pacient cu diabet tip 2 tratat cu Metformin, care vrea să ia Ibuprofen pentru dureri de spate și bea supliment de ceai verde / spirulină).
**Pasul 2.** **Rulăm sistemul AI în două moduri:**
- Baseline: LLM generic fără context integrat.
- RAG cu datasetul nostru: LLM-ul primește secțiunile extrase din dataset (drug_interactions, contraindications, dosage_and_administration).
**Pasul 3.** Pentru fiecare rezultat măsuram:
- Rata de detecție a incompatibilităților: Verificăm dacă avertismentul privind afectarea renală sau riscul crescut de sângerare este identificat corect.
- Acuratețea orarului de administrare: Verificăm dacă instrucțiunile particulare (ex: „se ia cu alimente”, „se ia dimineața pe stomacul gol cu 30 min înainte de masă”) sunt extrase fără halucinații.
