link al corso https://app.pluralsight.com/library/courses/jmeter-getting-started

Cosa sono i performance tests?
	JMeter prima versione risale a 20 anni fa
	I test prestazionali DEVONO partire all'inizio di sviluppo, con esecuzioni durante tutta la fase dello sviluppo software
	i performance test DEVONO essere eseguito dopo i test funzionali, cioe' dopo che siamo sicuri che l'app funziona correttamente
	prestazioni sono misurati in termini di
		- tempo di risposta (tempo di richiesta + tempo elaborazione richiesta + tempo di risposta)
		- banda (throughtput), puo' essere misurato in num di transazioni, num richieste/risposte, num byte o kbyte..
		- affidabilita, in che modo vengono notificati errori, meno errori abbiamo meglio e'
		- scalabilita', dice come sistema scala di fronte ad un carica elevato (scalabilita' orizzontale)
	requisiti prestazionali
		- tempo medio di risposta
		- sistema deve supportare per es. 60pagine x 1sec
		- sistema deve supportare un num. di utenti (es. utente X max num transazioni 3000)
	Workflow nella fase di testing:
		i test vanno disegnati e implementati
		preparazione ambiente di test
			versione piu' vicina a quella di Prod
	Tipi di performance test
		- aumentando numero utenti
		- aumentando numero di richieste
		SMOKE TEST
			primo test da eseguire, carico normale, giusto x verificare se i test funzionano
		LOAD TEST
			test eseguito a diverso carico x monitorare il comportamento dell'applicazione
		STRESS TEST
			x testare il carico quando l'applicazione smette di funzionare in modo adeguato
			si puo' aumentare sia il numero di richieste che numero di utenti
		SPIKE TEST
			sono i test con periodici carichi elevati per verificare se l'app e' abbastanza robusta x supportare i picchi anomali
		ENDURANCE TEST
			un carico medlio/alto mantenuto x lungo periodo di tempo, x verificare se l'app ha memory leak o non chiude correttamente le connessioni di rete o a db
	Quando eseguiamo i performance test micuriamo i parametri
		tempo di risposta
		throughtput
		affidabilita'
		scalabilita'
	ma prima bisogna definire i requisiti prestazionali
	dopo di che possiamo
		disegnare e buildare i nostri test
		preparare l'ambiente di test
		eseguire i test
		analizzare il risultato
		ottimizzare
	con JMeter possiamo testare diversi server
		web, LDAP, email 
	possiamo definire le variabili 
	JMeter prevede un sistema di plugin x gestire le customizzazioni
Introduzione a JMeter
	JMeter funziona a livello di protocollo, simulando richieste di rete
	i limiti di carico che vogliamo generare sono dettati dal HW della macchina dalla quale andiamo a lanciare i test
	JMeter e' scritto in java -> usa java thread x simulare gli utenti.
	JMeter e' multi piattaforma e multi threaded
	requisiti di installazione
		non serve una macchina potente
		jmeter usa 1 thread x ogni utente che vogliamo simulare
		ok 4 core e 16GB di RAM, consente di simulare migliaia di utenti
		e' meglio separare jmeter dall'app che vogliamo testare, cioe' avere 2 macchine separate, in modo che due app non competono x le risorse
			x questo motivo e' meglio avere una connessione 1GB/sec (1 gigabit x sec)
		JMeter gira sulla JVM, dobbiamo prestare attenzione alle due cose
			1. e' meglio installare JDK e non JRE, entrambe hanno la JVM ma JDK puo' avere funzionalita' aggiuntive che servono a JMeter
			2. JMeter supporta specifiche versioni di Java, al 26/10/2019 ultima versione e' 5.1.1 e supporta java 8+, e' consigliato di usare o 
				Java8 o Java11, che sono LTS, installando cmq ultima minor x le ultime modifiche che possano riguardare la security
	installazione jmeter
		scaricare zip dal sito ufficiale
		struttura di cartelle di jmeter
			\bin - contiene eseguibili per lanciare jmeter
			\extras - funzionalita' aggiuntive, per es. generazione certificati
			\ilb - librerie necessarie x lanciare jmeter
			...cartelle x i docs...
	esecuzione jmeter
		modalita' UI, iterattiva
			x creare e validare gli script, eseguire script semplici
			laniare jmeter.bat
		modalita' script, linea di comando
			eseguire jmeter -n -t testFile.jmx
							-l (x specificare resultfile.csv)
							-j (x specificare logfile.log)
							-e -o (output folder x report)
							-h (help)
							-? (command line options)
			jmeter-n.cmd (usato per eseguire il file in modalita' command line)
			jmeter-n-r.cmd (usato per lanciare un file remoto)
			jmeter-server.bat (x lanciare jmeter in modalita' server, x il testing su piu' macchine)
			jmeter-t.cmd (x lanciare il test in commando mode)
			jmeterw.cmd (x lanciare jmeter in modalita' grafica, senza la console)
		configurazione dei plugin
			JMeter Plugins Code in Google Code (JP@GC), uno di piu' usati
			x installare un plugin copiare il jar sotto lib\ext
			o usare il jmeter plugin manager, vedi https://jmeter-plugins.org/, in questo modo noi mettiamo solo il jar di plugin manager sotto lib\ext
				e dopo abbiamo la voce Plugins Manager in JMeter x manipolare i plugin
			4 plugin consigliati dall'attore del corso
				Custom Thread Groups (opzioni aggiuntivi in configurazione dell'utente)
				3 Basic Graphs (grafici che riportano dati come tempo medio di risposta, threads attivit, transazioni ok/ko)
				Throughput shaping timer (configurazione RPS, garantisce Request Per Second)
				Dummy Sampler
			NOTA: vedi il sito di plugins x una descrizione 
		NOTA: jmeter non e' un browser! 
Configurazione di un Test Plan
	ogni piano e' organizzato in una struttura ad albero, albero di componenti
	ogni componente ha il suo scope, che definisce chi di altri componenti lo puo' chiamare, o quali altri componenti puo' chiamare lui stesso
		di solito il componente accede agli elementi di suo stesso livello o livelli sottostanti
		praticamente ogni voce/elemento dell'albero e' un componente (componente=elemento, sinonimi)
	vediamo in dettaglio i componenti di JMeter
		Test Plan
			componente root
			possiamo definire le variabili per tutto il test
			impostare se eseguire Thread Groups in modo seguenziale o parallelo (quando abbiamo piu' Thread Groups)
			tearDown thread groups sono i gruppi eseguiti alla fine di test  
			functional test mode - attivare solo x i test "piccoli", salva i dati di risposte e appesantisce molto il jmeter che richiede sempre piu' risorse
			aggiungiamo al classpath di jmeter le librerie che servono x eseguire i test
		Thread Groups
			e' l'entry point x nostri test
			ogni thread e' un utente che esegue i test, N threads = N utenti
			possiamo creare piu' thread groups, per ogni caso d'uso (per es. il gruppo di utenti standard, e un gruppo per utenti amministratori)
			abbiamo i vari threadGroups
				setup thread group - possiamo definire delle azioni da compiere prima che jmeter parte ad eseguire il thread group principale
				tearDown thread group - azioni da compiere alla fine 
				thread group - il gruppo principale 
					definiamo il numero di thread (che corrisponde al numero di utenti)
					se eseguire i test in modo ciclico, quanti cicli fare, si puo' lasciare anche l'impostazione di eseguire all'infinito
					configurazione dello scheduler, permette di definire il tempo di esecuzione del thread group (duration)
		ogni Thread Group puo' avere piu' configuration elements
			CSV Data Set Config - lettura di un file CSV contenente per esempio le variabili da impostare
			HTTP Header Manager - sovrascrittura di headers HTTP
			HTTP Cookie Manager - salva per ogni utente il cookie da inviare, si puo' aggiungere manualmente i cookie condivisi con piu' utenti
			HTTP Cache Manager - attivazione del meccanismo di caching, per esempio se il documento HTTP non e' cambiato, viene caricato dalla cache e la
				response body è vuota
			HTTP Request Default - definizione delle variabili di default condivisi con tutti gli elementi 
			NOTA: possiamo spostare gli elementi su diverse posizione nell'albero
		elementi Controllers
			Controllers sono i figli di Thread Groups
			ci sono due tipi di Controller
				Logic Controller
					permette customizzare la logica che definisce quando inviare le richieste, per es. quando una condizione e' true
					If Controller - determinazione quando eseguire i propri figli
					Transaction Controller - gestione tempo di esecuzione di tutti figli
						opzione 'Generate Parent Sample' serve per far apparire ogni richiesta del controller come figlia del controller stess
					Loop Controller - cicla i figli N volte
					While Controller - definizione condizione while di esecuzione 
				Sampler
					permette di definire la richiesta di un certo tipo
					x ogni Logic Controller possiamo aggiungere N Samplers
					i sampler piu' utilizzati sono
						HTTP Request (impostazione di una richiesta HTTP)
		Timers
			di default JMeter esegue i test senza pause
			i timer consentono introdurre un delay tra una richiesta e altra
			timer introduce un delay prima di ogni richiesta nel suo scope
			timer consente anche di simulare un carico reale sul server, evitando un carica non realistico
			timer puo' essere inserito in ogni livello dell'albero, si applicano ai Controller o Sampler in quali e' stato aggiunto
			se abbiamo piu' timer, jmeter somma il tempo e esegue la pausa lunga la somma di tempi
			ci sono vari timi di timer, vedi menu di jmeter
		Assertions
			consentono di validare le risposte, verificando i dati 
			possono essere inseriti a livello di Thread Group, Controller o Sampler
			non e' consigliato usare con file HTML e XML, sono pesanti da parsare
			Response Assertion 
				consente di validare per esempio la dimensione di una response
		Listeners
			consentono accedere alle risposte e metriche aggregate
			raccoglie informazioni al suo livello o livelli sottostanti
			il listener piu' usato e' View Results Tree
				visualizza albero di tutte le risposte ricevute durante i test
			Summary Report
				crea una tabella riepilogativa
			Aggregate Report
				simile al Summary Report, ma e' piu' computazionalmente pesante
			NOTA: i listener accedono agli stessi dati, quello che cambia e' la visualizzazione.  POSSANO usare tanta memoria se risposte sono tante. 
		Ordine di esecuzione e regole
			ordine e' dettato dalla gerarchia (albero) di elementi e dalla loro sequenza
			per esempio assertions sono eseguiti per ogni sampler al suo livello 
			in pocke parole i sampler sono delle richieste e assetional lavorano sulle risposte
			ordine di esecuzione del piano da parte di JMeter
				1. esecuzione elementi di configurazione
				2. Pre processors
				3. Timers
				4. Logic Controllers / Samplers
				5. Post processors
				6. Assertions
				7. Listeners
				ultime tre non sono eseguiti se non c'e' una risposta del server
				timers sono eseguiti solo se ci sono sampler a quali possano essere applicati
			regole degli Elementi di configurazione NOTA:
				- elementi di configurazione come User Defined Variables, sono processati all'avvio di test, indipendentemente da dove sono stati inseriti
				- elemento di configurazione nel ramo corrente ha una priorita' maggiore all'elemento del livello superiore 
				- elementi di configurazione di default sono mergati, invece i managers NO!
				- es. User Defined Variables sono applicati a tutti i controller indipendentemente da dove si trovano 
			regole generali
				elementi sono reimpostati in base all'ordine di esecuzione
				elementi dello stesso tipo sono eseguiti dopo agli stessi elementi di livelli superiori
				certi elementi sono eseguiti prima o dopo ogni sampler del proprio scope
				NOTA: nel piano di esecuzione possiamo avere un ordine diverso da quello di effettiva esecuzione, jmeter riordina elementi in base alla loro
				tipologia
				es. tutti timer dello stesso livello sono eseguiti prima di OGNI richiesta
				se vogliamo eseguire un time alla volta prima della richiesta rispettando l'ordine di posizionamento nel piano di esecuzione, dobbiamo
					usare l'elemento Think Time 
		Esecuzione dei test
			menu Run - Start / Start no pauses (senza eseguire i timer)
			validiamo il test prima di eseguirlo, jmeter esegue una iterazione e stampa nel View Result Tree il risultato
			possiamo fare clear su ogni listener x pulire la history 
			linciamo il run a livello di Thread Group
			oppure dalla linea di comando "jmeter -n -t "Test Plan.jmx"
Registrare i test
	possiamo usare Script Recorder (che fa da proxy server), configurando opportunamente il nostro browser (impostando il proxy), in modo che 
	proxy intercetta le richieste del browser effettuando un opportuna registrazione
	questo approccio automatizza la registrazione delle richieste lato jmeter che vogliamo usare per eseguire i successivi test dicarico
	e' consigliato usare firefox, che permette la configurazione di proxy dedicato al browser 
	configuriamo il proxy su firefox (puntiamo a localhost:8888)
	su chrome possiamo usare https://www.blazemeter.com/
	ATTENZIONE: firefox dalla versione 66 non proxa piu' indirizzi localhost, dobbiamo disabilitare questa opzione nelle impostazioni di firefox, oppure
		usare indirizzo IP
	la configurazione lato JMeter prevede l'utilizzo dei template di default 
	scegliamo il template Recording da File-Templates
	cliccando Crea, viene creato il nuovo Test Plan con elementi di default configurati gia' per eseguire la registrazione di richieste 
	aggiungiamo un timer in modo da evere una pausa tra una richiesta e altra
	cliccando Start sull'elemento HTTP(s) Test Script Recorder, parte la registrazione delle richieste
	nella finestra che si apre, scegliamo Transaction Name in base la funzionalita' che stiamo registrando (es. Home, Login etc.)
	per ogni registrazione viene creato un nuovo Transaction Controller con Sampler in base le richieste ricevute dal browser
	quando abbiamo finito con la registrazione fermiamo Script Recorder e validiamo il nostro test plan generato 
	la registrazione viene salvata nel file recording.xml 
	Organizzazione del recording script
		tasto dx su ogni trx controller - apply naming policy, consente rinominare i sampler seguendo una convenzione interna
		se apriamo la configurazione di Test Script Runner, possiamo aggiungere esclusioni suggeriti nel tab Request Filering
		se eliminiamo tutte le esclusioni, jmeter nel momento di registrazione creera una nuova richiesta per ogni richiesta inviata dal browser
	Registrazione richieste HTTPS
		dobbiamo usare JMeter HTTPS certificate, generato nel momento di avvio di Script Recorder
		il certificato generato viene salvato nella %JMETER_HOME%\bin, ed e' valido di default 7gg (e' cambiabile nel jmeter.properties)
		importiamo il certificato nel nostro browser come CA (Certificate Authority)
		rieseguiamo la chiamata
Rendere lo script dinamico (test dinamici)
	caricamento dati da un file CSV
		usiamo CSV Data Set Config Element per impostare il caricamento di username e password da un file *.csv
		referenziamo le variabili ${username} e ${password} nella richiesta di Login
	uso di Pre e Post processore
		usiamo CSS Selector (post processor element) x estrarre i dati dalla risposta HTTP 
		possiamo usare anche estratorre Regular Expression Extractor specificando l'espressione regolare 
		pre processore User Parameters, consente specificare i parametri da passare nella richiesta HTTP per esempio
	jmeter functions
		possa essere impostati per qualsiasi elemento del test
		usiamo il tool menu->tools->function helper tools
		esempio: nel contesto della nostra app possiamo usare le funzioni per generare dinamicamente il rating e il titolo di una recensione
		usiamo funzione Random x generare un numero da 1 a 5
		usiamo funzione Random String x generare una sringa variabile in base alla lunghezza e caratteri impostati
	come migliorare i test
		aggiungiamo Once Only Controller per le richieste alla pagina Home e Login
		aggiungiamo nuovo Transaction Controller x recuperare le categorie di prodotti e successivamente avere un prodotto random per eseguire la votazione
		aggiungiamo Gaussian Random Timer, applicato ad ogni richiesta, e' un timer gausiano che consente specificare le deviazione (es. 3000) e offset
			(es. 500) - tale impostazione genera un valore random tra 2500 e 3500
		aggiungiamo Response Assertion x verificare lo stato della risposta 
		per rendere i tests piu' realistici usiamo i thread group piu' avanzati
		installo il "Custom Thread Groups" plugin
			aggiungiamo Ultimate Thread Group al nostro Test Plan
			spostiamo tutti gli elementi sotto questo Thread Group eliminando dopo quello di prima
			configuration nuovo Thread Group impostando i valori come 
				num. di threads di partenza
				incremento su ogni iterazione
				pausa tra una e altra iterazione
	utilizzo di test fragments
		utili per esempio nel caso quando abbiamo piu' thread groups che condividono le chiamate come quelle alla pagina /Home e /Login
		test fragments contengono altri elementi con lo scopo di riutilizzarli
		ci sono due tipi di controller che possano referenziare i Tests Fragments
			- Module Conroller
				referenzia test fragments nello stesso Test Plan
			- Include Controller
				referenzia test fragments presenti in un file jmeter esterno
		aggiungiamo un nuovo Thread Group -> aggiungiamo al suo interno un Module Controller referenziando il Test Fragment contenente Home Contoller
		modifichiamo Custom Thread Group convertendo Home Controller in un Module Controller referenziando il Test Fragment di prima e eliminando 
			la Sampler presente
		in questo modo abbiamo due Thread Groups che puntano allo stesso Test Fragment che esegue la chiamata alla pagina Home
		possiamo salvare solo il Test Fragment come un file separato
		aggiungendo un Include Controller possiamo referenziando specificando il path al file 
		NOTA: possiamo lanciare i Thread Group uno alla volta (tasto dx - run)
Raccolta e analisi dei risultati di test
	le metriche importanti sono
		- Response time
		- Throughtput (num. di KBytes/sec) 
		- Error rate
	e' importante misurare le prestazioni lato server, monitorando
		- CPU
		- Memory
		- Network
	JMeter listeners consentono monitorare e analizzare le metriche menzionate prima
		View Results Tree
			fornisce una ricca visualizzazione dei risultati di test
			filtri diversi
			visualizzazione HTML delle risposte
			questo listener consuma parecchie risorse, e' consigliato di usarlo solo per debugging e validazione di Test Plans
			possiamo limitare la visualizzazione mostrando solo gli errori 
		Summary Report
			
		Aggregate Report
			i tempi sono riportati in milisecondi 
		i report stampano i dati riportando i nomi delle label - E' IMPORTANTE impostare questi nomi in modo da facilitare la successiva lettura dei risultati
		installiamo il plugin "3 Basic Graphs" per aggiungere altri listeners
			Active Threads Over Time
			Response Time
			Transactions per Second
	Monitoring Server Metrics
		Il plugin di riferimento e' PerfMon
		installiamo questo plugin dal Plugin Manager in JMeter
		installiamo PerfMon Server Agent sul server che vogliamo monitorare
			il link https://github.com/undera/perfmon-agent
		una volta che abbiamo tutto installato, aggiungiamo il listener di PerfMon al nostro Test Plan in JMeter
	JMeter Report Dashboard
		prima di eseguire il report dalla linea di comando e' meglio disabilitare tutti i listeners 
		e' consigliato in tutti Transaction Controllers togliere l'opzione "Generate Parent Sample"
			per avere un risultato piu' accurato
		il comando da lanciare e'
			jmeter -n -t recorded-test.jmx -l results.csv -e -o output
			la folder output contiene HTML dell'analisi generata
			results.csv contiene i dati usati per i test e risultati (possiamo aprire questo file all'interno di listeners usati nel nostro Test Plan)
		le metriche che vediamo sono:
			APDEX (Application Performance inDEX)
				misura la soddisfazione dell'utente tenendo conto del tempo di risposta
				classifica il tempo di risposta in tre categorie
					1. Satisfied (<=T, 2 secondi)
					2. Tolerated (>T and <= 4T)
					3. Frustrated (>4T)
				la formula x calcolare APDEX: (SatisfiedCount + (ToleratedCount/2)) / TotaleSamples
					= 0, nessun utente e' sodisfatto
					= 1, tutti utenti sono sodisfatti
			N grafici che ci mostrano le statische sull'andamento di threads, errori, etc..
		nel file bin/reportgenerator.properties possiamo customizzare la modalita' di generazione di questo report
		fare il riferimento alla documentazione ufficiale x vedere tutti i parametri customizzabili
	
		
		
		