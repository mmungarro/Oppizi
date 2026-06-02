# opizzi
---

# **Part 2 – API Testing: Open Charge Map**

## **1. Overview**

This project contains automated API tests for the public **Open Charge Map API** endpoints:

- `GET /poi/`
- `GET /referencedata/`

The tests were created using **Postman**, following QA best practices:

- Status code validation  
- Response time validation  
- JSON Schema validation  
- Business logic validation  
- Documentation and reporting  

---

##  **2. Requirements**

To run the tests, you need:

- **Postman** (latest version)
- **Newman** (optional, for CLI execution)
- Internet connection
- Open Charge Map API Key (if required)

---

## **3. How to Import the Collection**

1. Open Postman  
2. Click **Import**  
3. Select the file:
    [Collections](./Evidence/Oppizi_Collection.postman_collection.json)

4. Import the environment file (if provided)  
5. Set your API key in the environment variable:
     - Use browser DevTools to get your API Key

---

## ▶️ **4. How to Run the Tests**

### **A. Run inside Postman**

1. Open the collection  
2. Select the request  
3. Click **Send**  
4. Open the **Tests** tab to view results  

### **B. Run using Newman (optional)**

**Install html reporter**, Follow the instructions on [htmlextra](https://github.com/DannyDainton/newman-reporter-htmlextra)
```bash
newman run Oppizi_Collection.postman_collection.json
```

Generate an HTML report:

```bash
newman run Oppizi_Collection.postman_collection.json -r htmlextra
```

---

## **5. Tests Implemented**

###  GET /poi/

Validations:

- Status code = **200**
- Response time < **1000ms**
- POI JSON Schema validation
- Business logic:
  - Must return **at least one POI**
  - Each POI must contain **valid latitude and longitude**

---

### GET /referencedata/

Validations:

- Status code = **200**
- Response time < **1000ms**
- Flexible JSON Schema validation
- Business logic:
  - `ChargerTypes.length > 0`
  - `ConnectionTypes.length > 0`

---

## **6. JSON Schemas**

Schemas are stored in Postman Collection Variables:

- `col_poischema`
- `col_referenceDataSchema`

The Reference Data schema is **flexible**, allowing:

- `string | null`
- `boolean | null`
- `number | null`
- Optional objects
- Optional arrays

This prevents failures caused by inconsistent API responses.

---

## **7. Business Logic Validations**

### GET /poi/
- API must return at least one charging point  
- Each charging point must include valid coordinates  
- Schema must match expected structure  

### GET /referencedata/
- Must include at least one charger type  
- Must include at least one connection type  
- Flexible schema must validate successfully  

---

## **8. Test Report (Example)**

### **Execution Summary**

| Endpoint | Status | Time | Schema | Business Logic |
|---------|--------|------|--------|----------------|
| GET /poi/ | ✔ Passed | 312ms | ✔ Passed | ✔ Passed |
| GET /referencedata/ | ✔ Passed | 487ms | ✔ Passed | ✔ Passed |

** All test passed successfully **

- [Postman Test Results Image](https://drive.google.com/file/d/1KBm0-JqoS93f6fAwvdKXBN3Is6VypiFg/view?usp=sharing)
- [HTML Report Image](https://drive.google.com/file/d/19kqt9rnTwASTOZes1_MfhXBXzlANMbxK/view?usp=sharing)


### **Observations**

- API responds quickly (average < 500ms)  
- Some fields from Reference_Data request return `null`, requiring a flexible schema  
- No functional issues detected  
- Reference Data is large; validating only key rules is recommended  

---

## 📦 **9. Deliverables**

This project includes:

- ✔ Postman Collection with all tests  
- ✔ Collection variables containing schemas  
- ✔ README.md (this document)  
- ✔ Optional: Newman reports or screenshots  

---

## **10. Contact**

If you need something else or you have any doubts, please feel free to [contact me:](mailto:mmungarro@gmail.com)

---
