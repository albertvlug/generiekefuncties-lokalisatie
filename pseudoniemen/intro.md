# In Jap en Jinneke taal…

Met een BSN kan de identiteit van een persoon makkelijk herleid worden.
Als een partij het BSN mag gebruiken, moeten ze dat zo min mogelijk doen. Dus zoveel mogelijk gebruik maken van pseudoniemen.
Als twee partijen die beide het BSN mogen gebruiken, gegevens over dezelfde persoon willen delen, kan dat als ze hetzelfde pseudoniem gebruiken. Dat willen we niet, want met een vast pseudoniem per BSN is het pseudoniem (relatief) makkelijk te kraken.  Dus willen we per partij (per zorgaanbieder) een ander pseudoniem voor dezelfde persoon / hetzelfde BSN
Als twee partijen gegevens aanleveren over dezelfde persoon / hetzelfde BSN, maar met ieder een ander pseudoniem, moeten die twee pseudoniemen ergens gekoppeld kunnen worden. Traditioneel zou hiervoor een trusted third partij worden ingezet, die een tabel bijhoudt op basis waarvan de twee pseudoniemen gekoppeld kunnen worden. Echter, deze koppeltabel is een privacy hotspot en zal goed beveiligd moeten worden. 
Polymorfe pseudonimisering (voor één persoon verschillende pseudoniemen per databron) is cryptografisch doorontwikkeld, waardoor een dergelijke privacy hotspot vermeden kan worden. De twee pseudoniemen van één persoon bij twee zorgaanbieders worden niet via een tabel gekoppeld, maar zodanig cryptografisch met elkaar verbonden, dat 
* ze niet (in een tabel of database) opgeslagen hoeven te worden, maar ter plekke gegenereerd kunnen worden
* 	ze versleuteld kunnen worden zodat alleen de ontvanger er iets mee kan
* 	ook partijen die geen BSN mogen verwerken, mee kunnen doen in de pseudonimisering, zonder het gebruik van BSN   

# De ‘concepten’ 
•	Een zorgaanbieder maakt voor iedere ingeschreven patiënt een PIP aan (o.b.v. BSN). Als Jan de patiënt/cliënt is, is de PIP van de huisarts van Jan een andere dan de PIP die de apotheek genereerd van Jan.
•	Met deze PIP kan een PS (polymorf) pseudoniem voor Jan worden aangemaakt door de huisarts en die is anders dan het (polymorf) pseudoniem voor Jan van de apotheek. Uit het PS kan geen BSN worden afgeleid. Dit PS wordt niet gebruikt in de communicatie. Het PS is niet te brute-forcen, want het staat bij de zorgaanbieder in een beveiligde omgeving en is voor één persoon bij iedere zorgaanbieder een ander nummer
•	Als een zorgaanbieder (bv. de huisarts) wil communiceren met een andere partij die ook het BSN mag gebruiken (bv. een apotheek), dan kan de huisarts het eigen PIP versleutelen naar een VI (versleutelde identiteit) die alleen door de apotheek is te gebruiken. Als de apotheek de VI ontsleuteld heeft, is niet het PIP, en ook niet het PS van de patiënt beschikbaar, maar het BSN! Voorwaarden voor deze decryptie: de apotheek moet hiervoor in het OIN register te boek staan als ‘BSN gerechtigd’ en de apotheek gebruikt voor de decryptie de private sleutel die (eerder) van BSN-K ontvangen is.
•	Als een zorgaanbieder (bv. de huisarts) wil communiceren met een andere partij die geen BSN mag verwerken (bv. de leverancier van een PGO), dan kan de huisarts het eigen PIP versleutelen naar een VP (versleuteld pseudoniem) die alleen door de betreffende DVP is te gebruiken. Als de DVP de VP ontsleuteld heeft, is er niet het PIP, en ook niet het PS, en zeker niet het BSN beschikbaar, maar het PS van (alleen) die DVP voor die patiënt.  
Privacy bescherming
De nummers die uitgewisseld worden (VP en VI) hoeven nergens opgeslagen te worden, omdat de betrokken partijen ze real-time kan (laten) genereren wanneer ze nodig zijn om data van een persoon op te halen of te koppelen. De nummers die wel opgeslagen worden (PS) zijn onherleidbaar naar persoon. 

Aanzet uitwerkingen
Setup / eerste configuratie
Stap 1: leverancier van zorgaanbieders meldt HSM aan bij BSN-K
•	AanmeldenHSM(KvK): HSMid

Stap 2: HSM meldt zorgaanbieder aan bij BSN-K
•	AanmeldenZorgaanbieder(OIN-ZA1,HSMid): PrivKeyZA1

Uitwerking per use case
I.	Primaire zorg, lokalisatie Waar 

•	ZA1: registratie bij lokalisatie index
o	TransformeerNaarVP(PIP@ZA1, LokalisatieIndex): VP@LokalisatieIndex
o	LokalisatieIndex.Registreer(VP@LokalisatieIndex)
•	LokalisatieIndex: Verwerken van registratie
o	Decrypt(VP@Index, PrivKeyIndex): Ps@LokalisatieIndex
o	Save(Ps@LokalisatieIndex)
•	ZA2: Raadplegen, stap1: lokaliseren
o	TransformeerNaarVP(PIP@ZA2, LokalisatieIndex): VP@LokalisatieIndex
o	LokalisatieIndex.Lookup(VP@LokalisatieIndex)
•	LokalisatieIndex: Lookup
o	Decrypt(VP@Index, PrivKeyIndex): Ps@LokalisatieIndex
o	Lookup(Ps@LokalisatieIndex)
o	return ZA1

I.	primaire zorg, use case uitwisselen (hier: opvragen) zonder BSN

•	ZA2: opvragen data bij ZA1
o	TransformeerNaarVI(PIP@ZA2, ZA1): VI@ZA1
o	GET fhir/patient?identifier=VI@ZA1|versleuteldeIdentiteit
•	ZA1: verzamelen data
o	Decrypt(VI@ZA1, PrivKeyBSN-ZA1) : BSN
o	GET internal/fhir/patient?identifier=BSN
•	ZA1: opleveren data
o	TransformeerNaarVI(PIP@ZA1, ZA2): VI@ZA2
o	return result patient?identifier=VI@ZA2|versleuteldeIdentiteit


 
Op basis van aanzet Steven:
Concepten 

•	PIP polymorf identiteit pseudoniem, meest generiek vorm
Specifiek voor de aanvragen de partij, bv authenticatiedienst obv bsn als stamgegeven.
Die krijgt een pip die bij dat bsn hoort, specifiek voor die authenticatiedienst
•	PS pseudoniem, niet te herleiden naar BSN
Reliant party, persistent, ongeacht welke authenticatiedienst een pip heeft aangevraagd, hetzelfde ps komt bij dezelfde authenticatie dienst uit
•	VP versleuteld pseudoniem, versleuteld voor de ontvanger, alleen die reliant party kan het uitpakken
•	VI versleutelde identiteit, ziet er ook steeds anders uit, maar er komt hetzelfde pseudoniem uit.

NB PIP , VP, VI hebben randomisatie, pseudoniem wat er na ontsleuteling uit komt is steeds hetzelfde
Hier is PIP, daar komt rol 02 bij maak VP of VI voor diverse clubs. Datum en stelselversie zijn velden die wel in VP en VI meekunnen, maar geen impact hebben op resultaat.

Operaties:
Activeren (BSN, ZA1): PIP@ZA1 		
Zorgaanbieder 1 zet BSN om bij BSN-K in een PIP

Afleiden PS uit PIP:
PIP eerst naar VP (obv OIN authenticatiedienst), VP kan alleen met private key van ZA1 ontsleuteld worden, dan PS er uithalen obv private key (die je van BSN-K hebt gehad)

Communiceren met anderen obv pseudoniem: 
Vraag aan HSM: Transformeer pip voor ZA1 (mezelf) en voor ander. Die twee VP’s zijn dan aan elkaar gelinkt 
BV: ZA1 vraagt obv PIP een VP voor zichzelf VP1 en voor index VP2. ZA2 vraagt obv PIP een VP voor zichzelf (VP3) en voor index (VP4). Dat zijn alleemaal verschillende VPs. De index kan VP2 en VP4 decrypten en daar komt dezelfde PS uit.

Communiceren met anderen obv VI
Transformeer naar VI, decrypt terug naar PS en check of er BSN uit komt.

PIP is versleuteld, die kan je niet ontsleutelen, alleen transformeren. Dus VP of VI nodig. Om te voorkomen dat 2 zorgaanbieders dezelfde code krijgt, worden ze encrypt, zodanig dat alleen de ontvanger het terug kan halen.  


In bijzondere situaties kan je vanuit BSN direct naar VP of VI gaan. Direct VP en direct VI
Stel we gebruiken BSN-K (authenticatie stelsel)
Dan hoeven we als zorg niet een apart stelsel op te tuigen, we kunnen in de diverse berichten bij PATID een VP vullen 
Wat is verschil tussen bsn-verwerkende partij en niet-bsn verwerkende partij?
RSIG gaat over BSN gerechtigden. Van elk OIN is bekend of je bsn gerechtigd bent.
Voor het onsleutelen van VI heb je iets nodig dat je niet krijgt als je geen bsn-gerechtigde bent.

Als je geen bsn mag verwerken, kan je 

Uitrol/implementatie stapsgewijs:
Hybride: je kan desgewenst PIP introduceren en voor zorgaanbieders die nog niet zo ver zijn, of met oude pseudoniemen werken, ook BSNs of  

