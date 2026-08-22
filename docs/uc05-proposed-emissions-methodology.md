# UC-05 — Metodologia emissiva proposta

**Versione:** 0.1  
**Data:** 22 agosto 2026  
**Stato:** metodologia candidata — da approvare  
**Ambito:** supporto metodologico al Macro Use Case UC-05

## 1. Scopo e posizione rispetto a UC-05

Questo documento descrive una metodologia candidata per trasformare i dati di
traffico osservati sul campo in una stima della massa di CO₂ veicolare associata
a una tratta e a un intervallo temporale.

La metodologia non costituisce un Use Case runtime e non impone COPERT come
unica soluzione possibile. Il flusso funzionale UC-05 rimane valido anche se
viene approvato un metodo emissivo alternativo, purché restino dichiarati e
compatibili:

- formula, unità e perimetro emissivo;
- categorie veicolari supportate;
- fattori o funzione emissiva e relativo dominio di applicazione;
- fonti, versione, territorio e periodo di riferimento;
- assunzioni, limitazioni e stato di approvazione.

Il risultato di UC-05 è una **stima di massa emissiva**, non una misurazione
della concentrazione atmosferica e non una previsione della qualità dell'aria.

## 2. Perimetro emissivo candidato

La baseline proposta stima la **CO₂ Hot tailpipe**, cioè la massa di CO₂ emessa
allo scarico con il motore a temperatura operativa. Non comprende, salvo futura
estensione esplicitamente approvata:

- cold start;
- emissioni evaporative;
- energia e combustibili upstream;
- ciclo di vita dei veicoli;
- CO₂ equivalente riferita ad altri gas climalteranti.

Questo perimetro corrisponde agli output `CO2_HOT` verificati nel POC COPERT
5.9.2 ed evita di attribuire al runtime dati che i dispositivi di campo non
forniscono.

## 3. Formula runtime proposta

Per ciascuna categoria veicolare `c`:

```text
A_c = N_c × d_c                         [veicolo-km]
E_c = A_c × EF_c(v_c)                  [gCO₂]
E   = Σ_c E_c                          [gCO₂]
```

Dove:

- `N_c` è il numero di transiti unici della categoria `c` nell'intervallo;
- `d_c` è la distanza in km attribuita a ogni transito;
- `A_c` è l'attività della categoria, espressa in veicolo-km;
- `v_c` è la velocità rappresentativa della categoria;
- `EF_c(v_c)` è il fattore emissivo medio della categoria alla velocità
  considerata, espresso in gCO₂/(veicolo-km);
- `E_c` è la massa stimata per categoria;
- `E` è la massa totale stimata nell'intervallo.

Nella proposta corrente `d_c` coincide con la lunghezza della tratta associata
al dispositivo. Si assume quindi che ciascun transito conteggiato percorra
l'intera tratta. L'assunzione va rivalutata in presenza di rampe, accessi
intermedi o più punti di rilevazione.

Se il campo fornisce una velocità media distinta per categoria, UC-05 usa
`v_c`. Se viene fornita una sola velocità rappresentativa della tratta, lo
stesso valore `v` viene applicato alle categorie dell'intervallo; questa scelta
deve essere dichiarata nei metadati del risultato.

La durata della finestra delimita i transiti osservati e non è un moltiplicatore
della formula: l'attività è già espressa da `N_c × d_c`.

## 4. Dati runtime richiesti

La metodologia utilizza l'osservazione validata prodotta da UC05-01:

- identificativo del dispositivo e della tratta;
- direzione di scorrimento;
- inizio e fine dell'intervallo;
- conteggio dei transiti unici per categoria;
- velocità media per categoria oppure velocità rappresentativa della tratta;
- indicatori di completezza e qualità disponibili.

Le categorie candidate sono `CAR`, `MOTORCYCLE`, `BUS` e `TRUCK`. Se FRONTIERE
confermerà la tassonomia COCO standard, i riferimenti candidati sono
`car/2`, `motorcycle/3`, `bus/5` e `truck/7`. La semantica effettiva della
categoria `truck` e le classi realmente esposte devono essere confermate con il
partner.

## 5. Preparazione offline del profilo di flotta

### 5.1 Fonti

La configurazione POC combina:

1. **ACI Open Parco Veicoli 2024 — provincia di Salerno**, per la consistenza
   dello stock per categoria, alimentazione e classe Euro;
2. **ISPRA — Dati COPERT Trasporto Strada 1990–2024**, foglio `veickm`, per le
   quote nazionali di attività/VKT nelle dimensioni non esposte da ACI;
3. **COPERT 5.9.2 Tier 3**, per il calcolo dei fattori Hot tailpipe CO₂ alle
   condizioni di guida definite nel POC.

ACI descrive il parco registrato al 31 dicembre e non il traffico effettivamente
osservato a Baronissi. ISPRA fornisce un proxy nazionale di attività. Il profilo
ottenuto è quindi un **profilo proxy di attività**, non la composizione certa
della flotta transitata in una specifica fascia oraria.

### 5.2 Perimetro ACI usato nel POC

| Categoria UC-05 | Stock ACI Salerno 2024 | Regola sorgente |
|---|---:|---|
| CAR | 737.963 | Autovetture (AV) |
| MOTORCYCLE | 138.710 | Motocicli (MC) |
| BUS | 2.233 | Autobus (AB) |
| TRUCK | 94.162 | Autocarri trasporto merci (AM) + trattori stradali (TS) |
| SPECIAL_TBD | 15.991 | Autoveicoli speciali (AS), esclusi dalla v0.1 |

TRUCK aggrega veicoli commerciali leggeri e pesanti perché la tassonomia
runtime candidata contiene una sola classe `truck`. Una classificazione più
granulare richiederebbe un nuovo mapping e nuovi fattori medi.

Il crosswalk completo, le assunzioni di mapping e i controlli di riconciliazione
sono disponibili in [UC-05 Fleet Crosswalk v0.1](./uc05-fleet-crosswalk-v0.1.md).

### 5.3 Calcolo dei pesi

Per una tecnologia COPERT `i` della categoria `c`, alimentazione `f` e classe
Euro `e`:

```text
q_(c,i) = S_ACI_(c,f,e) × p_ISPRA_(i | c,f,e,2024)
w_(c,i) = q_(c,i) / Σ_j q_(c,j)
```

Dove:

- `S_ACI_(c,f,e)` è lo stock ACI locale;
- `p_ISPRA_(i | c,f,e,2024)` è la quota condizionata di vehicle-km ISPRA;
- `w_(c,i)` è il peso normalizzato della tecnologia nel profilo della
  macro-categoria.

Per ogni categoria vale `Σ_i w_(c,i) = 1`. I pesi descrivono la composizione del
profilo e **non sono fattori emissivi**. Alimentazione, classe Euro,
massa/cilindrata e sottocategoria restano incorporate nelle tecnologie COPERT
ponderate. La quota elettrica resta nel denominatore del profilo e contribuisce
con zero alla CO₂ Hot tailpipe.

Non viene effettuata alcuna conversione di fattori emissivi da COPERT 5.8.1 a
5.9.2: ISPRA/COPERT 5.8.1 è usato come proxy per la composizione e l'attività;
i fattori sono calcolati direttamente con COPERT 5.9.2.

## 6. Generazione dei fattori con COPERT 5.9.2

Il POC usa:

- COPERT 5.9.2, Tier 3;
- ambiente `Urban Off Peak`;
- pendenza stradale 0%;
- carico HDT/BUS 50%;
- sette velocità: 10, 20, 30, 40, 50, 60 e 70 km/h;
- output Hot tailpipe CO₂ aggregati nelle quattro categorie UC-05.

I pesi normalizzati sono trasformati in flotte sintetiche con la stessa
attività media per tecnologia. Per ogni macro-categoria il denominatore è:

```text
D_c = 10^6 veicoli × 10^4 km/veicolo = 10^10 veicolo-km
```

Se `T_(c,i)(v)` è la massa Hot CO₂ in tonnellate prodotta da COPERT per la
tecnologia `i`, il fattore medio è:

```text
EF_c(v) = [Σ_i T_(c,i)(v) × 10^6 g/t] / 10^10 veicolo-km
```

Il pacchetto di input e lo script di esecuzione sono descritti nel
[README del package COPERT 5.9.2 v0.1.2](../artifacts/analysis/uc05/copert-592-ef-package-v0.1.2/README.md).
Gli output e la procedura di aggregazione sono documentati nel
[README dei risultati](../artifacts/analysis/uc05/copert-592-ef-results-v0.1/README.md).

## 7. Fattori emissivi ottenuti nel POC

Valori in gCO₂/(veicolo-km):

| Velocità (km/h) | CAR | MOTORCYCLE | BUS | TRUCK |
|---:|---:|---:|---:|---:|
| 10 | 273,68 | 234,51 | 1.930,66 | 852,67 |
| 20 | 200,02 | 145,74 | 1.321,68 | 636,25 |
| 30 | 166,22 | 113,42 | 1.006,54 | 509,22 |
| 40 | 147,64 | 99,11 | 833,20 | 431,64 |
| 50 | 137,20 | 93,09 | 725,43 | 383,53 |
| 60 | 131,90 | 91,83 | 656,82 | 357,29 |
| 70 | 130,29 | 93,82 | 616,58 | 349,35 |

La tabella machine-readable completa è disponibile in
[`UC05_COPERT_592_AGGREGATED_EF_v0.1.csv`](../artifacts/analysis/uc05/copert-592-ef-results-v0.1/UC05_COPERT_592_AGGREGATED_EF_v0.1.csv).

Gli EF più elevati alle basse velocità derivano dalle curve empiriche
velocità–consumo/emissione usate dalla metodologia Tier 3. Non è stato aggiunto
un coefficiente manuale basato sul tempo di permanenza.

## 8. Selezione del fattore a runtime

Tra due velocità della griglia, la proposta usa interpolazione lineare:

```text
EF_c(v) = EF_c(v_i)
        + [(v - v_i) / (v_(i+1) - v_i)]
        × [EF_c(v_(i+1)) - EF_c(v_i)]
```

La griglia 10–70 km/h è quella verificata nel POC e non rappresenta un limite
intrinseco di COPERT. Prima dell'uso operativo occorre approvare una delle
seguenti strategie:

- estendere la griglia a tutte le velocità realistiche della tratta;
- approvare esplicitamente una regola per i valori esterni al dominio;
- dichiarare la stima non disponibile quando manca un fattore applicabile.

Non deve essere applicato un fallback implicito.

## 9. Esempio runtime

Ipotesi:

- intervallo completo: 15 minuti;
- lunghezza tratta: 1,2 km;
- velocità rappresentativa: 40 km/h per tutte le categorie;
- transiti: 100 CAR, 15 MOTORCYCLE, 3 BUS e 12 TRUCK.

| Categoria | N_c | A_c = N_c × 1,2 | EF_c(40) | E_c |
|---|---:|---:|---:|---:|
| CAR | 100 | 120,0 veicolo-km | 147,64 | 17.717 gCO₂ |
| MOTORCYCLE | 15 | 18,0 veicolo-km | 99,11 | 1.784 gCO₂ |
| BUS | 3 | 3,6 veicolo-km | 833,20 | 3.000 gCO₂ |
| TRUCK | 12 | 14,4 veicolo-km | 431,64 | 6.216 gCO₂ |
| **Totale** | **130** | **156,0 veicolo-km** | — | **28.717 gCO₂** |

La stima complessiva è quindi circa **28,72 kgCO₂** nell'intervallo.

## 10. Validità, limiti e condizioni di approvazione

COPERT implementa la metodologia EMEP/EEA Tier 3 e consente di produrre fattori
dipendenti dalle caratteristiche veicolari e dalle condizioni di guida. Il POC
dimostra la fattibilità tecnica della procedura e la coerenza dimensionale dei
risultati; non costituisce, da solo, una validazione scientifica locale.

Limitazioni principali:

- stock provinciale ACI usato come proxy della flotta transitante;
- quote VKT ISPRA nazionali usate per dimensioni non disponibili in ACI;
- assenza di profili locali distinti per fascia oraria o tipo di giornata;
- categoria TRUCK aggregata;
- ipotesi POC su road type, carico e pendenza;
- granularità e qualità della velocità di campo da confermare;
- cold start e componenti non Hot tailpipe escluse.

Prima dell'uso operativo devono essere approvati almeno:

1. tassonomia e semantica degli input FRONTIERE;
2. distanza attribuita ai transiti e assunzione di attraversamento completo;
3. profilo di flotta e fonti proxy;
4. configurazione COPERT e perimetro emissivo;
5. dominio di velocità e regola di selezione/fuori dominio;
6. criteri di validazione, aggiornamento e versionamento dei fattori.

Lo storico prodotto da UC05-03 potrà supportare confronti e raffinamenti, ma i
soli conteggi per macro-categoria non permettono di osservare alimentazione,
classe Euro, massa o cilindrata. L'aggiornamento dei pesi interni richiede dati
locali più ricchi oppure nuove assunzioni esplicitamente approvate.

## 11. Evidenze e fonti

### Evidenze nella repository

- [Crosswalk ACI–ISPRA–COPERT](./uc05-fleet-crosswalk-v0.1.md)
- [Package di input COPERT 5.9.2](../artifacts/analysis/uc05/copert-592-ef-package-v0.1.2/README.md)
- [Risultati e procedura di aggregazione](../artifacts/analysis/uc05/copert-592-ef-results-v0.1/README.md)
- [Tabella EF aggregata in CSV](../artifacts/analysis/uc05/copert-592-ef-results-v0.1/UC05_COPERT_592_AGGREGATED_EF_v0.1.csv)
- [Manifest dei risultati](../artifacts/analysis/uc05/copert-592-ef-results-v0.1/result_manifest_v0.1.json)

### Fonti esterne

- [ACI Open Parco Veicoli](https://opv.aci.it/WEBDMCircolante/)
- [ACI — note metodologiche Open Parco Veicoli](https://opv.aci.it/WEBDMCircolante/noteOPV.html)
- [ISPRA — Inventario nazionale delle emissioni](https://emissioni.sina.isprambiente.it/inventario-nazionale/)
- [EMEP/EEA Road Transport Guidebook](https://copert.emisia.com/wp-content/uploads/2024/07/1.A.3.b.i-iv-Road-transport-2024.pdf)
- [COPERT methodology](https://copert.emisia.com/copert/methodology/)
- [Ultralytics COCO taxonomy](https://github.com/ultralytics/ultralytics/blob/main/ultralytics/cfg/datasets/coco.yaml)
