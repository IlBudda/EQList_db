# Prompt per l'agente autonomo: catalogo completo per brand

Incolla tutto quello che segue (dalla riga "MISSIONE" in poi) come primo messaggio di una nuova sessione Claude Code sul repo `IlBudda/EQList_db`, con il connettore Google Sheets attivo.

---

## MISSIONE
Per **ogni brand** presente nei fogli del Google Sheet `1hfBYQhWBHzwMcpVTuc2Tg8pKrB8mgC25ENayFKyOlgw`:
1. ricostruisci il **catalogo completo** del brand partendo **solo da fonti ufficiali** (sito web del produttore e PDF che ospita);
2. inserisci ogni prodotto **nel foglio giusto**, con le colonne compilate. Senza **mai inventare** nulla: se un dato non è nella fonte ufficiale, la cella resta vuota.

Lavora in autonomia finché tutti i brand non sono completati.

## FOGLI E COLONNE
| Categoria | Foglio | Prodotti nuovi in | Colonne |
|---|---|---|---|
| Luci e accessori luce | LG | Pending LG | `id, brand, name, appCategoryId, categoryId, subcategoryId, imageUrl, PreviewImg, accessories, description, cct, crmx, beamAngle, consumo, ampere, Presa, codiceIP, weight, websiteUrl` |
| Camere e accessori camera | Camera | Pending Camera | da `id` a `codiceIP` per le camere, `accessoryType … powerSource` per gli accessori (vedi intestazione del foglio) |
| Ottiche e accessori ottica | Ottiche | Pending Ottiche | `id … description` più `mount … websiteUrl` (colonne I:AF) |
| Grip | Grip | Pending Grip | `gripType … storageTemp, websiteUrl` |

- Per **LG, Camera e Ottiche**, con i loro Pending, le colonne sono definitive: **non aggiungerne, non toglierne, non rinominarle**.
- Per **Grip** e per **qualsiasi altro foglio** (Foto, Audio, Storage, Facility): se servono colonne nuove **chiedi prima all'utente**, con una proposta che includa esempi presi dalle fonti ufficiali, e aspetta la risposta. Nel frattempo prosegui con gli altri brand e categorie.
- I prodotti già presenti vanno aggiornati **dove si trovano**. I prodotti nuovi vanno nel **Pending** della categoria, con `stato` = "Da verificare" dove la colonna esiste.
- Per decidere la categoria (`categoryId` / `subcategoryId` / `appCategoryId`) usa i valori già presenti nel foglio. Non creare valori nuovi senza chiedere.

## REGOLE DEI DATI (non negoziabili)
- Tutto in **inglese**, nel formato già usato dalle colonne (guarda le righe esistenti dello stesso foglio). Metrico prima, imperiale dopo la barra.
- **Fonti:** solo il dominio ufficiale del brand e i suoi PDF. Mai rental house, rivenditori, cinelenses, forum o Wikipedia. I modelli fuori produzione si compilano solo se il produttore pubblica ancora una scheda; altrimenti restano come sono.
- **Descrizione:** testo ufficiale in inglese (2-4 frasi). Sostituisci quelle vuote, non in inglese o prese da siti non ufficiali.
- **Celle già piene:** non sovrascrivere. Fanno eccezione `websiteUrl` (va sulla pagina ufficiale) e i valori palesemente in contrasto con la scheda ufficiale (per esempio l'apertura): in quel caso vince l'ufficiale e lo annoti.
- **id:** `brand_famiglia_modello` minuscolo con underscore, unico in tutto il file (per esempio `zeiss_supreme_prime_15mm`). Le varianti (uncoated, SF, aperture o versioni diverse) hanno id distinti; i kit restano righe separate. Nomi nel formato del produttore, per esempio "SUPREME PRIME 15mm T1.8"; brand scritto come il produttore, in maiuscolo.
- **Id calcolati:** in LG, Camera e Pending Camera l'`id` è una formula calcolata da brand e nome. Per i prodotti nuovi copia la formula della riga sopra; per rinominare cambia il nome. In Pending Ottiche l'id è un valore normale.
- **Immagini:** immagine ufficiale del prodotto caricata nel repo immagini (`https://github.com/IlBudda/PagheApp_db/blob/main/<Brand>/<Categoria>/<file>?raw=true`). Le immagini nuove vanno pushate su main; modifiche ed eliminazioni su un branch.
- **Doppioni:** unisci solo se brand, famiglia, modello e variante coincidono. Nella riga che resta tieni la descrizione più completa, riempi i campi vuoti con quelli delle righe eliminate e aggiorna ogni riferimento all'id vecchio in **tutti** i fogli.
- **Collegamenti nei due versi**, nella colonna `accessories`:
  - accessori ↔ prodotti compatibili secondo il produttore;
  - ottiche ↔ camere **solo per attacco nativo** (gli adattatori non contano; mount intercambiabili ufficiali sì).

## RUOLI
- **Tu (modello principale):** inventario, confronto catalogo ufficiale ↔ foglio, piani dei doppioni e degli inserimenti, verifiche, tutte le operazioni distruttive (eliminare righe, rinominare) e i report.
- **Agenti economici** (`model: "sonnet"`, `subagent_type: "general-purpose"`, in parallelo e in background) per:
  - raccogliere il catalogo e le schede ufficiali di un brand o di una famiglia, producendo un JSON con le chiavi esatte delle colonne;
  - fare i caricamenti meccanici da file già preparati.
- Ogni agente riceve: file di input, file di output, fonti ammesse, il divieto di toccare fogli e repo (salvo scritture meccaniche esplicite), il divieto di inventare valori, un report breve. Ogni risultato degli agenti va riverificato da te.

## METODO TECNICO (dalle sessioni precedenti)
- Locale italiano: `;` nelle formule. Un valore che inizia con "+" va scritto con l'apostrofo davanti.
- findReplace regex in lotti di al massimo 25-30 richieste; un timeout significa che non è stato applicato. Pattern per sostituire un token: `(^|\s)OLD(,|$)` → `$1NEW$2`.
- **Righe esistenti:**
  1. Crea il foglio temporaneo `_map_tmp_<brand>` con colonne: numero di riga, valori nuovi ("" = invariato).
  2. Scrivi la formula `=IF(B1="";INDEX('Foglio'!$A:$<ultima>;$A1;COLUMN(B1)-1);B1)` e controlla ISNUMBER/ISERROR.
  3. Fai la copyPaste PASTE_VALUES riga per riga.
  4. Rileggi **tutto** il foglio e confrontalo cella per cella con l'atteso: devono risultare 0 differenze impreviste.
- **Prodotti nuovi:** aggiungi le righe in fondo al Pending (appendDimension se serve), copia formule e formati della riga sopra, poi verifica.
- Prima di eliminare righe, fai una copia della scheda (`duplicateSheet` → `_bak_<scheda>_<brand>`). Eliminala solo dopo che la verifica è passata, poi elimina anche i fogli temporanei.
- **Collegamenti per attacco:** raggruppa per combinazione di attacchi, poi usa una tabella chiave→stringa e una formula LET, infine un'unica copyPaste della colonna `accessories`; verifica con un diff.
- get_values con più di 25k token viene salvato su file: fai una lettura per turno, perché due letture nello stesso turno possono finire nello stesso file.
- Git: lavora sul branch `claude/vibrant-gauss-7w8kgw` (le immagini nuove vanno su main, come detto sopra). Non aprire PR senza richiesta.

## PROCEDIMENTO
**Fase 0 – Inventario.** Leggi `HANDOFF.md`, poi tutti i fogli. Crea `PROGRESS.md` nel repo con una tabella: foglio, brand, righe presenti, id duplicati, URL del sito ufficiale e stato (da fare / in corso / fatto / bloccato). Nella colonna stato:
- ARRI (luci, camere, accessori camera, grip, ottiche), Zeiss e Cooke (ottiche) sono **già lavorati** solo sulle righe esistenti: resta da fare il confronto con il catalogo completo (prodotti mancanti);
- le colonne di LG/Pending LG dopo `weight` sono state eliminate dall'utente: i dati tecnici aggiunti lì non esistono più; restano solo le colonne della tabella sopra.

Lavora brand per brand, partendo da quelli con più righe.

**Per ogni brand:**
1. **Catalogo ufficiale.** Un agente raccoglie dal sito ufficiale l'elenco completo dei prodotti per categoria, con URL, schede e descrizioni. Se il catalogo è grande, dividilo per famiglia su più agenti.
2. **Confronto.** Per ogni prodotto ufficiale decidi tra: già presente (riga da aggiornare), mancante (riga nuova nel Pending) o doppione (da unire). Le righe del foglio senza corrispondenza ufficiale (noleggio, legacy, generiche) restano: ricevono solo id puliti e unici, senza dati inventati.
3. **Costruzione.** Righe finali con id, nome, categoria, dati, descrizione, websiteUrl e immagine. Scarta i dati attribuiti alla variante sbagliata.
4. **Scrittura e verifica.** Prima gli aggiornamenti, poi gli inserimenti, poi le eliminazioni dei doppioni e l'aggiornamento dei riferimenti. Ogni passo va verificato.
5. **Collegamenti** nei due versi, poi verifica: nessun id orfano, nessun id doppio, nessuna voce ripetuta.
6. **Chiusura.** Elimina backup e fogli temporanei, aggiorna `PROGRESS.md` (numeri, cosa manca e perché), fai commit e push.

## QUANDO FERMARSI E CHIEDERE
- Colonne nuove, nei fogli diversi da LG, Camera e Ottiche.
- Valori nuovi di `categoryId` / `subcategoryId` / `appCategoryId`.
- Unioni o attribuzioni che richiedono un'ipotesi non supportata dalla fonte ufficiale.
- Operazioni negate o errori di permessi: non aggirarli, annotali e prosegui con il resto.
- In tutti gli altri casi prosegui in autonomia.

## FINE LAVORO
Hai finito quando ogni brand in `PROGRESS.md` è "fatto" o "bloccato" con un motivo. A quel punto:
- esegui un controllo globale su tutti i fogli: id unici, riferimenti risolti, nessun foglio temporaneo o di backup rimasto, nessuna descrizione non in inglese;
- aggiorna `HANDOFF.md`, fai commit e push;
- consegna un report finale in italiano con totali per foglio e brand (aggiornati, aggiunti, uniti, collegati), le decisioni prese, i casi bloccati e le domande aperte.
