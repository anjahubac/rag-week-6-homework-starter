# Nedeljni izveštaj — NACRT

## 1. Osnovne informacije

| Polje | Odgovor |
| --- | --- |
| Ime i prezime | Anja Hubač |
| Adresa e-pošte | anja.hubac@gmail.com |
| Discord korisničko ime | Anja |
| Nedelja | Nedelja 6 — RAG |
| Par / tim | Tim 6 - Aleksa Čevizović i Anja Hubač |
| Moj konkretan doprinos / uloga | Aleksa i ja smo odgovornost podelili na jednake delove. Ja sam vodila Outcome 1 (uticaj chunking-a, konfiguracije A–C), Outcome 2 (top-k retrieval, Dice skor i proračun rangiranja), ekstenziju za chunking po rečenicama, podešavanje okruženja i pokretanje testova. Obe AI sesije (OpenAI Codex, pa Claude Code) pokrenula sam na svom računaru. Aleksa je vodio Outcome 3 (citati i References), Outcome 4 (odbijanje), Outcome 5 (finalni par dokaza) i ekstenziju za stemming, i pregledao je i proverio urađeno. |
| Datum predaje | 2026-10-14 |
| Reference na rad i dokaze | Dokazi su u fajlovima projekta: `evidence.md` (Outcome 1–5 i sekcija 6 — ekstenzija, sa komandama i punim izlazima), `docs/homework-checklist.md` (22 od 23 stavke), `README.md` (sekcija o ekstenziji). Provera: `./scripts/run_tests.sh` → 57 passed (Python 3.9.6, `pytest==8.3.3`). |

## 2. Moj status

**Status:** Završeno

Outcome 1–5 i ekstenzija su gotovi, a 22 od 23 stavke checklist-e su označene. Neoznačen je samo demo lead — ove nedelje smo oboje odsutni, što smo zapisali u `evidence.md`.

## 3. Rad ove nedelje

Zadatak za šestu nedelju bio je da na gotovom Python projektu pokažemo osnovne RAG koncepte: pripremu korpusa, chunking sa overlap-om, top-k retrieval, odgovore sa citatima chunk ID-jeva i odbijanje pitanja na koja korpus nema odgovor. Dokaze smo predali u `evidence.md` po zadatoj strukturi, a `docs/homework-checklist.md` smo koristili kao smernicu.

Početni kod je imao nekoliko problema koje sam prvo ispravila uz OpenAI Codex. Starter je odgovarao na „Who won the world cup in 1958?“ jer su se poklapale obične reči kao „the“ i „in“, ponovljene reči su veštački podizale skor, a citati su navodili sve pronađene chunk-ove. Uveli smo listu stop-reči, Dice sličnost `2 * shared / (query_terms + chunk_terms)`, odbijanje kada neki pojam iz pitanja ne postoji u pronađenom kontekstu, citate samo za chunk-ove koji dodaju nove pojmove i posebnu References sekciju. CLI sada ispisuje konfiguraciju, rangirane chunk-ove, skorove i tekst, što je bilo potrebno za dokaze.

Moj deo u dokazima:

- **Outcome 1 — uticaj chunking-a.** Isto pitanje sam pokrenula sa `chunk_size=70` i menjala samo overlap. Bez overlap-a reč „overlap“ se deli na „over“ i „lap“ i sistem odbija pitanje; sa `overlap=20` reč ostaje cela, skor raste sa 0.40 na 0.80 i odgovor ima citat.
- **Outcome 2 — top-k retrieval.** Primeri sa `top_k=2` i `top_k=3` i ručno izračunati Dice skorovi koji potvrđuju redosled.
- **Ekstenzija — chunking po rečenicama** (`--strategy sentence`). Chunk-ovi se prave od celih rečenica, a overlap se meri u celim rečenicama. Za isto pitanje skor je 0.92 i citira se čista rečenica umesto teksta koji počinje usred rečenice. Prva verzija je predugačku rečenicu sekla po karakterima i napravila chunk koji sadrži samo „.“ — test je to uhvatio, pa se sada predugačke rečenice dele na granicama reči.

Pre svega ovoga morala sam da sredim okruženje: originalni `.venv` je pokazivao na Python 3.12 koji ne postoji na mom računaru, pa sam napravila novi `.venv-local` (Python 3.9.6). Broj testova je rastao sa 9 na 29 posle Codex-a i na 57 sa ekstenzijom; svi prolaze. Pred predaju je svaka komanda iz `evidence.md` pokrenuta još jednom, da izlazi u dokumentu odgovaraju trenutnom kodu.

Šta ne radi dobro: chunking po karakterima seče reči i rečenice, chunking po rečenicama „e.g.“ vidi kao kraj rečenice, a pošto je poklapanje leksičko, sinonimi i parafraze se ne prepoznaju.

AI sam koristila kroz OpenAI Codex, a zatim kroz Claude Code. Codex je ispravio retrieval, odbijanje, citate i CLI i napisao prvu verziju `evidence.md`. Claude Code mi je posebno pomogao da razumem šta zapravo radimo: objasnio mi je kako RAG funkcioniše i prošao sa mnom kroz ceo projekat fajl po fajl, od korpusa i chunking-a do retrieval-a, skora i odbijanja. Posle Codex-ove implementacije mi je i lepo obrazložio šta je tačno Codex promenio i zašto. Nakon toga je implementirao ekstenziju i dopunio dokumentaciju. Svi izlazi u `evidence.md` su iz stvarnih pokretanja.

(**NEDOSTAJE:** tvojim rečima — šta si ti lično odlučila, pregledala ili promenila u odnosu na predloge AI alata; jedna teškoća ili stvar koju si naučila o RAG-u; da li ti treba pomoć od tutora ili „pomoć trenutno nije potrebna“)

## 4. Sledeći korak

(**NEDOSTAJE:** jedna konkretna akcija sa jasnim završetkom, npr. „Pre druge sesije proveriću sa tutorom da li je demo lead obavezan i, ako jeste, upisaćemo ga u `evidence.md`.“)

## 5. Poverljiva napomena za tutora

Nema dodatne napomene.
