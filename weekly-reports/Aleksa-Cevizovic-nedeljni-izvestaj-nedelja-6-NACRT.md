# Nedeljni izveštaj — NACRT

## 1. Osnovne informacije

| Polje | Odgovor |
| --- | --- |
| Ime i prezime | Aleksa Cevizovic |
| Adresa e-pošte | aleksacevizovic@gmail.com |
| Discord korisničko ime | Aleksa Cevizovic |
| Nedelja | Nedelja 6 |
| Par / tim | Tim 6 - Aleksa Cevizovic i Anja Hubac |
| Moj konkretan doprinos / uloga | Posao smo podelili napola. Moj deo je bio ono što se dešava posle retrieval-a: citati u odgovoru (Outcome 3), odbijanje pitanja bez dovoljno dokaza (Outcome 4) i finalni par primera (Outcome 5). Od ekstenzije sam vodio stemming. Pregledao sam i proverio urađeno. Anja je vodila chunking i retrieval (Outcome 1 i 2), chunking po rečenicama, okruženje i testove, a AI sesije su pokretane na njenom računaru. |
| Datum predaje | 2026-10-14 |
| Reference na rad i dokaze | Dokazi su u `evidence.md`: Outcome 3–5 i deo o stemming-u u sekciji 6, sa tačnim komandama i izlazima. `./scripts/run_tests.sh` prolazi sa 57 testova. |

## 2. Moj status

**Status:** Završeno

Domaći je završen: svi outcome-i i ekstenzija su urađeni i imaju dokaze. Iz checklist-e nije označen samo demo lead, jer ove nedelje nismo prisutni ni Anja ni ja, pa ga nismo odredili.

## 3. Rad ove nedelje

Ove nedelje smo radili na RAG domaćem. Dobili smo mali Python projekat sa korpusom od tri dokumenta i trebalo je da pokažemo ceo tok: dokumenti se dele na chunk-ove, za pitanje se pronađu najrelevantniji chunk-ovi, a sistem odgovara samo na osnovu njih, sa citatima, ili odbija ako u njima nema odgovora. Nema LLM poziva; retrieval je leksički, a odgovor citira tekst pronađenih chunk-ova.

Moj deo je bio da odgovor bude pošten prema izvorima.

Kod citata (Outcome 3) pravilo je da se citira samo chunk koji zaista doprinosi odgovoru, odnosno koji pokriva neki pojam iz pitanja koji prethodni chunk-ovi nisu pokrili. Na primer, za „What is retrieval augmented generation?“ pronađena su dva chunk-a, ali se citira samo `doc-1-chunk-0`, jer `doc-2-chunk-0` sadrži samo reč „retrieval“, koja je već pokrivena. Odgovor i References sekcija su odvojeni, a svaka referenca ima chunk ID, naslov i dokument.

Kod odbijanja (Outcome 4) zanimljiv slučaj je „Who invented retrieval augmented generation?“. Tema se poklapa sa prvim dokumentom, ali nigde ne piše ko je izmislio RAG, pa sistem odbija jer reč „invented“ ne postoji u pronađenom kontekstu. Za potpuno nepovezano pitanje, o Svetskom prvenstvu 1958, retrieval je prazan i odgovor je isto odbijanje. U Outcome 5 su ova dva tipa pitanja prikazana jedan pored drugog, sa komandom i punim izlazom.

Stemming je ekstenzija koja rešava jedan konkretan problem: pitanje „What does metadata help with?“ bilo je odbijeno bez obzira na podešavanje chunking-a, jer „help“ i „helps“ nisu ista reč. Sa opcijom `--stem` reči se svode na osnovu uz nekoliko jednostavnih pravila za nastavke (-ies, -s, -ing, -ed, završno -e), skor raste sa 0.10 na 0.21 i dobija se odgovor sa citatom. Važno je da se stemming primenjuje na isti način na tri mesta: u skoru, u proveri za odbijanje i u izboru citata. Proverili smo i da „Who invented RAG?“ i dalje biva odbijeno.

Za ekstenziju je dodato 28 testova, ukupno ih je 57 i svi prolaze. Komande iz `evidence.md` su ponovo pokrenute nad trenutnim kodom i izlazi se poklapaju sa onim što piše u dokumentu.

Ograničenja ostaju. Stemmer ne zna nepravilne oblike (ran/run) i nekad previše skrati reč (retrieve postaje retriev). Odbijanje je zasnovano na rečima, a ne na značenju, pa sistem može da odbije tačno postavljeno pitanje sa drugim rečima, a to što se sve reči poklapaju još ne znači da kontekst stvarno odgovara na pitanje.

Kod, testovi i dokumentacija su pisani uz OpenAI Codex i Claude Code, u sesijama koje su pokretane kod Anje.

Rad smo proveravali i pokretali zajedno kod Anje, na njenom računaru. Najviše mi je koristilo što sam bolje razumeo kako se chunking i stemming koriste i video kako rade u praksi: kako različita podela teksta menja skor i odgovor, i kako stemming pomaže da se poklope različiti oblici iste reči.

## 4. Sledeći korak

Nemam planiran sledeći korak za ovaj domaći.

## 5. Poverljiva napomena za tutora

Nema dodatne napomene.
