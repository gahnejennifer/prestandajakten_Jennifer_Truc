| Ändring              | LCP dashboard (median) | CLS dashboard | JS inloggningssidan (gzip) | JS dashboarden (gzip) |
|----------------------|------------------------|---------------|----------------------------|-----------------------|
| Före                 |  8,22 s                      | 0,37 s              |   143,42 kB                        |  143,42 kB                    |
| 1. …                 |                        |               |                            |                       |

## Fynd

**LCP-elementet (2a)**
- LCP-element: `<img>` med hero.png på dashboarden.
- Uppdelning av LCP (tredje mätningen, 8,21 s):
  - Time to first byte: 11 ms
  - Resource load delay: 1 523 ms → bilden upptäcks sent
  - Resource load duration: 6 627 ms → filen är enorm
  - Element render delay: 51 ms → Över 80 % av LCP är bildens storlek, resten är att den hittas sent.

**Nätverket (3a)**
- Största och långsammaste filen: hero-BDLRZ5ff.png, 6 486 kB, 6,80 s. Nästan hela sidans 6,6 MB.
- JS: index-DgSmv3V4.js, 144 kB, 316 ms. All kod för alla sidor i en fil.
- Långsammaste API-anropet: consumption, 615 ms för 0,4 kB (user 170 ms och refresh 172 ms). Waiting for server response ≈ 612,91 ms → servern väntar, svaret är inte stort.

**Bundlen (3b)**
- lodash: 564,7 kB (41,5 % av bundlen), gzip 95,86 kB. Största enskilda biblioteket. Importeras helt (`import _ from 'lodash'`) men används bara till två funktioner: _.upperCase(appEnv) i App.vue och _.debounce i DashboardView.vue (som bara gör console.log vid resize). App.vue laddas alltid → lodash belastar även /login.
- chart.js: 395,8 kB (29,1 % av bundlen), gzip 77,56 kB. ConsumptionChart.vue importerar 'chart.js/auto' = alla diagramtyper, fast vi bara ritar ett stapeldiagram. Används bara på dashboarden men ligger i index-*.js.
- Tillsammans är lodash och chart.js ca 70 % av all JavaScript.
- OBS: DashboardView.test.js mockar 'chart.js/auto' – måste ändras vid fix D.

**Vad hoppar och vad väntar (3c)**
- Allt från "hej Anna" dyker upp först, sen kommer "din förbrukning" och bilden (och den laddas långsamt in) som trycker ner innehållet under sig. LCP är först grön och har ett värde, men det slutliga värdet kommer efter bilden har laddats in och är då rött. CLS kommer innan bilden laddats in (ganska direkt). 
- hero.png börjar hämtas först efter att consumption svarat (bilden väntar på API:t, förklarar resource load delay ~1,5 s).

**Flaskhalsen som inte är vår**
- /api/v2/consumption väntar ca 600 ms innan den svarar (setTimeout i mock-api/server.js). Kraftlys API, inte frontendkoden. Rapporteras till den som äger API:t. Ligger kvar vid poängmätningen.