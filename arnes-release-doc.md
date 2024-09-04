# Arnes release workbook
---

### Klargjøring
* Sørg for at du har u/p til alle servere
* Last ned releasen fra [mlds](https://mlds.ihtsdotools.org/#/viewReleases)
---

### Smoketest [Demo](https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)
* Deaktivere DailyBuild
  >**PUT /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >Request body:  
  ```
  {
  "name": "Norwegian Edition",
  "owner": "The Norwegian Directorate of Health",
  "countryCode": "no",
  "defaultLanguageCode": "no",
  "dailyBuildAvailable": false
  }
  ```
  >*Execute*  
* Verifiser at dailyBuild er av og at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  >Response body:  
  ```
  {
  .
  .
  "dependantVersionEffectiveTime": YYYYMMDD,
  "dailyBuildAvailable": false,
  .
  .
  }
  ```
* Lag en import job
  >**POST /imports**  
  >Request body:  
  ```
  {
  "type": "SNAPSHOT", 
  "branchPath": "MAIN/SNOMEDCT-NO", 
  "createCodeSystemVersion": false  
  }
  ```
  >*Execute*  
  >*Kopier importId*  

##### !!!!! Må createCodeSystemVersion settes til "true" når man ikke skal kjøre en merge? Må vi kjøre en merge på smoketest? Bør vi droppe smoketest? So many questions. Det må vel tilgjengeligjøres "noe" for at hdir skal kunne teste? Må man kjøre rollback på Demo til slutt og loade på nytt med derivater?

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "WAITING_FOR_FILE"
  .
  .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "COMPLETED"
  .
  .
  }
  ```
* ##### !!! Sannsynligvis kjøre merge, eller delete/create /MAIN/SNOMEDCT-NO/REFSETS. Siste tipp er at man kjører med cCSV true og ikke gjør noe mer? Da kan NRC sjekke NO release i browser? 
* Gi beskjed til helsedirektoratet slik at de kan teste
---

### Importer i [DailyBuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) for eksport av derivater

* Deaktivere dailyBuild
  >**PUT /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >Request body:  
  ```
  {
  "name": "Norwegian Edition",
  "owner": "The Norwegian Directorate of Health",
  "countryCode": "no",
  "defaultLanguageCode": "no",
  "dailyBuildAvailable": false
  }
  ```
  >*Execute*  
* Verifiser at dailyBuild er av og at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
  .
  .
  "dependantVersionEffectiveTime": YYYYMMDD,
  "dailyBuildAvailable": false,
  .
  .
  }
  ```
* ##### !!! I følge gjeldene doc: rollback
* Rollback DailyBuild 
  >**POST /codesystems/{shortname}daily-build/rollback**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
* Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
  >**GET /{branch}/authoring-stats**  
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    "newConceptsCount": 0,
    "inactivatedConceptsCount": 0,
    "reactivatedConceptsCount": 0,
    "changedFsnCount": 0,
    "inactivatedSynonymsCount": 0,
    "newSynonymsForExistingConceptsCount": 0,
    "reactivatedSynonymsCount": 0,
    "executionTime": 1725441442988,
    "title": "Authoring changes since last release"
  }
  ```
* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
  "type": "SNAPSHOT", 
  "branchPath": "MAIN/SNOMEDCT-NO", 
  "createCodeSystemVersion": "true/false"  !!!! <- hva blir rett her?
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
  .
  .
  location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
  .
  .
  }
  ```
  >*Kopier importId*  

##### createCodeSystemVersion må vel også være true her for at release av derivater kan kjøres etterpå ??? Eller holder det at den er lastet in???

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "WAITING_FOR_FILE"
  .
  .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "COMPLETED"
  .
  .
  }
  ```
---

### Generer og hent rf2 pakker
* Generer rf2 pakker for derivatene fra [refset-editor](https://refset.terminologi.helsedirektoratet.no)
  - Log in
  - Velg ***Release*** oppe til høyre
  - Velg organisasjonsenhet og eventuellt fjern referansesett fra utvalg
    > Direktoratet for e-helse - Diagnoser: Fjern hake for ICPC2 (importeres via backend API)  
    > Direktoratet for e-helse - Legemidler: Alle markert  
    > International Council of Nurses (ICN): Alle markert  
  - RelaseCodeSystem NewDependantVersion: 20YY-MM-15
  - Versjonsdato: 20YYMM15
  - *Lag ny versjon i snowstorm dailybuild og eksporter pakke klar til produksjon*: markert
  - La resten være som det er og trykk på ***Lag ny versjon***
* Last ned de zippede rf2 pakkene fra Azure
  - portal - storage accounts - stfsflsoutputsprodnoe1 - containers - refset-release-dev
---
##### Her må vel ICPC2 tilgjengeligjøres på DB på noe vis? Eller er den tilgjengelig i REFSETS branch og det er den som blir syncet rundt før den blir nuket og rebuilt til slutt??=? Backend sync DB /MAIN/SNOMEDCT-NO/REFSETS -> DB /MAIN/SNOMEDCT-NO ??? 
---

### Importere norsk ekstensjon og derivater i [prod](https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)

* Verifiser at SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
  .
  .
  "dependantVersionEffectiveTime": YYYYMMDD,
  "dailyBuildAvailable": false,
  .
  .
  }
  ```
* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
  "type": "SNAPSHOT", 
  "branchPath": "MAIN/SNOMEDCT-NO", 
  "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
  .
  .
  location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
  .
  .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "WAITING_FOR_FILE"
  .
  .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "COMPLETED"
  .
  .
  }
  ```
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

* Importere ICPC2 til prod fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Trykk på **Authorize** knappen oppe til høyre
  >**POST /api/RefsetTools/sync
  >Request body:
  ```
  {
  "userName": "username",
  "password": "password",
  "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
  "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
  "targetBaseUri": "https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
  "targetBranch": "MAIN/SNOMEDCT-NO",
  "refset": {
    "sourceRefsetId": "68101000202102",
    "sourceModuleId": "51000202101",
    "targetRefsetId": "68101000202102",
    "targetModuleId": "51000202101",
    "includeMembers": false,
    "deleteFromTarget": false,
    "deleteDiffsFromTarget": false,
    "addToTarget": true,
    "addWithNewMemberId": false
  }
  }
  ```
* Gå tilbake til [prod](https://snowstorm.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
  "effectiveDate": 20YYMM15,
  "description": "[Month] release, norsk ekstensjon",
  "internalRelease": false
  }
  ```
  >*Execute*  
* ##### !!! delete/create /MAIN/SNOMEDCT-NO/REFSETS ???
---

### Importere norsk ekstensjon og derivater i [dailybuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)

* ##### !!! Rollback? If so:
* Rollback DailyBuild 
  >**POST /codesystems/{shortname}daily-build/rollback**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
* Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
  >**GET /{branch}/authoring-stats**  
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    "newConceptsCount": 0,
    "inactivatedConceptsCount": 0,
    "reactivatedConceptsCount": 0,
    "changedFsnCount": 0,
    "inactivatedSynonymsCount": 0,
    "newSynonymsForExistingConceptsCount": 0,
    "reactivatedSynonymsCount": 0,
    "executionTime": 1725441442988,
    "title": "Authoring changes since last release"
  }
  ```
* ##### Rollback complete
* Start med norsk ekstensjon, gjenta deretter prosedyren under med rf2 pakkene lastet ned fra Azure (Diagnoser, Legemidler og ICN):

* *!!!-- prosedyre start --!!!*

* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
    "type": "SNAPSHOT", 
    "branchPath": "MAIN/SNOMEDCT-NO", 
    "createCodeSystemVersion": "false"  
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
  .
  .
  location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
  .
  .
  }
  ```
  >*Kopier importId*  

* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "WAITING_FOR_FILE"
  .
  .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "COMPLETED"
  .
  .
  }
  ```
* Norsk ekstensjon
* Diagnoser
* Legemidler
* ICN

* *!!!-- prosedyre stop --!!!*

* Importere ICPC2 til dailybuild fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Trykk på **Authorize** knappen oppe til høyre
  >**POST /api/RefsetTools/sync
  >Request body:
  ```
  {
  "userName": "username",
  "password": "password",
  "sourceBaseUri": "https://dailybuild.terminologi.helsesdirektoratet.no/snowstorm/snomed-ct",
  "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
  "targetBaseUri": "https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
  "targetBranch": "MAIN/SNOMEDCT-NO",
  "refset": {
    "sourceRefsetId": "68101000202102",
    "sourceModuleId": "51000202101",
    "targetRefsetId": "68101000202102",
    "targetModuleId": "51000202101",
    "includeMembers": false,
    "deleteFromTarget": false,
    "deleteDiffsFromTarget": false,
    "addToTarget": true,
    "addWithNewMemberId": false
  }
  }
  ```
* Gå tilbake til [dailybuild](https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/) og lag en ny versjon av SNOMEDCT-NO
  >**POST /codesystems/{shortName}/versions  
  >shortname: SNOMEDCT-NO  
  >Request body:  
  ```
  {
  "effectiveDate": 20YYMM15,
  "description": "[Month] release, norsk ekstensjon",
  "internalRelease": false
  }
  ```
  >*Execute*  
* ##### !!! delete/create /MAIN/SNOMEDCT-NO/REFSETS ???
* ##### !!! enable dailyBuild ?
---

### Importere norsk ekstensjon og derivater for [demo](https://demo.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/swagger-ui/index.html#/)
* ##### !!! Deaktivere dailyBuild ?? Er av idag 04.09.2024
* Rollback Demo
  >**POST /codesystems/{shortname}daily-build/rollback**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
* Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
  >**GET /{branch}/authoring-stats**  
  >branch: MAIN/SNOMEDCT-NO  
  >*Execute*  
  ```
  {
    "newConceptsCount": 0,
    "inactivatedConceptsCount": 0,
    "reactivatedConceptsCount": 0,
    "changedFsnCount": 0,
    "inactivatedSynonymsCount": 0,
    "newSynonymsForExistingConceptsCount": 0,
    "reactivatedSynonymsCount": 0,
    "executionTime": 1725441442988,
    "title": "Authoring changes since last release"
  }
  ```
* Verifiser at (dailyBuild er av og at) SNOMEDCT-NO har riktig avhengighet til den internasjonale utgaven
  >**GET /codesystems/{shortname}**  
  >shortName: SNOMEDCT-NO  
  >*Execute*  
  ```
  {
  .
  .
  "dependantVersionEffectiveTime": YYYYMMDD,
  "dailyBuildAvailable": false,
  .
  .
  }
  ```
* Lag en import job
  >**POST /imports**
  Request body:
  ```
  {
  "type": "SNAPSHOT", 
  "branchPath": "MAIN/SNOMEDCT-NO", 
  "createCodeSystemVersion": "false"
  }
  ```
  >*Execute*  
  >Response body:  
  ```
  {
  .
  .
  location: https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct/imports/[LANG importId TALLSTRENG HER]
  .
  .
  }
  ```
  >*Kopier importId*  
* Sjekk status på import job
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "WAITING_FOR_FILE"
  .
  .
  }
  ```
* Importer fil
  >**GET /imports/{importId}/archive**  
  >ImportId:*importId fra tidligere steg*  
  >file: *choose file* - velg zip filen du lastet ned fra mlds i første steg  
  >*Execute*  
* Vent 5ish min og ta en ny sjekk av status på import job som over. Hvis status er "RUNNING", vent litt til
  >**GET imports/{importId}**  
  >ImportId: *importId fra forrige steg*  
  >*Execute*  
  >Response body:  
  ```
  {
  "status": "COMPLETED"
  .
  .
  }
  ```

---


### Importere norsk ekstensjon og derivater for SLV
* ##### ikke rollback
---
### Importere norsk ekstensjon og derivater for SLV-TEST
* ##### ikke rollback
---
---
---



### Importere ICPC2 til DailyBuild fra [backend](https://app-fsre-backend-prod-noe-1.azurewebsites.net/index.html)
* Trykk på **Authorize** knappen oppe til høyre
  >**POST /api/RefsetTools/sync
  >Request body:
  ```
  {
  "userName": "username",
  "password": "password",
  "sourceBaseUri": "https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
  "sourceBranch": "MAIN/SNOMEDCT-NO/REFSETS",
  "targetBaseUri": "https://dailybuild.terminologi.helsedirektoratet.no/snowstorm/snomed-ct",
  "targetBranch": "MAIN/SNOMEDCT-NO/DECKICPC2",
  "refset": {
    "sourceRefsetId": "68101000202102",
    "sourceModuleId": "51000202101",
    "targetRefsetId": "68101000202102",
    "targetModuleId": "51000202101",
    "includeMembers": false,
    "deleteFromTarget": false,
    "deleteDiffsFromTarget": false,
    "addToTarget": true,
    "addWithNewMemberId": false
  } 
}
  ```


---

### Importere norsk ekstensjon + derivater i alle miljøer. Start med Demo.

* Deaktivere DailyBuild (ikke prod)
>**PUT /codesystems/{shortname}**  
>shortName: SNOMEDCT-NO  
>Request body:  
```
{
"name": "Norwegian Edition",
"owner": "The Norwegian Directorate of eHealth",
"countryCode": "no",
"defaultLanguageCode": "no",
"dailyBuildAvailable": false
}
```
>*Execute*  

* Verifiser at dailyBuild er av
>**GET /codesystems/{shortname}**  
>shortName: SNOMEDCT-NO  
>*Execute*  
* Rollback DailyBuild (ikke prod)
>**POST /codesystems/{shortname}daily-build/rollback**  
>shortName: SNOMEDCT-NO  
>*Execute*  
*Sjekk at *Authoring Stats* står til 0. Hvis ikke gjør en ny rollback.
>**GET /{branch}/authoring-stats**  
>branch: MAIN/SNOMEDCT-NO  
>*Execute*  

* Sjekke at kodesystemet for den norske utgaven (SNOMEDCT-NO) har riktig avhengighet til den internasjonale utgaven
* Oppgrader avhengighet til riktig internasjonale versjon hvis nødvendig
* Start import Norsk ekstensjon
>**GET /codesystems/{shortname}  
>shortName: SNOMEDCT-NO  
>*Execute*  
```
{

}
```
* Generer rf2 pakker for derivatene fra [refset-editor](https://refset.terminologi.helsedirektoratet.no)
   - Log in
   - Velg ***Release*** oppe til høyre
   - Velg organisasjonsenhet og eventuellt fjern referansesett fra utvalg
     > Direktoratet for e-helse - Diagnoser: Fjern hake for ICPC2 (importeres via backend API)
     > Direktoratet for e-helse - Legemidler: Alle markert  
     > International Council of Nurses (ICN): Alle markert  
   - RelaseCodeSystem NewDependantVersion: 20YY-MM-15
   - Versjonsdato: 20YYMM15
   - La resten være som det er og trykk på ***Lag ny versjon***


