# Handoff: catalogo EQList (Google Sheet)

## Progetto
- Foglio: `1hfBYQhWBHzwMcpVTuc2Tg8pKrB8mgC25ENayFKyOlgw`.
- sheetId: Pending LG 520145386, LG 0, Pending Camera 1089733184, Camera 1150262836, Pending Ottiche 602795316, Ottiche 1477753055, Pending Grip 1368848797, Grip 1970721972.
- Repo immagini: `https://github.com/IlBudda/PagheApp_db/blob/main/<Cartella>/<file>?raw=true`. Le immagini nuove vanno direttamente su main; modifiche ed eliminazioni su un branch. Il branch di lavoro è `claude/vibrant-gauss-7w8kgw`.

## Regole fisse dell'utente
- Dati in inglese, schema di riferimento LG (per esempio "Tungsten", non "Tungsteno").
- Descrizioni e dati solo dal sito ufficiale del produttore. Senza fonte ufficiale si lascia com'è e lo si dichiara.
- id nel formato `brand_nome`, minuscolo, con underscore.
- Accessori collegati in entrambe le direzioni (prodotto ↔ accessorio).
- Modo di lavoro richiesto: Opus pianifica, verifica e fa le operazioni delicate; agenti economici (sonnet) per raccolta dati e caricamenti meccanici.

## Note tecniche importanti
- Locale italiano: nelle formule si usa `;` come separatore.
- In LG, Camera e Pending Camera l'id (colonna A) è una formula `=LOWER(SUBSTITUTE(B&"_"&C;" ";"_"))`: per rinominare si cambia il nome (colonna C). In Pending Ottiche gli id sono valori normali.
- findReplace con regex: con 70 o più richieste va in timeout (e non viene applicato); lotti da 25-30 funzionano. Non tocca le celle con formule.
- Sostituire un token: `(^|\s)OLD(,|$)` → `$1NEW$2`. Togliere un doppione adiacente: `(^|\s)X, X(,|$)`.
- Un valore che inizia con "+" va scritto con l'apostrofo davanti (`'+60°…`), altrimenti diventa #ERROR!.
- Metodo sicuro per scritture sparse:
  1. Foglio temporaneo con colonne: numero di riga, valori nuovi ("" = invariato).
  2. Formula `=IF(B1="";INDEX('Foglio'!$A:$AF;$A1;COLUMN(B1)-1);B1)`.
  3. Controllo ISNUMBER/ISERROR.
  4. copyPaste PASTE_VALUES riga per riga.
  5. Rilettura completa e diff.
  6. Eliminazione dei doppioni (dal basso verso l'alto) e del foglio temporaneo.
- Collegamenti per attacco: raggruppare per combinazione di attacchi, poi usare una tabella chiave→stringa e una formula LET nel foglio temporaneo, infine un'unica copyPaste sulla colonna G.
- get_values con più di 25k token viene salvato su file. Attenzione: due letture nello stesso turno possono finire nello stesso file.

## Fatto
- **LG / Pending LG**:
  - 20 colonne tecniche aggiunte; doppioni ARRI uniti (ARRIMAX 18/12, M8, M18, M40, M90, SkyPanel X21).
  - 64 luci ARRI compilate dal sito ARRI.
- **Camera**:
  - Schema di 41 colonne, più 7 colonne per gli accessori (accessoryType…powerSource); camere e accessori ARRI compilati.
  - Pending Camera unito in Camera (135 righe, FX6 unita); Pending Camera ora contiene solo l'intestazione.
- **Accessori camera ARRI**: doppioni ZMU-4 e UDM-1 uniti, id puliti (`arri_hi-5`, `arri_mb-14`, …), 49 righe compilate.
- **Grip ARRI**: 12 colonne aggiunte (stabilizedAxes…storageTemp) e 10 righe compilate dai manuali ARRI.
- **Ottiche ARRI**: schema I:AF, 93 doppioni uniti, 98 ottiche compilate.
- **Ottiche Zeiss**: 38 doppioni uniti; 181 id unici `zeiss_…`; 64 ottiche con dati ufficiali; 131 collegate a 39 camere per attacco nativo.
- **Ottiche Cooke**: 70 doppioni uniti; 331 id unici `cooke_…`; circa 160 ottiche con dati ufficiali; aperture corrette dove sbagliate; 241 collegate a 39 camere per attacco.

## Aperto / da decidere
- Ottiche e camere senza attacco indicato (50 Zeiss, 90 Cooke, 21 camere): restano scollegate finché non si compila l'attacco.
- Righe probabilmente errate: Cooke "Anamorphic/i 45mm" e "S8/i 350mm", non presenti nei listini ufficiali.
- id duplicato preesistente: `cooke_/_lomo_lomo_cooke_anamorphic`.
- LG: restano in italiano le descrizioni di DOWNLIGHT, DAYLIGHT FRESNEL, X CERAMIC, ARRI 2000/5000 W, MULTI-PAR e set ARRILITE 750. Decidere se "ARRI 2000 W"/"5000 W" sono le True Blue T2/T5 e se il set ARRILITE 750 va collegato o unito.
- Mancano i dati ufficiali per: modelli fuori produzione e storici (ARRISUN, Compact, Pocket PAR, Zeiss CP.2/DigiPrime, Cooke S4 pre-/i e vintage), EB MAX (solo PDF), peso e dimensioni di molti accessori ECS.
- Prossimi marchi di ottiche suggeriti dal flusso: Angénieux, Leitz, Canon, Sigma, Fujinon, con lo stesso procedimento (piano doppioni → agenti per i dati → caricamento verificato → collegamento per attacco).
