# POC COPERT 5.9.2 per UC-05

| Campo | Valore |
|---|---|
| Data del test | 19/08/2026 |
| Versione COPERT | 5.9.2, Tier 3 |
| Scenario | 2024, 70 km/h, Urban Off Peak |
| Stato | Prova tecnica preliminare, non validazione scientifica |

## Scopo

Questi artefatti documentano una prova end-to-end eseguita con COPERT 5.9.2
su Windows. Il test verifica che sia possibile:

1. configurare tecnologie riconducibili alle quattro macro-categorie di UC-05;
2. fornire stock, attività e condizioni di guida tramite workbook;
3. eseguire COPERT in modalità CLI;
4. leggere le emissioni `CO2_HOT` e ricavare fattori in gCO₂/vehicle-km.

Il POC non usa ancora i profili ACI/ISPRA pesati e non produce la matrice
operativa completa 10–70 km/h. È un riferimento tecnico per comprendere il
formato degli input e degli output COPERT.

## Contenuto della cartella

| File | Contenuto | Utilità |
|---|---|---|
| [`UC05_POC_592.cop`](UC05_POC_592.cop) | Progetto COPERT salvato durante la prova. | Permette di riaprire e ispezionare lo scenario con COPERT 5.9.2. È un file binario e non è leggibile direttamente nell'interfaccia Git. |
| [`UC05_POC_592_INPUT_70KMH.xlsx`](UC05_POC_592_INPUT_70KMH.xlsx) | Workbook di input per lo scenario a 70 km/h. | Mostra tecnologie, stock, attività, velocità, quote stradali e configurazioni usate. |
| [`UC05_POC_592_CLI_RESULTS_70KMH_RUN1.xlsx`](UC05_POC_592_CLI_RESULTS_70KMH_RUN1.xlsx) | Workbook generato dalla CLI COPERT. | Contiene gli output completi; per UC-05 il foglio principale è `CO2_HOT`. |

## Configurazione del test

Il workbook di input contiene otto tecnologie illustrative:

| Macro-categoria UC-05 | Tecnologia COPERT nel POC |
|---|---|
| `CAR` | Passenger Cars, Petrol, Mini, Euro 6 d/e |
| `CAR` | Passenger Cars, Petrol PHEV, Medium, Euro 6 d/e |
| `CAR` | Passenger Cars, LPG Bifuel, Medium, Euro 6 d/e |
| `CAR` | Passenger Cars, CNG Bifuel, Medium, Euro 6 d/e |
| `TRUCK` | Heavy Duty Trucks, Diesel, Rigid ≤7,5 t, Euro VI D/E |
| `BUS` | Buses, Diesel, Urban Buses Midi ≤15 t, Euro VI D/E |
| `BUS` | Buses, Diesel Hybrid, Urban Buses Diesel Hybrid, Euro VI D/E |
| `MOTORCYCLE` | L-Category, Petrol, Motorcycles 4-stroke 250–750 cm³, Euro 5 |

Per ogni riga sono stati impostati:

- stock: 1.000 veicoli;
- attività media: 10.000 km;
- quota Urban Off Peak: 100%;
- velocità Urban Off Peak: 70 km/h;
- quote Urban Peak, Rural e Highway: 0%;
- pendenza: 0%;
- carico HDT/BUS: 50%;
- quota primaria/secondaria per PHEV e bus ibrido: 64%/36%.

Le temperature presenti nel file sono valori di test e non costituiscono una
configurazione meteorologica validata. Il risultato considerato qui è
`CO2_HOT`, quindi non include il contributo cold-start.

## Lettura dell'output CO₂

Il foglio `CO2_HOT` riporta le emissioni in tonnellate per l'attività annuale
configurata. Per ciascuna tecnologia del POC:

$$
A_i = 1.000\;veicoli \times 10.000\;km = 10.000.000\;vehicle\text{-}km
$$

Il fattore corrispondente si ottiene con:

$$
EF_i = \frac{CO2\_HOT_i\;[t] \times 10^6\;[g/t]}
{10.000.000\;[vehicle\text{-}km]}
$$

Nella configurazione corrente il valore numerico in gCO₂/vehicle-km è quindi
pari al valore in tonnellate diviso 10.

### Risultati del run a 70 km/h

| Tecnologia | `CO2_HOT` (t) | EF derivato (gCO₂/vehicle-km) |
|---|---:|---:|
| CAR — Petrol Mini | 950,0987 | 95,0099 |
| CAR — Petrol PHEV Medium | 962,6417 | 96,2642 |
| CAR — LPG Bifuel Medium | 1.281,3648 | 128,1365 |
| CAR — CNG Bifuel Medium | 912,0405 | 91,2041 |
| TRUCK — Diesel Rigid ≤7,5 t | 2.585,1178 | 258,5118 |
| BUS — Diesel Urban Midi ≤15 t | 8.424,3782 | 842,4378 |
| BUS — Diesel Hybrid | 5.018,5203 | 501,8520 |
| MOTORCYCLE — Petrol 4-stroke 250–750 cm³ | 888,0025 | 88,8003 |

I valori della tabella sono specifici delle otto tecnologie e delle assunzioni
del POC. Non sono i fattori medi definitivi delle macro-categorie UC-05.

## Collegamento con la metodologia proposta

Nel metodo candidato, ogni macro-categoria contiene più tecnologie COPERT con
pesi $w_{c,i}$ derivati da ACI e ISPRA. Il fattore medio viene calcolato come:

$$
EF_c(v)=\sum_i w_{c,i}\,EF_i(v)
$$

Il procedimento di costruzione dei pesi e i relativi limiti sono descritti nel
[`fleet crosswalk`](../../../docs/uc05-fleet-crosswalk-v0.1.md).

## Limiti del POC

- usa otto tecnologie illustrative con stock e attività uniformi;
- non rappresenta la composizione reale del traffico di Baronissi;
- verifica un solo nodo di velocità, 70 km/h;
- non applica i pesi ACI Salerno × attività ISPRA;
- non costituisce una prova dell'accuratezza scientifica degli EF;
- non misura concentrazioni atmosferiche e non stima la qualità dell'aria;
- deve essere ripetuto se cambiano versione COPERT, flotta, condizioni di guida
  o perimetro emissivo.

Per l'uso operativo servono profili approvati, una griglia di velocità
versionata, regole per l'interpolazione e un protocollo di validazione.
