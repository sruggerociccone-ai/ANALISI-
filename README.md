# ANALISI-
ANALISI PY
import csv
import sys
import os

def converti_popolarita(valore):
    if valore is None:
        return None
    
    VALORESTRINGA = str(valore).strip()
    if VALORESTRINGA == "":
        return None
    
    try:
        NUMFLO = float(VALORESTRINGA)
        NUMINT = int(NUMFLO)
        
        if NUMINT < 0 or NUMINT > 100:
            return None
            
        return NUMINT
    except ValueError:
        return None

def report_qualita(totali, valide, vuote, sbagliate):
    print("\n=== REPORT QUALITÀ DATI ===")
    print(f"Righe totali analizzate:   {totali}")
    print(f"Righe VALIDE (buone):      {valide}")
    print(f"Righe con dato mancante:   {vuote}")
    print(f"Righe con dato sbagliato:  {sbagliate}")
    
    somma_controllo = valide + vuote + sbagliate
    print(f"Somma di controllo:        {somma_controllo}")
    if somma_controllo == totali:
        print("-> Verification superata: I conti tornano perfettamente.")
    else:
        print("ATTENZIONE: I conti non tornano!")

def leggi_brani(percorso):
    TOTALIRIGHE = 0
    BUONE = 0
    VUOTE = 0
    SBAGLIATE = 0
    BRANI = []

    try:
        with open(percorso, mode='r', encoding='utf-8') as f:
            PULITE = (linea.rstrip('\n\r').rstrip(';') for linea in f)
            LETTORE = csv.DictReader(PULITE, delimiter=',')
            
            for riga in LETTORE:
                TOTALIRIGHE += 1
                valore_pop = riga.get('popularity') 
                
                if valore_pop is None or valore_pop.strip() == "":
                    VUOTE += 1
                else:
                    risultato = converti_popolarita(valore_pop)
                    if risultato is not None:
                        BUONE += 1    
                        riga['popularity'] = risultato
                        BRANI.append(riga)
                    else:
                        SBAGLIATE += 1
                        
    except FileNotFoundError:
        print(f"Errore critico: Il file '{percorso}' non è stato trovato.")
        sys.exit(1)

    return BRANI, TOTALIRIGHE, BUONE, VUOTE, SBAGLIATE

def conta_per_genere(brani):
    conteggi = {}
    for brano in brani:
        genere_greggio = brano.get("track_genre") or "Sconosciuto"
        genere = genere_greggio.strip(";").replace('"', '').strip()
        
        if not genere:
            genere = "Sconosciuto"
            
        conteggi[genere] = conteggi.get(genere, 0) + 1
    return conteggi

def analizza_artisti_e_generi(brani):
    conteggio_brani_artista = {}
    generi_per_artista = {}
    righe_artisti_multipli = 0

    for brano in brani:
        artisti_greggi = brano.get("artists") or ""
        genere = brano.get("track_genre") or "Sconosciuto"
        
        if ";" in artisti_greggi:
            righe_artisti_multipli += 1
            lista_artisti = [a.strip() for a in artisti_greggi.split(";") if a.strip()]
        else:
            lista_artisti = [artisti_greggi.strip()] if artisti_greggi.strip() else ["Sconosciuto"]

        for artista in lista_artisti:
            conteggio_brani_artista[artista] = conteggio_brani_artista.get(artista, 0) + 1
            
            if artista not in generi_per_artista:
                generi_per_artista[artista] = {}
            generi_per_artista[artista][genere] = generi_per_artista[artista].get(genere, 0) + 1

    return conteggio_brani_artista, generi_per_artista, righe_artisti_multipli

def main():
    cartella = os.path.dirname(os.path.abspath(__file__))
    percorso_file = os.path.join(cartella, "dataset.csv")
    
    print("Avvio analisi del dataset...")
    # Carichiamo i dati e riceviamo anche le metriche di qualità
    elenco_brani, tot, val, vuo, sbag = leggi_brani(percorso_file)
    
    # Il main coordina e stampa il report di qualità come richiesto dal professore
    report_qualita(tot, val, vuo, sbag)
    
    # --- TASK 3: Generi Ordinati ---
    conteggi_genere = conta_per_genere(elenco_brani)
    generi_ordinati = sorted(conteggi_genere.items(), key=lambda x: x[1], reverse=True)
    print(f"\n[TASK 3] Primi 5 generi per numero di brani:")
    for gen, num in generi_ordinati[:5]:
        print(f"  - {gen}: {num} brani")

    # --- TASK 4 & 5: Analisi Avanzata Artisti ---
    contatore_artisti, mappa_generi, righe_multiple = analizza_artisti_e_generi(elenco_brani)
    
    print(f"\n[TASK 4] Righe totali con artisti multipli (;): {righe_multiple}")
    
    top_10_artisti = sorted(contatore_artisti.items(), key=lambda x: x[1], reverse=True)[:10]
    print("\n[TASK 4] Top 10 Artisti con più brani (considerando i singoli componenti):")
    for i, (artista, num_brani) in enumerate(top_10_artisti, 1):
        print(f"  {i}. {artista} ({num_brani} brani)")

    print("\n[TASK 5] Genere prevalente per i Top 5 artisti:")
    top_5_artisti = top_10_artisti[:5]
    for artista, _ in top_5_artisti:
        diz_generi = mappa_generi[artista]
        genere_top = max(diz_generi, key=diz_generi.get)
        print(f"  - {artista} -> Genere Prevalente: '{genere_top}' ({diz_generi[genere_top]} brani)")

if __name__ == "__main__":
    main()
