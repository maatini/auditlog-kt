# Plan: Neues Repository `auditlog-kt` aus Branch `kotlin-version`

**Ziel:** Erstellung eines eigenständigen, sauberen GitHub-Repositories `maatini/auditlog-kt`, das die Kotlin-Version der Bibliothek als primäres und zukünftiges Haupt-Repository enthält.

**Datum:** 2026  
**Ausgangslage:** Branch `kotlin-version` ist vollständig funktionsfähig, alle Tests grün, Java-Interop verifiziert.

---

## 1. Warum ein separates Repository?

| Aspekt                    | Begründung |
|---------------------------|----------|
| **Saubere Trennung**      | Das Original-Repo `auditlog` bleibt die stabile Java-1.x-Variante. Keine riesigen "Language Switch"-Commits in der History. |
| **Branding & Discoverability** | `auditlog-kt` signalisiert klar: "Das ist die moderne Kotlin-Version". Bessere Auffindbarkeit für Kotlin-Nutzer. |
| **Unabhängige Versionierung** | `auditlog` bleibt bei `1.x`, `auditlog-kt` kann mit `2.0.0` starten (Major wegen Sprachwechsel). |
| **Wartbarkeit**           | CI/CD, Dependabot, Releases können auf Kotlin-spezifische Bedürfnisse optimiert werden (z.B. Kotlin 2.x, kover, etc.). |
| **Einfachere Migration**  | Nutzer sehen sofort zwei klare Angebote: Java (stabil) und Kotlin (empfohlen). |

**Entscheidung:** Kein Fork, sondern bewusst ein neues, frisches Repository mit bereinigter Historie.

---

## 2. Entscheidungen vorab (müssen getroffen werden)

| Thema                        | Empfehlung | Alternative |
|-----------------------------|------------|-------------|
| **Repo-Name**               | `auditlog-kt` | `audit-log-kt`, `auditlog-kotlin` |
| **Maven Coordinates**       | Beibehalten (`io.audit:audit-log-core`) | Neues Artifact `audit-log-kt-core` |
| **Package-Name**            | `io.audit.core` unverändert (stabile API) | Umbenennung (breaking) |
| **Historie**                | **Clean History** (nur Kotlin-Commits) | Volle History mit Filter |
| **example/**                | Java-Demo als Interop-Nachweis behalten | Zusätzlich Kotlin-Demo anlegen |
| **Verbleibende Java-Tests** | Bleiben vorerst (JMH + Integration) | Schrittweise nach Kotlin portieren |
| **Erster Release**          | `2.0.0` (Major wegen Sprachwechsel) | `1.1.0-kt` (weicher Übergang) |

**Empfohlene Variante:** Maven-Koordinaten + Package beibehalten → minimale Umstellung für bestehende Nutzer.

---

## 3. Phasenplan (inkrementell & sicher)

### Phase 0 – Vorbereitung (auf `kotlin-version`)
- [ ] Finalen Stand des Branches festlegen (letzter Commit)
- [ ] `KOTLIN_MIGRATION_PLAN.md` und ggf. dieses Plan-Dokument bereinigen oder als `docs/` ablegen
- [ ] README, CLAUDE.md, pom.xml auf "Kotlin-first" finalisieren
- [ ] Alle sensiblen Daten / lokalen Pfade prüfen (`.devbox`, lokale DBs etc. sind bereits ignoriert)
- [ ] Tag `v2.0.0-rc1` oder ähnlich auf dem Branch setzen (optional)

### Phase 1 – Neues Repository anlegen
- [ ] Auf GitHub neues Repository `maatini/auditlog-kt` als **leer** anlegen (kein README, keine .gitignore)
- [ ] README mit kurzem Hinweis: "Kotlin-Version der Audit-Log-Bibliothek – Nachfolger von auditlog"
- [ ] Settings: Enable GitHub Packages, Discussions, Issues, Wiki nach Bedarf

### Phase 2 – Inhalt importieren (Clean History)
**Empfohlene Methode: "Fresh Clone + Selective Import"** (sauberste Historie)

```bash
# 1. Neues lokales Repo
git clone https://github.com/maatini/auditlog-kt.git auditlog-kt
cd auditlog-kt

# 2. Aus dem bestehenden Repo den reinen Kotlin-Stand exportieren
cd ../auditlog
git checkout kotlin-version

# 3. Alle Dateien des aktuellen Standes in das neue Repo kopieren
git archive kotlin-version | tar -x -C ../auditlog-kt

cd ../auditlog-kt
git add .
git commit -m "feat: initial Kotlin 2.1 version (migrated from auditlog)"

# 4. Push
git push -u origin main
```

**Alternative (git filter-repo):** Wenn man Teile der Historie behalten möchte:
```bash
git clone --branch kotlin-version git@github.com:maatini/auditlog.git auditlog-kt
cd auditlog-kt
git filter-repo --path-rename '' ''   # oder gezieltes Filtern
```

**Empfehlung:** Die "Fresh Clone + archive"-Methode für eine wirklich saubere "Kotlin-only" Historie.

### Phase 3 – Aufräumen & Polishing im neuen Repo
- [ ] `.github/workflows/` prüfen und ggf. anpassen (JMH, Release, CI für Kotlin)
- [ ] `devbox.json` anpassen (falls nötig)
- [ ] `run_demo.sh` und `example/` testen
- [ ] README: Klaren Hinweis auf den Vorgänger-Repo `auditlog` + Migrationspfad
- [ ] `CLAUDE.md` aktualisieren (neues Repo, neuer Fokus)
- [ ] `KOTLIN_MIGRATION_PLAN.md` entfernen oder in `docs/history/` verschieben
- [ ] Ersten Release-Tag `v2.0.0` vorbereiten

### Phase 4 – Maven / Publishing
- [ ] Entscheiden, ob `distributionManagement` auf neue GitHub Packages URL angepasst wird
- [ ] Ersten Release via GitHub Action oder manuell durchführen
- [ ] Sonatype / Maven Central Eintrag prüfen (falls angestrebt)

### Phase 5 – Kommunikation & Migration
- [ ] Im Original-Repo `auditlog`:
  - `README.md` mit deutlichem Hinweis auf `auditlog-kt` als empfohlene Kotlin-Variante ergänzen
  - Issue oder Discussion anlegen: "Kotlin-Version jetzt unter auditlog-kt verfügbar"
- [ ] Release Notes für 2.0.0 schreiben
- [ ] Optional: `auditlog` als "Java Edition" markieren (z.B. via Topics oder README-Badge)

---

## 4. Risiken & Mitigation

| Risiko                        | Wahrscheinlichkeit | Mitigation |
|-------------------------------|--------------------|----------|
| Verwirrung bei Nutzern ("Welches Repo soll ich nehmen?") | Hoch | Sehr klare READMEs in beiden Repos + Migrationsguide |
| Doppelte Pflege von CI/CD     | Mittel | Nach der Etablierung von `auditlog-kt` die Workflows im alten Repo auf "maintenance only" reduzieren |
| Maven-Koordinaten-Konflikt    | Niedrig | Koordinaten bewusst gleich lassen → Nutzer können per Dependency einfach wechseln |
| Verlorene Git-Historie        | Mittel | Bei Bedarf die volle Historie als `auditlog-kt-full-history` Tag/Branch mitliefern |

---

## 5. Erfolgskriterien

- Neues Repo `auditlog-kt` ist öffentlich erreichbar
- `git clone && mvn clean test` funktioniert sofort
- Java-Demo (`example/`) kompiliert und läuft
- README macht unmissverständlich klar, dass dies die Kotlin-Version ist
- Beide Repos verlinken sauber aufeinander

---

## 6. Nächste konkrete Schritte (nach Freigabe des Plans)

1. Entscheidungen aus Abschnitt 2 final treffen (vor allem Maven Coordinates)
2. Phase 0 auf `kotlin-version` abschließen
3. Phase 1–2 innerhalb eines Termins durchführen (idealerweise mit zweitem Pair)
4. Phase 3–5 in den Folgetagen

---

**Dieser Plan folgt den Agenten-Regeln des Projekts:** inkrementell, mit klaren Exit-Kriterien pro Phase, Fokus auf saubere Daten (Historie) und hoher Benutzerfreundlichkeit.

---

*Erstellt auf Branch `kotlin-version` – Stand nach vollständiger Kotlin-Migration*