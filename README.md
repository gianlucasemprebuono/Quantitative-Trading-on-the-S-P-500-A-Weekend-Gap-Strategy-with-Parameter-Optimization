📈 Analisi Quantitativa e Backtasting su S&P 500: Strategia di Trading Weekend-Based📈

Ciao,sono Gianluca Semprebuono, studente magistrale in Financial Risk and Data Analysis presso La Sapienza, Università di Roma.

Voglio condividere con voi un progetto in Python che sto sviluppando, focalizzato sull'analisi quantitativa applicata allo S&P 500.

Il codice implementa una strategia di trading strutturata in più fasi, che combinano analisi tecnica, analisi degli scenari di gap tra venerdì e lunedì, conferma pre-market e simulazione/backtest con gestione dinamica del rischio e ottimizzazione dei parametri. Di seguito viene spiegata esaustivamente la logica e il flusso della strategia:

1. Acquisizione dei Dati e Calcolo degli Indicatori Tecnici
Scaricamento dei dati

La funzione download_data utilizza la libreria yfinance per scaricare i dati giornalieri (prezzo di apertura, chiusura, massimo, minimo, volume) per un dato ticker (in questo esempio lo S&P500 con ticker ^GSPC) tra due date specificate.
Calcolo degli Indicatori

Medie Mobili (SMA):
SMA_10 e SMA_50 sono calcolate come la media mobile semplice dei prezzi di chiusura su finestre di 10 e 50 giorni, utili per identificare trend a breve e medio termine.
RSI (Relative Strength Index):
Calcolato su 14 periodi, misura la forza e la velocità delle variazioni dei prezzi, evidenziando condizioni di ipercomprato o ipervenduto.
MACD e Signal Line:
Il MACD è la differenza tra l’EMA a 12 e 26 periodi, mentre la linea di segnale è una media esponenziale (9 periodi) del MACD, utile per individuare segnali di inversione.
Bollinger Bands:
La banda centrale è la SMA a 20 giorni; le bande superiore e inferiore si ottengono aggiungendo e sottraendo due volte la deviazione standard, per valutare la volatilità.
ROC (Rate of Change):
Misura la variazione percentuale dei prezzi su 12 periodi.
OBV (On Balance Volume):
Un indicatore che integra il volume con il movimento dei prezzi, accumulando il volume in base alla direzione del prezzo.
ATR (Average True Range):
Indica la volatilità media su 14 giorni, utilizzato poi per definire stop loss dinamici.
2. Creazione del Dataset Friday-Monday e Definizione degli Scenari
Accoppiamento dei giorni

La funzione pair_friday_monday crea dei coppie in cui per ogni lunedì viene associato il venerdì immediatamente precedente.
Per ciascuna coppia si registrano dati come:
Per lunedì: apertura, massimo, minimo, chiusura, volume.
Per venerdì: solo il prezzo di chiusura.
Definizione degli Scenari

Utilizzando la funzione assign_scenario, ogni coppia viene classificata in quattro scenari in base alla relazione tra il gap (differenza tra il venerdì e il lunedì) e l’andamento intraday del lunedì:
Scenario 1 (Bullish): Il prezzo di chiusura di venerdì è inferiore all’apertura di lunedì e il lunedì chiude in rialzo (close > open).
Scenario 2 (Bearish): Venerdì chiude sotto l’apertura di lunedì, ma il lunedì chiude in ribasso.
Scenario 3 (Bullish): Venerdì chiude sopra l’apertura di lunedì e il lunedì è rialzista.
Scenario 4 (Bearish): Venerdì chiude sopra l’apertura di lunedì e il lunedì è ribassista.
Inoltre, viene previsto un filtro per escludere coppie in cui sono presenti eventi macroeconomici ad alto impatto (utilizzando un placeholder per l’integrazione con un calendario economico).
3. Integrazione degli Indicatori Tecnici e Regressione
Merge dei Dati

La funzione merge_indicators_to_pairs unisce i dati delle coppie Friday-Monday con gli indicatori tecnici, associando ad ogni lunedì i valori degli indicatori calcolati in precedenza.
Modello di Regressione

Con perform_regression, viene creata una variabile target binaria “Success” (1 se il lunedì chiude in rialzo, 0 altrimenti).
Viene usata una regressione lineare con indicatori selezionati (SMA_10, SMA_50, RSI, MACD, MACD_Signal, ROC, OBV) per valutare la relazione tra gli indicatori e il successo del trade.
Il modello genera un Indicator_Score che viene poi normalizzato su una scala 0-1, fornendo un punteggio che rappresenta la “forza” predittiva degli indicatori per il trade.
4. Analisi del Pre-Market e Calcolo della Trade_Probability
Analisi Pre-Market

La funzione analyze_pre_market scarica dati a intervalli di 1 minuto per il pre-market (8:00-9:30 AM EST) per ogni lunedì, analizzando:
Il volume totale nel pre-market.
La tendenza (bullish se il prezzo di chiusura supera l’apertura, bearish se è il contrario, neutral in caso di parità).
Calcolo della Trade_Probability

La funzione compute_trade_probability combina:
Il punteggio normalizzato degli indicatori (Indicator_Score_Norm).
La frequenza degli scenari (cioè quanto spesso si verifica un dato scenario nella storia, utile come proxy probabilistica).
Se sono disponibili dati pre-market e il volume supera una soglia, il punteggio viene corretto:
Viene aumentato (fattore di conferma) se il trend pre-market conferma l’aspettativa dello scenario.
Viene ridotto (fattore di contraddizione) se il trend non è in linea con l’aspettativa.
5. Simulazione della Strategia: Backtesting, Equity Curve e Gestione del Rischio
Simulazione dei Trade

Le funzioni backtest_strategy_dynamic, simulate_equity_curve_dynamic e simulate_trades_details_dynamic simulano l’esecuzione dei trade in base a una soglia minima di Trade_Probability.
Il calcolo del rendimento del trade include:
Take Profit (TP) e Stop Loss (SL):
Se il rendimento lordo supera il TP o scende sotto il livello di SL, viene fissato il profitto o la perdita.
Stop Loss Dinamico:
Se attivato, utilizza il valore dell’ATR (volatilità) per adattare dinamicamente il livello di SL.
Allocazione Dinamica del Capitale:
La percentuale di capitale investita può variare in base alla Trade_Probability, aumentando l’esposizione in presenza di segnali più forti.
Leva Finanziaria:
Il calcolo del trade size include un fattore leva (es. 50x) per amplificare i guadagni o le perdite.
Durante la simulazione vengono inoltre calcolati metriche di performance come:
Final Capital, Profitto Totale, Percentuale di Successo.
Max Drawdown: La massima perdita dal picco di equity.
Sharpe Ratio: Basato sui log return per valutare il rapporto rischio/rendimento.
6. Ottimizzazione dei Parametri: Grid Search e Bayesian Optimization
Grid Search

Viene eseguita una ricerca esaustiva su una griglia di parametri (percentuale di investimento, TP e SL) per identificare la combinazione che massimizza il capitale finale nel backtest.
I risultati della grid search forniscono una “strategia migliore” in termini di performance storica.
Ottimizzazione Bayesiana

Utilizzando la libreria bayes_opt, viene definita una funzione obiettivo che esegue il backtest e restituisce il capitale finale.
I parametri ottimizzati (invest_pct, TP, SL) vengono ricercati in modo più efficiente rispetto al grid search, monitorando anche la convergenza attraverso un grafico.
7. Esecuzione e Output Finali
Flusso Principale

Il blocco if __name__ == "__main__": coordina l’intero processo:
Scaricamento e Preparazione: Dati storici e indicatori tecnici vengono scaricati e calcolati.
Dataset Friday-Monday: Vengono accoppiati i dati di venerdì e lunedì, classificati per scenario e filtrati per eventi macroeconomici.
Integrazione e Regressione: Gli indicatori vengono uniti ai dati dei trade e la regressione lineare produce il punteggio predittivo.
Analisi Pre-Market e Calcolo Trade_Probability: Si integra l’analisi del pre-market per confermare o smentire la validità dello scenario.
Backtest e Simulazione: Vengono simulati i trade con gestione dinamica del rischio e visualizzata l’equity curve.
Ottimizzazione: Sia il grid search che l’ottimizzazione bayesiana vengono utilizzati per affinare i parametri della strategia.
Output Finali: Vengono stampate statistiche, salvati file CSV con i dettagli dei trade e tracciati grafici dell’equity curve e della convergenza dell’ottimizzazione.
Riassunto della Strategia
Analisi Tecnica e Gap Weekend:

Vengono sfruttati indicatori tecnici (SMA, RSI, MACD, Bollinger Bands, ROC, OBV, ATR) per valutare trend e volatilità.
La logica si concentra sui gap tra il venerdì e il lunedì, classificando gli scenari in base alla direzione del gap e al comportamento intraday del lunedì.
Modello Predittivo:

Una regressione lineare stima la probabilità di successo (trade vincente) basandosi sui valori degli indicatori tecnici, creando un punteggio normalizzato (Indicator_Score_Norm).
Conferma Pre-Market:

L’analisi del pre-market integra un ulteriore livello di conferma, correggendo il punteggio se il trend pre-market è in linea (o contrario) con lo scenario atteso.
Gestione Dinamica del Rischio e Allocazione:

La strategia adotta stop loss dinamici basati sull’ATR e un’allocazione variabile del capitale in funzione della forza del segnale, con leva finanziaria per amplificare gli effetti.
Ottimizzazione e Backtesting:

Tramite grid search e ottimizzazione bayesiana vengono testati e affinati i parametri chiave (percentuale d’investimento, TP, SL) per massimizzare le performance storiche, monitorando metriche come il capitale finale, il max drawdown e lo Sharpe Ratio.






Codice B:

Esegue una grid search semplice testando tutte le combinazioni possibili sui lunedì, scegliendo la combinazione che massimizza il capitale finale.Adotta una strategia più semplificata, concentrandosi sui lunedì, simulando i trade con spread e slippage e ottimizzando i parametri in maniera diretta e statica.
