# API-szerződések

Ez a repository a Tantusz összes API-jának egyetlen hiteles forrása. API-végpontot, hitelesítési szerződést és kérés-/válaszsémát itt módosítunk.

- `services/`: publikus, böngészőből a BFF-en keresztül elérhető API-k.
- `internal/`: kizárólag belső service-ek számára készült szerződések; ezekből nem generálunk frontend-klienst.
- `components/`: megosztott sémák.
- `consumers/`: közös definíciókra hivatkozó, fogyasztónként összeállított generálási belépőpontok.

A backend, frontend és assistant a `swagger/tantusz-api-specs` submodule-on keresztül fogyasztja ezt a repót. Módosítás után azonos spec-commitra kell frissíteni a fogyasztókat és újragenerálni a kódot. A submodule-munkapéldányokban API-definíciót nem szerkesztünk. Generált kódot nem módosítunk kézzel.
