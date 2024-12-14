=== Metaprogramiranje

Za začetek se pride skozi tako, da se definira induktivni sintakse z evalvacijo,
in doda `notation`, da evaluacija in konstruktorji tipa zgledajo lepše.
Precedenca deluje tako, da višja številka veže strožje, čeprav to pogosto ne dela for some reason.

Naslednji korak je, da se definira sintaktični razred izrazov in sintakso za evalvacijo,
ki se potem z `macro_rules` ali elaboracijo evalvirajo. Od tu naprej pravim temu kar elaboracija.
Tu je treba paziti na več stvari. Prvo, sintaktične kategorije so _sintaktične_,
torej spremenljivke pravega tipa ne pašejo na mesta kjer pričakujemo sintaktični izraz.
Med drugim to pomeni, da ne moremo uporabljati oklepajev v izrazih, torej se splača definirati
`syntax "(" class ")" : class`. Ni mi še očitno kako pametno vložiti izraze v `class`.
Drugo, če želimo dokazati izreke _o sintaksi_ za to še vedno potrebujemo tisti induktivni tip
sintakse in evalvacijo. Elaboracija namreč sintaktično pretvarja izraze preden lean sploh pride
do njih, tako da o elaboraciji ne moramo pokazati ničesar.
(To je rekel Andrej, pa še nisem čisto stestirala, ampak se zdi, da bi moralo tako delati)

Nepredvidena težava pri elaboraciji je nasatala, ko se je uporabljalo spremenljivke.
Elaboracija sintaktično spreminja izraze, in spremenljivk ne vstavlja po imenu, ampak po referenci.
To pomeni, da naivna elaboracija ne dela v drugih kontekstih!
[Tu je primer sprememb, ki so potrebne](https://github.com/andrejbauer/triposes/commit/67613a85)
V bistvu je treba specifično spremenljivko zamenjati s konstruktorjem, če pa želimo
uporabljati funkcije na argumentih, pa to storimo izven sintakse, če je mogoče (`do` in `let`).

Vgrajeni sintaktični razredi so na primer
 - `term`: vsi izrazi
 - `ident`: imena spremenljivk
 - `num`: številske vrednosti
Podpira tudi sezname. `class,*` predstavlja "nič ali več izrazov razreda `class` ločenih z `,`".
Vejico lahko zamenjamo s katerim koli simbolom (ne vem s katerimi točno). Poleg tega lahko
`*` zamenjamo s `+` in dobimo "en ali več izrazov …".
Lahko pa napišemo `class,*,?`, kjer `,?` predstavlja "seznam se lahko konča z vejico".

Paziti je treba, da se definicije sintakse ne preveč prekrivajo, torej definirati seznam kot
`[ class,* ]` je malo nevarno.

Ukaz `macro_rules` dela samo za `term`, torej je najbolje, da svoj razred `class` zaviješ v
neko sintakso, ki ga pretvori v izraz, torej `syntax "[class|" class "]" : term` zgleda še
najbolj konsistentno. Ta sintaksa je mišljena kot implementacijska, torej verjetno je bolje,
da je definirana kot lokalna sintaksa.

Lokalna sintaksa velja samo znotraj sklopa, v katerem je definirana, med tem ko pa
sintaksa sklopa (`scoped syntax`) velja povsod, kjer je ta sklop odprt.
Splošna sintaksa pa "pobegne" iz sklopa v katerem je definirana, in velja v preostanku datoteke.

