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
| **Sursă structură** | **tirandazdylan/drug-labels-snapshot – Hugging Face dataset** |
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
  "set_id": "a5e883e4-6993-4e78-9e5c-0cf38c4b1234",
  "id": "da441b80-1a22-48fb-bc24-897b71891111",
  "version": "14",
  "effective_time": "20231015",
  "fetched_on": "2024-03-01",
  "openfda_last_updated": "2024-02-28",
  "brand_name": "Glucophage, Metformin Hydrochloride",
  "generic_name": "METFORMIN HYDROCHLORIDE",
  "manufacturer_name": "Bristol-Myers Squibb Company",
  "product_type": "HUMAN PRESCRIPTION DRUG",
  "route": "ORAL",
  "rxcui": "860975, 860978",
  "boxed_warning": "WARNING: LACTIC ACIDOSIS: Postmarketing cases of metformin-associated lactic acidosis have resulted in death...",
  "indications_and_usage": "Metformin hydrochloride tablets are indicated as an adjunct to diet and exercise to improve glycemic control in adults with type 2 diabetes mellitus.",
  "dosage_and_administration": "The recommended starting dose is 500 mg orally twice a day, given with meals. Dosage increases should be made in increments of 500 mg weekly.",
  "contraindications": "Severe renal impairment (eGFR below 30 mL/min/1.73 m2). Known hypersensitivity to metformin. Acute or chronic metabolic acidosis.",
  "warnings_and_cautions": "Lactic acidosis; Vitamin B12 deficiency; Hypoglycemia with concomitant use of insulin or insulin secretagogues.",
  "drug_interactions": "Carbonic anhydrase inhibitors may increase risk of lactic acidosis. Cationic drugs eliminated by renal tubular secretion may increase metformin concentrations.",
  "use_in_specific_populations": "Geriatric Use: Assess renal function more frequently.",
  "pregnancy": "Metformin should be used during pregnancy only if clearly needed.",
  "do_not_use": null,
  "ask_doctor": null,
  "stop_use": null
}
```
</details>
<details>
  <summary>Exemplul 2: Medicament Ibuprofen</summary>
  
  ```json
{
  "drug": "ibuprofen",
  "query": "openfda.generic_name.exact:\"IBUPROFEN\"+AND+openfda.product_type:\"HUMAN OTC DRUG\"+AND+openfda.route:\"ORAL\"",
  "set_id": "c1f77d33-4011-4f11-b0e2-6638f364abcd",
  "id": "e912f91a-7b3c-41da-a719-f9c316712222",
  "version": "6",
  "effective_time": "20230620",
  "fetched_on": "2024-03-01",
  "openfda_last_updated": "2024-02-28",
  "brand_name": "Advil, Motrin IB",
  "generic_name": "IBUPROFEN",
  "manufacturer_name": "Haleon US Holdings LLC",
  "product_type": "HUMAN OTC DRUG",
  "route": "ORAL",
  "rxcui": "152923, 197806",
  "boxed_warning": null,
  "indications_and_usage": "Temporarily relieves minor aches and pains due to: headache, muscular aches, toothache, arthritis; temporarily reduces fever.",
  "dosage_and_administration": "Take 1 tablet every 4 to 6 hours while symptoms persist. If pain or fever does not respond, 2 tablets may be used.",
  "contraindications": null,
  "warnings_and_cautions": "Stomach bleeding warning: This product contains an NSAID, which may cause severe stomach bleeding.",
  "drug_interactions": "Do not use if taking aspirin for heart attack or stroke without talking to your doctor. Do not take with other NSAIDs.",
  "use_in_specific_populations": null,
  "pregnancy": null,
  "do_not_use": "If you have ever had an allergic reaction to any other pain reliever/fever reducer; right before or after heart surgery.",
  "ask_doctor": "Ask a doctor before use if you have high blood pressure, heart disease, liver cirrhosis, kidney disease, or asthma.",
  "stop_use": "Stop use and ask a doctor if you experience faintness, vomit blood, or have bloody stools."
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
