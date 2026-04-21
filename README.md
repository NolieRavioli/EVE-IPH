# EVE Isk per Hour (EVE-IPH)

EVE Isk per Hour is a Windows desktop application for [EVE Online](https://www.eveonline.com/) that helps industrialists plan and price manufacturing, invention, reactions, mining, and other production activities. It calculates ISK-per-hour profitability across blueprints, components, and market data using the EVE Static Data Export (SDE) and live data from CCP's ESI API.

This repository is written in VB.NET (Visual Studio, .NET Framework, WinForms) against SQLite via `System.Data.SQLite`.

---

## Requirements

- Windows 10 or later
- [.NET Framework 4.7.2+](https://dotnet.microsoft.com/download/dotnet-framework) (matches [app.config](app.config))
- Visual Studio 2019 or later with the **Visual Basic** workload (only required if building from source)
- A populated `EVEIPH DB.sqlite` database file placed next to the executable (see below)

> **You must supply the SQLite database yourself.** The database is *not* committed to this repository because it is large and is regenerated each time CCP publishes a new SDE.

---

## Building `EVEIPH DB.sqlite`

If you launch the app without a properly-built database you will see SQLite errors such as:

```
System.Data.SQLite.SQLiteException: 'SQL logic error
no such table: ESI_STATUS_ITEMS'
```

This happens because EVE-IPH expects a SQLite file named **`EVEIPH DB.sqlite`** to live in the application's working directory, and that file must contain both the SDE tables *and* the EVE-IPH-specific tables (`ESI_STATUS_ITEMS`, `ESI_ENDPOINT_ROUTE_TO_SCOPE`, character/asset cache tables, etc.).

To build that database, use the companion project:

### [EVE-SDE-Database-Builder](https://github.com/EVEIPH/EVE-SDE-Database-Builder)

Steps:

1. Clone and build (or download a release of) the SDE Database Builder.
2. Run it and point it at the latest CCP SDE.
3. Choose **SQLite** as the output target and have it produce a file named `EVEIPH DB.sqlite`.
4. Make sure the builder is configured to emit the EVE-IPH auxiliary tables. At minimum the following tables must exist or EVE-IPH will throw `no such table` errors at startup:
   - `ESI_STATUS_ITEMS`
   - `ESI_ENDPOINT_ROUTE_TO_SCOPE`
   - all standard SDE tables (`invTypes`, `industryActivity*`, `mapSolarSystems`, etc.)
5. Copy the resulting `EVEIPH DB.sqlite` next to `EVE Isk per Hour.exe` (in `bin\Debug` or `bin\Release` when running from Visual Studio, or in the install directory for a packaged build).

The expected filename is hard-coded in [Globals.vb](Globals.vb#L57):

```vbnet
Public Const SQLiteDBFileName As String = "EVEIPH DB.sqlite"
```

---

## Building from source

1. Clone the repo.
2. Open [EVE Isk per Hour.sln](EVE%20Isk%20per%20Hour.sln) in Visual Studio.
3. Restore NuGet packages (Newtonsoft.Json, System.Data.SQLite, etc. — already vendored under `packages\`).
4. Build the `EVE Isk per Hour` project.
5. Drop `EVEIPH DB.sqlite` into the build output folder (`bin\Debug\` or `bin\Release\`).
6. Run.

The solution also contains two helper projects:

- **EVE-IPH-Update-Program** — handles patching the database between SDE releases.
- **EVEIPH SQLite DLL Updater** — swaps out the native SQLite interop DLL when needed.

Neither helper is required for normal use of the main app.

---

## ESI / CCP API registration

EVE-IPH uses CCP's ESI for character data (skills, blueprints, assets, industry jobs, standings, market orders, etc.). To use the character-linked features you need to register an application at <https://developers.eveonline.com/> and plug your **Client ID** and **Secret Key** into the in-app ESI settings dialog. The app uses the standard OAuth2 flow with a localhost callback.

Without ESI credentials the manufacturing/pricing features still work — you just can't import your own character.

---

## Project layout (high level)

- [frmMain.vb](frmMain.vb) — main UI and orchestration.
- [Blueprint.vb](Blueprint.vb), [BuildBuyItems.vb](BuildBuyItems.vb), [ConvertToOre.vb](ConvertToOre.vb) — production math.
- [ESI.vb](ESI.vb), [EVEAssets.vb](EVEAssets.vb), [EVEBlueprints.vb](EVEBlueprints.vb), [EVEIndustryJobs.vb](EVEIndustryJobs.vb), [EVELoyaltyPoints.vb](EVELoyaltyPoints.vb), [EVENPCStandings.vb](EVENPCStandings.vb), [EVESkillList.vb](EVESkillList.vb) — ESI clients and per-character data caches.
- [DBConnection.vb](DBConnection.vb) — thin wrapper around `System.Data.SQLite`.
- [Globals.vb](Globals.vb) — global constants (including the DB filename above).
- `frm*.vb` — WinForms dialogs (assets viewer, blueprint manager, character skills, error log, ESI status, settings, etc.).
- `EVE-IPH-Update-Program\` — separate updater app.
- `EVEIPH SQLite DLL Updater\` — separate native-DLL updater app.

---

## License

EVE Online and the EVE logo are trademarks of CCP hf. — this project is not affiliated with or endorsed by CCP.

