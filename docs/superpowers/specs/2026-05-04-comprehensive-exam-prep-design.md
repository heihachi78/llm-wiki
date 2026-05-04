---
title: "Comprehensive Exam Prep Analysis (HU) — Design Spec"
date_created: 2026-05-04
date_updated: 2026-05-04
status: approved
---

# Comprehensive Exam Prep Analysis (HU) — Design Spec

## 1. Cél

Egy önálló magyar nyelvű, esszé-stílusú vizsgafelkészítő tananyag, amely
a `felho-prog` wiki tudásanyagából egységes, lineárisan tanulható
dokumentumot szintetizál. A diák a dokumentumból közvetlenül vizsgára
készülhet anélkül, hogy a teljes wiki-tartalmat egyenként végig kelljen
böngésznie.

A cél **nem** a wiki helyettesítése — a cél egy olyan átfogó analízis-oldal,
amely az `analyses/` családba illeszkedik, és önállóan tanulható tananyagot
kínál a kurzus minden vizsgára kerülő témaköréből.

## 2. Hatókör — kötelező témakörök

### 2.1 Felhasználó által megjelölt kulcstémák

A dokumentumnak kötelezően ki kell térnie az alábbiakra:

1. A felhőrendszerek és felhőalapú számítástechnika alapjait megalapozó
   technológiák.
2. Nagyméretű elosztott rendszerek erőforrás-kezelési problémái, dinamikusan
   változó terhelési szintek mellett.
3. Szolgáltatásorientált számítástechnikai fogalmak.
4. A felhőtechnológiai piramis szolgáltatási rétegei (IaaS / PaaS / SaaS /
   FaaS): mit nyújtanak, ajánlott use case, előnyök/hátrányok.
5. Kötegelt feldolgozás Hadoop/MapReduce stílusban — Hadoop klaszter
   architektúrája, MapReduce programozási modell és alkalmazási területei.
6. Adatfolyam-feldolgozás alapjai és felhasználási esetei — előnyök,
   kihívások, domináns frameworkek, ablakozás.
7. Serverless / felhőfüggvények — fogalom, típusok, indítás, összekapcsolás,
   Pub/Sub eseménykézbesítés alapjai.
8. Felhőalapú adattárolás és adatkezelés — különböző tároló- és
   adatbázis-szolgáltatások, valamint tulajdonságaik (struktúra, biztonság,
   titkosítás, replikáció, skálázhatóság stb.).

### 2.2 Wikiből hozzáadott kiegészítő témák

A wiki azon témái, amelyeket a 2.1 lista nem fed le, de oktatási értékkel
bírnak és bekerülnek a dokumentumba:

- Felhőtelepítési modellek (public / private / hybrid / community).
- NIST referenciamodell — szereplők (Consumer, Provider, Broker, Auditor,
  Carrier).
- Földrajzi hierarchia (global / regional / zonal) és hibatűrési zónák.
- A felhő történeti íve (mainframe → kliens-szerver → grid → SOA → REST →
  virtualizáció → cloud).
- Felhő-árazási modellek (per-second, on-demand, reserved, spot,
  sustained-use, committed-use, preemptible).
- CPU-bound vs. I/O-bound problémák — a Big Data motiváló dichotómia.
- Big Data és a V-modell (Volume / Velocity / Variety + 4-fázisú
  életciklus).
- Stream-keretrendszer programozási axisok — natív vs. mikro-batch,
  deklaratív vs. kompozicionális.
- Apache Beam egységesített programozási modellje (Pipeline / PCollection /
  PTransform).
- Üzenetkézbesítési szemantika (at-most-once / at-least-once /
  exactly-once).
- Cloud service orchestration — lazán csatolt, eseményvezérelt komponensek.
- Konkrét GCP-zászlóshajók kategóriánként (Compute Engine, App Engine,
  Kubernetes Engine, Cloud Functions, BigQuery).
- Wide-column NoSQL adatmodell (Bigtable példáján).
- AI/ML SaaS-példák (Vision AI, Cloud Translation, Text-to-Speech).

## 3. Struktúra — 8 fejezet × 36 alkérdés

A dokumentum első szintje témablokk (H2), második szintje alkérdés (H3).
Minden alkérdés **kérdő mondat formájában** szerepel a fejlécben — közvetlenül
egy lehetséges vizsgakérdést formál, és alatta áll a ½–1 oldalas
esszé-válasz.

### 3.1 Fejezet- és alkérdés-lista

A `[M]` jelzés a felhasználó által biztosított eredeti mintakérdést jelöli.
Ezeknél a kérdés szövegezése és hangsúlya megmarad a vizsga-formában.

#### 1. Felhő-alapok és motiváció (4)

- 1.1 [M] Miért alapvető a virtualizáció a felhőtechnológiában? Hogyan járul
  hozzá a felhő adatközpontok hatékonyságához?
- 1.2 [M] Milyen IT erőforrás-kezelési problémák oldhatók meg
  felhőkörnyezetben, és hogyan nyújt a felhő ezekre megoldást?
- 1.3 Hogyan vezetett az elosztott rendszerek fejlődési íve a felhőhöz?
- 1.4 Mi a CPU-bound és I/O-bound problémák közötti különbség, miért releváns
  ez a Big Data és felhő kontextusában?

#### 2. Felhő-architektúra és modellek (3)

- 2.1 Milyen szereplők alkotják a NIST felhő-referenciamodellt, mi a
  felelősségük?
- 2.2 Mit jelentenek a felhőtelepítési modellek (public / private / hybrid /
  community), és mikor melyiket választjuk?
- 2.3 Hogyan szervezi a felhőszolgáltató földrajzilag az erőforrásait
  (global / regional / zonal), és hogyan kapcsolódik ez a hibatűréshez és a
  késleltetéshez?

#### 3. Szolgáltatási rétegek — x-aaS piramis (6)

- 3.1 [M] Sorold fel a felhő-rendszerek különböző szolgáltatás-szintjeit
  (x-as-a-Service), és röviden ismertesd, melyik szint mire való, mit nyújt
  a felhasználóknak.
- 3.2 IaaS részletesen — mit nyújt, mikor használjuk, előnyök és hátrányok,
  jellemző GCP-szolgáltatás.
- 3.3 PaaS részletesen — ugyanezen szempontok szerint.
- 3.4 SaaS részletesen — ugyanezen szempontok szerint.
- 3.5 Hogyan illeszkedik a CaaS és FaaS a klasszikus 3-szintű piramisba,
  miben különbözik tőle?
- 3.6 Milyen felhő-árazási modellek léteznek, és mikor melyik gazdaságos?

#### 4. Szolgáltatásorientált architektúra a felhőben (3)

- 4.1 [M] Szoftverfejlesztési szempontból hogyan jelenik meg a
  szolgáltatás-orientált architektúra és programozási módszertan a felhő
  alkalmazásokban?
- 4.2 Mit jelent az at-most-once / at-least-once / exactly-once
  üzenetkézbesítési szemantika, és hogyan választunk közöttük?
- 4.3 Mi az a cloud service orchestration, milyen mintákkal építünk
  integrált felhő alkalmazást lazán csatolt komponensekből?

#### 5. Kötegelt feldolgozás — Hadoop és MapReduce (4)

- 5.1 Mi a Big Data, és hogyan írja le a V-modell? Milyen életciklus-fázisokon
  megy át az adat?
- 5.2 [M] Röviden jellemezd a MapReduce programvégrehajtási elvet, annak
  előnyeit és hátrányait. Milyen feladatok végrehajtására fejlesztették ki,
  és azokat hogyan támogatja?
- 5.3 Hogyan épül fel egy Hadoop klaszter (HDFS, YARN, MapReduce), és hogyan
  kezeli a feladatokat?
- 5.4 Mik a Hadoop hátrányai, és miért volt szükség Spark / utód-keretrendszerekre?

#### 6. Adatfolyam-feldolgozás (6)

- 6.1 [M] Melyek az adatfolyam-programozási modell fő jellemzői? Milyen
  helyzetekben vagy alkalmazásoknál használhatók? Mi ennek a számítási
  formának az előnye?
- 6.2 Mik a batch és a stream feldolgozás közötti különbségek, és melyek a
  stream fő kihívásai?
- 6.3 Mit jelent a natív stream és a mikro-batch implementáció, illetve a
  deklaratív és a kompozicionális programozási modell? Hogyan osztályozzuk
  a frameworkek két ortogonális tengelyét?
- 6.4 Mi az ablakozás, mire kell, és milyen ablaktípusok léteznek
  (fixed / sliding / session / global)?
- 6.5 Melyek a domináns stream-feldolgozó keretrendszerek (Spark Streaming,
  Flink, Storm, Kafka, Beam), és mikor melyiket választjuk?
- 6.6 Mi az Apache Beam egységesített modellje (Pipeline / PCollection /
  PTransform), miért fontos?

#### 7. Serverless és felhőfüggvények (5)

- 7.1 [M] Mit takar a Cloud Function fogalom? Mi a szerepük, hogyan jelennek
  meg a felhő alkalmazásokban? Térj ki az előnyeikre és hátrányaikra.
- 7.2 [M] Mi az a háttérben futó (background) felhőfüggvény, mik az előnyei
  és hátrányai? Hogyan lehet meghívni? Adj példákat a felhasználására.
- 7.3 Milyen függvénytípusok léteznek (HTTP vs. background), és mikor melyiket
  választjuk?
- 7.4 Hogyan indíthatók függvények — milyen trigger-rendszer kategóriák
  vannak, hogyan kapcsolódnak össze függvények?
- 7.5 Hogyan működik a Pub/Sub publish-subscribe rendszer, milyen kézbesítési
  minták (1-N, N-1, N-M) léteznek, és mi a szerepe felhő alkalmazásban?

#### 8. Felhő-adattárolás és adatkezelés (5)

- 8.1 Milyen adattárolási kategóriák léteznek a felhőben (object / blokk /
  fájl / relációs / NoSQL / analytikus), és mikor melyiket használjuk?
- 8.2 Milyen tulajdonságok jellemzik a felhő-adatszolgáltatásokat
  (replikáció, biztonság, titkosítás, skálázhatóság, konzisztencia), és
  miért fontosak ezek?
- 8.3 Mi a wide-column NoSQL adatmodell, hogyan működik (Bigtable példáján)?
- 8.4 Mit nyújtanak az adatambar-szolgáltatások (analytikus DB-k)? Mi a
  két-réteges architektúrájuk és az oszloporientált tárolás szerepe?
- 8.5 Mit jelent az AI/ML SaaS mint speciális adatfeldolgozó
  szolgáltatás-kategória, és milyen jellemző példák (Vision AI / Translation /
  Text-to-Speech típusúak) találhatók GCP-n?

## 4. Szövegszabályok

A dokumentum **törzsszövegére** alkalmazandók (a frontmatter és a wiki-szintű
integrációs lépések — `index.md`, `log.md` — kívül esnek):

- **Tilos** verbatim idézet, slide- vagy oldalszám-hivatkozás.
- **Tilos** wiki-link a törzsben (a frontmatter `sources:` mező hordozza
  a teljes attribúciót).
- **Tilos** metanyelv a tananyagra ("a tananyag említi…", "a jegyzet kitér
  rá…", "a wiki tárgyalja…", stb.). A diák ezeket nem tanulja meg, csak
  felesleges méretet adnak.
- **Tilos** programkód, pszeudokód, programpélda — kizárólag elméleti
  tartalom.
- **Megengedett** és kívánatos: konkrét termék- és koncepciónevek
  (Compute Engine, App Engine, BigQuery, Bigtable, MapReduce, Pub/Sub,
  Apache Beam, Vision AI stb.) használata a szövegben — ezek vizsgára
  tanulandó tananyag-elemek, nem hivatkozások.
- **Hangnem:** közvetlen, leíró, didaktikus magyar nyelv, rövidebb tagmondatos
  szerkezetek, vizsgára közvetlenül használható szövegezés.

## 5. Méret-célzás

- **Cél:** 25–30 oldal A4, kb. 15-18 ezer szó.
- **Alkérdés-szint:** tipikusan **½ oldal** (kb. 250-350 szó); ahol a téma
  összetettsége indokolja, **maximum 1 oldal** (kb. 500 szó).
- A `[M]` mintakérdéseknél törekedni kell a felső intervallum-szélre
  (komplex, bővebb választ várnak vizsgán), míg a szintetizált alkérdések
  jellemzően a szűkebb félfeles tartományba illeszkednek.

## 6. Fájl-elhelyezés és metaadat

- **Fájl:** `felho-prog/wiki/analyses/comprehensive-exam-prep-hu.md`.
- **Frontmatter:**

```yaml
---
title: "Cloud Programming — átfogó vizsgafelkészítő elemzés (HU)"
type: analysis
tags: [exam-prep, comprehensive, hu, cloud-programming, juhasz-textbook]
sources:
  - juhasz-cloud-programming-textbook.md
  - cloud-programming-1.md
  - google-cloud-platform-overview.md
  - gcp-practical-1.md
  - data-processing-in-the-cloud-1.md
  - data-processing-in-the-cloud-2.md
  - data-stream-processing.md
  - beam-sdk.md
  - functions.md
  - pub-sub-service.md
  - gcp-bigtable-bigquery-datastudio.md
date_created: 2026-05-04
date_updated: 2026-05-04
---
```

## 7. Wiki-integráció

A dokumentum elkészülte után frissítendő (a fájlon kívül):

- **`felho-prog/wiki/index.md`** — új sor az "Analyses" blokkba, formátum:
  `- [Cloud Programming — átfogó vizsgafelkészítő elemzés (HU)](analyses/comprehensive-exam-prep-hu.md) — Magyar nyelvű, vizsgára felkészítő szintézis a teljes kurzus tananyagáról; 8 fejezet × 36 alkérdés esszé-formában.`
- **`felho-prog/wiki/log.md`** — új bejegyzés:
  `## [2026-05-04] analyze | Comprehensive exam prep (HU)` + 1-2 mondatos
  leírás.
- **QMD újraindexelés** — `qmd update -c felho-prog && qmd embed`, hogy az
  új dokumentum bekerüljön a szemantikus keresésbe.

## 8. Forráskezelés a megíráskor

A megírás során használandó wiki-források:

- **Elsődleges törzs:** Juhász-tankönyv (`sources/juhasz-cloud-programming-textbook.md`)
  és a 6 meglévő magyar fejezet-elemzés
  (`analyses/road-to-cloud-computing-hu.md`,
  `analyses/cloud-technology-foundamentals-hu.md`,
  `analyses/google-cloud-platform-hu.md`,
  `analyses/large-scale-job-execution-hu.md`,
  `analyses/data-stream-processing-hu.md`,
  `analyses/cloud-functions-hu.md`).
- **Kiegészítő:** a 11 source page és a wiki concept/entity oldalai —
  definíciók egységesítésére, framework-listák pontosítására,
  pub-sub minták és üzenetkézbesítési szemantika részleteire,
  storage/database-tulajdonságokra.
- **Megírási mód:** alkérdésenként a kapcsolódó wiki-oldalak elolvasása,
  majd a 4. szakaszban rögzített szövegszabályok szerinti önálló
  esszé-fogalmazás. A `sources:` frontmatter listája a felhasznált forrásokat
  hordozza, attribúciós linkek a törzsbe nem kerülnek.

## 9. Sikerkritériumok

- Mind a 8 felhasználói mintakérdés explicit, kérdő-mondat formájú
  alszekcióval szerepel; a szövegezés a vizsga-formát megőrzi.
- Mind a 8 felhasználó által megjelölt kötelező témakör érdemben lefedve.
- A 4. szakasz szövegszabályai betartva (nincs kód, nincs idézet, nincs
  wiki-link, nincs metanyelv).
- A dokumentum mérete 25–30 oldal A4 tartományban van.
- A fájl `analysis` típusú, a `sources:` frontmatter-mező teljes és
  konzisztens.
- A wiki-integrációs lépések (`index.md`, `log.md`, QMD re-index) lefutottak.
