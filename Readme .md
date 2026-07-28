# AI Dispečink — přehled stavu aplikace

Stav k 28. 7. 2026. Soubor: `ai-dispecink.html`, jeden soubor, 2 232 řádků, ~237 kB.
Běží po dvojkliku v prohlížeči, nic se neinstaluje, nic neodchází ven.

---

## K čemu to je

Trenažér, který učí volit **model, míru uvažování a vyhledávání na webu** podle typu úkolu.
Cíl není trefit nejsilnější model, ale nejlevnější, který úkol ještě zvládne.
Cena se počítá odhadem v eurech, aby byl rozdíl vidět v číslech, ne jen ve slovech.

## Šest záložek

| Záložka | Obsah |
|---|---|
| **Kde to pomůže** | Volba oboru, 10 karet použití s ukázkovým zadáním (rotují — 217 zadání, tlačítko „Ukázat jiné příklady"), „kde nepomůže", tabulka úspor času |
| **Trenažér** | Vlastní hra — směna 24 požadavků |
| **Přehled a ceny** | Modely daného prostředí, ceny za milion tokenů, převodní tabulka tříd napříč vendory |
| **Provozní pravidla** | Sedm pravidel (začni uprostřed, uvažování podle kroků, kontext před dražším modelem…) |
| **Kam co psát** | Prompt / projekt / trvalý postup / konektor |
| **Skills a agenti** | Claude Skill vs. ChatGPT Workspace Agent vs. Copilot Agent, se vzorovými zápisy |

## Jak trenažér funguje

1. **Obor** — 14 oborů + Namíchat všechno: Nákup a zásobování · Administrativa · Finance a účetnictví ·
   IT a provoz systémů · Vedení týmu · Sklad a logistika · Obchod a prodej · HR a personalistika ·
   Controlling a reporting · Daně · Pohledávky a fakturace · Bezpečnost a ochrana informací · IT vývoj ·
   Nemovitosti a správa budov
2. **Prostředí** — Claude / ChatGPT / Microsoft Copilot (přepne názvy modelů, přepínače uvažování i ceny)
3. **Režim** — Školení (verdikt a mikrolekce po každé odpovědi) nebo Ostrý test (vyhodnocení až na konci, export CSV)

Pak chodí požadavky. U každého se volí **model + míra uvažování + vyhledávání ano/ne**.

**Bodování:** 6 bodů za požadavek — 4 za třídu modelu, 1 za uvažování, 1 za vyhledávání.
**Razítka:** TREFA · TĚSNĚ · BEZ ZDROJŮ · PŘEPLACENO · PROJDE · SLABÉ · MIMO
**Ukazatele:** přesnost v procentech a útrata jako násobek optima, s přepočtem na měsíc, rok a deset lidí.

Nejlepší výsledek se ukládá do prohlížeče zvlášť pro každou kombinaci obor + prostředí + režim.

## Banka požadavků

**229 úloh** — úroveň 1: 68, úroveň 2: 91, úroveň 3: 70.
Z toho **oborové**: po deseti pro nákup, sklad, obchod, HR, controlling, daně, pohledávky, security,
IT vývoj a nemovitosti, po devíti pro administrativu, finance, IT provoz a vedení. IT vývoj navíc
sdílí devět úloh s IT provozem (`r:['it','vyvoj']`, celkem jich tedy vidí 19). Zbytek banky je společný.

Směna losuje 8 z každé úrovně. Skládá se ve třech vrstvách: nejdřív oborové úlohy, pak úlohy
z kategorií typických pro obor, zbytek ze společné banky. Pořadí se pak zamíchá.

---

## Kde se co edituje

Všechno podstatné je v konstantách na začátku `<script>`:

| Řádek | Konstanta | Co obsahuje |
|---|---|---|
| 399 | `ROLE` | Obory: název, popis, úvodní věta, prioritní kategorie, tabulka úspor |
| 522 | `PRIPADY_R` | Ukázková zadání na kartách, po oborech, v pořadí podle `PRIPADY` — 217 zadání, položka může být text nebo pole variant (náhodně se losuje; nákup 20, nemovitosti 17, ostatní po 15) |
| 745 | `PRIPADY` | 10 karet použití — téma, výchozí zadání, co získáte, pozor na |
| 791 | `KURZ` | Kurz USD → EUR (0,92) |
| 792 | `VSTUP_TOK` | Předpokládaný vstup na požadavek (12 000 tokenů) |
| 793 | `MESICNE` | Počet požadavků měsíčně pro přepočet (200) |
| 794 | `DELKA_SMENY` | Kolik požadavků z každé úrovně (8 → směna 24) |
| 796 | `PROSTREDI` | Claude / ChatGPT / Copilot: modely, ceny, přepínače uvažování, tabulka „kam co psát" |
| 867 | `TRIDY` | Názvy pěti tříd |
| 874 | `HINTY` | 10 mikrolekcí (`spousti` váže lekci na typ chyby) |
| 922 | `UKOLY` | Banka 229 požadavků |

### Formát jedné úlohy

```js
{u:2,                    // úroveň 1–3
 kat:'Data',             // kategorie (řídí oborové vážení)
 r:['finance'],          // nepovinné: jen pro tenhle obor; bez pole = pro všechny
 z:'Zadání požadavku…',
 k:'Kontext pod zadáním',
 best:['stredni'],       // třídy za plné body
 ok:['rychla','silna'],  // třídy za poloviční body
 ef:'med',               // správná míra uvažování: low / med / high
 efOk:['low'],           // přijatelná míra
 web:'ne',               // má být vyhledávání zapnuté? ano / ne
 proc:'Proč tahle třída…',
 dr:'Proč ne dražší…',
 tip:'Tip k zadání…'}
```

**Přidání úlohy:** zkopírovat blok, vložit do `UKOLY`, oddělit čárkou. Nic dalšího není potřeba —
kategorie i obory se dopočítají samy.

**Změna cen:** upravit `cin` / `cout` u modelu v `PROSTREDI` (USD za milion tokenů, přepočet kurzem).

**Přidání oboru:** nový záznam v `ROLE`, deset zadání v `PRIPADY_R`, volitelně úlohy s `r:['id']`.

---

## Ceny a předpoklady (k 27. 7. 2026)

Claude: Haiku 4.5 1/5 · Sonnet 5 3/15 · Opus 5 5/25 · Fable 5 10/50 (USD za milion tokenů)
ChatGPT: GPT-5.6 Luna 1/6 · Terra 2,50/15 · Sol 5/30
Copilot: platí se licencí, částky jsou odhad skutečné ceny výpočtu

Uvažování se účtuje jako výstup: low 800 · med 3 000 · high 12 000 tokenů.
Proto je vysoké uvažování běžně pětkrát až desetkrát dražší při stejném modelu.

---

## Otevřené věci

- **Provozní pravidla** a **Skills a agenti** používají příklady z nákupu a obor zatím neberou v potaz.
  Zvažovalo se je zvariantovat, zatím ponecháno jako společný základ.
- Banka se dá dál rozšiřovat po oborech (např. výroba, kvalita, právní).
- Zvažovalo se zmínit i osobní náklady (doma placené předplatné) v cenové poznámce nad trenažérem
  a v závěru směny — zatím je to jen v úvodním odstavci.
