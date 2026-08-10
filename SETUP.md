# GBSBFORYOU Outreach OS — Chrome + Gmail Setup

## क्यों पहले वाला index.html Chrome में नहीं खुला?
वह Google Apps Script Web App के अंदर चलने वाला HTML था और उसमें `google.script.run` इस्तेमाल हुआ था।
इसे सीधे Chrome में खोलने पर वह API मौजूद नहीं होती।

## Chrome Demo
`index.html` सीधे Chrome में खोलें। यह local dashboard है और browser localStorage में draft/contact data रखता है।

## वास्तविक Gmail सिस्टम
वास्तविक sending के लिए:
1. Google Sheet बनाएं।
2. Extensions → Apps Script खोलें।
3. पुराने package का `Code.gs` और Apps Script वाला `Index.html` जोड़ें।
4. `appsscript.json` permissions रखें।
5. `initializeSystem()` एक बार चलाएँ।
6. Google authorization दें।
7. Contacts sheet में official public contacts डालें।
8. Template में approved letter रखें।
9. पहले `Create Drafts for Selected` चलाएँ।
10. Drafts जाँचने के बाद `Send Selected` चलाएँ।

Chrome local dashboard और Gmail backend को बाद में एक ही Web App में जोड़ा जा सकता है।
