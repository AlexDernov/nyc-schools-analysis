## Environment Setup

Für dieses Projekt wird ein Conda-Environment verwendet, um alle benötigten Pakete in den richtigen Versionen bereitzustellen.

### Conda Environment

Das Environment ist in `environment.yml` definiert.  

**Erstellen des Environments:**

```bash
conda env create -f environment.yml
```
**Aktivieren des Environments:**

```bash
conda activate ds
```

**Hinweis:**
Dieses Environment enthält alle Pakete für Datenextraktion, Filterung, Merging und Analysen, einschließlich SQL-Interaktionen und Datenvisualisierung.
Die Versionen der Pakete sind fixiert, um reproduzierbare Ergebnisse sicherzustellen.