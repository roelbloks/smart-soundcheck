# Smart Soundcheck — web-prototype

Een losstaande web-app (één `index.html`, geen dependencies) die op je iPad in Safari
draait en met de microfoon de **akoestiek van de zaal** meet. Alles client-side; ná het
laden werkt de meting volledig offline.

Dit is het **web-prototype**. De native iPad-app uit `CLAUDE_PROMPT.md` (Soundcraft Ui12
read/write, USB-meetmic, WKWebView-mixer) is een apart, later traject dat volledige Xcode
vereist — die staat niet op deze Mac.

## Wat zit erin
- **Analyzer** — live 31-bands spectrum (zelfde ISO-frequenties als de Ui12 grafische EQ),
  RMS/piek in dBFS, clip-indicatie, samplerate, instelbare FFT-grootte en demping, peak-hold.
- **Zaalmeting** — leg per publieksplek een positie vast (3 s middeling), de app middelt
  alle posities en stelt **alléén brede cuts** voor (begrensd op −6 dB), met vertrouwen op
  basis van de spreiding tussen posities. Gemeten vs. voorgestelde curve + bypass A/B.
- **Testsignaal** — lokaal pink-noise en sine-sweep (20 Hz→20 kHz), *ingedrukt-houden* om te
  spelen, direct stoppen bij loslaten, laag standaardniveau + grote STOP-knop.
- **Vocal & advies** — startpunten voor je Shure Beta-zangkanaal (HPF, EQ, de-ess, comp),
  feedback-zones, Bose L1 Pro8-notitie en de veilige werkvolgorde.
- **Ui12 mixer** — snelkoppeling die de originele Ui12-webinterface in een nieuw tabblad
  opent. Deze app leest/schrijft **niets** naar de mixer.
- Sessies exporteren/importeren als JSON (Bestanden-app), posities blijven lokaal bewaard.

## Op je iPad zetten (mic werkt alleen via HTTPS)
iOS Safari geeft alleen microfoontoegang op een **beveiligde (https) pagina**. Gewone
`http://`-LAN-adressen worden geweigerd. Jij hebt Laravel Herd, dus de makkelijkste weg:

1. Deze map staat al in je Herd parked-pad, dus Herd serveert hem automatisch als
   `https://smart-soundcheck.test` (op de Mac).
2. Maak daar een publieke HTTPS-URL van voor de iPad:
   ```bash
   cd ~/Herd/"Smart Soundcheck"
   herd share
   ```
   Herd/Expose geeft een `https://…`-URL terug.
3. Open die URL op de iPad in Safari → **sta de microfoon toe** → Start meting.
4. Tik op Deel → *Zet op beginscherm* voor een schermvullende app-ervaring.

Alternatief zonder Herd: sleep deze map op <https://app.netlify.com/drop> voor een directe
HTTPS-URL, of gebruik `cloudflared tunnel --url http://localhost:8795`.

> Let op: een `herd share`/tunnel-URL is publiek zolang hij draait. De app bevat geen
> gevoelige data, maar stop de share als je klaar bent (Ctrl-C).

## Wat is echt en wat is nog ruw (eerlijk)
- **Echt:** mic-opname, FFT → 31-bands, niveaus/clip, meerdere posities middelen,
  resonantiedetectie → begrensde cut-voorstellen, testsignalen, JSON-opslag.
- **Ongekalibreerd:** dB-waarden zijn relatief. Echte SPL/kalibratie vereist een
  referentiemeetmicrofoon + bekende gevoeligheid (native-app-traject).
- **Geen transfer-functie:** dit meet het *spectrum* dat de mic hoort, niet de echte
  overdrachtsfunctie PA→zaal. Meet met pink-noise via de PA voor zinnige resultaten en
  EQ de zaal nooit blind vlak.
- **Mixer:** alleen een launcher. Geen read/write naar de Ui12 (bewust, veilig).

## Roadmap naar een veilige, gekalibreerde soundcheck
1. **Nu (dit prototype):** relatieve meting + conservatief advies, handmatig toepassen.
2. **Native iPad-app:** USB class-compliant meetmic + kalibratieprofiel (echte SPL),
   Ui12-webUI in één scherm (WKWebView), read-only mixerstatus.
3. **Gevalideerde mixer-write:** getest Ui12-protocol, parameter-diff + bevestiging +
   rollback; nooit automatische grote boosts of ongecontroleerde faders.
