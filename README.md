# tcg-scraper-python

Scrapers independientes para mantener colecciones de cartas de Magic: The Gathering y Pokémon TCG a partir de listas de mazos.

## Estructura

```text
scraper-tcg-python/
├── magic.app/      # Scraper de Magic
├── pokemon.app/    # Scraper de Pokémon
├── tests/          # Pruebas unitarias
├── requirements.txt
├── README.md
└── AGENTS.md       # Contexto para agentes LLM
```

Cada app tiene su propia configuración, caché, base de datos y directorio de salida. Los archivos locales están ignorados por Git. Las plantillas versionadas terminan en `.example`.

## Instalación

Requiere Python 3 y las dependencias de `requirements.txt`.

```bash
python -m venv .venv

# Linux/macOS
source .venv/bin/activate

# Windows PowerShell
.venv\Scripts\Activate.ps1

pip install -r requirements.txt
```

## Magic

La app `magic.app/` descarga mazos desde páginas individuales de `mtgdecks.net`. La lista se extrae del elemento `#arena_deck`, incluyendo el sideboard, y las tierras básicas se ignoran.

Configura las URLs en `magic.app/decks/decks.yaml`, partiendo de `magic.app/decks/decks.yaml.example`:

```yaml
decks:
  pauper: []
  standard:
    - https://mtgdecks.net/Standard/example-decklist-by-player-0000000
  modern: []
  premodern: []
  pioneer: []
```

Cada formato contiene una lista de URLs. El orden del YAML se conserva. Scryfall determina la legalidad Pauper y el tipo de carta. La prioridad de clasificación es `Creature`, `Land`, `Planeswalker`, `Instant/Sorcery`, `Artifact`, `Enchantment`.

Ejecuta desde `magic.app/`:

```bash
python scraper.py
python scraper.py --refresh
```

`--refresh` ignora `decks_cache.json` y vuelve a descargar los mazos. La base de datos `cards_db.json` se conserva para evitar consultas repetidas a Scryfall.

La salida se regenera en `magic.app/exports/` y contiene siempre estos archivos para cada categoría (`creatures`, `lands`, `planeswalkers`, `spells`, `artifacts`, `enchantments`):

```text
collection_pauper_<category>.txt
collection_non_pauper_<category>.txt
```

Cuando una carta aparece en varios mazos, se conserva la cantidad máxima. Las cartas desconocidas se informan en consola, pero no se escriben en un archivo adicional.

## Pokémon

La app `pokemon.app/` descarga decklists de `play.limitlesstcg.com`, separa las secciones Pokémon, Trainer y Energy, e identifica el subtipo de los Trainers consultando `limitlesstcg.com`. Las energías básicas se ignoran; otras energías se conservan.

Configura las URLs en `pokemon.app/decks/decks.yaml`, partiendo de `pokemon.app/decks/decks.yaml.example`:

```yaml
decks:
  standard:
    - https://play.limitlesstcg.com/tournament/example/player/example/decklist
  glc:
    - https://play.limitlesstcg.com/tournament/example/player/example/decklist
```

Los formatos son configurables y sus valores son listas de URLs. El formato `glc` fuerza la cantidad `1`; en los demás formatos se conserva la cantidad máxima entre mazos. Las cartas Pokémon incluyen su impresión, por ejemplo `4 Pikachu (SVI-062)`.

Ejecuta desde `pokemon.app/`:

```bash
python scraper.py
python scraper.py --refresh
```

`--refresh` vuelve a descargar los mazos y actualiza `decks_cache.json`. La base de datos `cards_db.json` almacena los subtipos consultados usando la clave `nombre|set|número`.

La salida se regenera en `pokemon.app/export/`:

```text
collection_<format>_pokemon.txt
collection_<format>_items.txt
collection_<format>_supporters.txt
collection_<format>_tools.txt
collection_<format>_stadiums.txt
collection_<format>_energies.txt
unknown_<format>.txt
```

`unknown_<format>.txt` contiene Trainers cuyo subtipo no pudo determinarse.

## Pruebas

Ejecuta la suite desde la raíz del repositorio con la librería estándar:

```bash
python -m unittest discover -s tests
```

También puedes validar la sintaxis de ambos scrapers:

```bash
python -m py_compile magic.app/scraper.py pokemon.app/scraper.py
```

## Fuentes y dependencias

- [Python](https://www.python.org/)
- [Requests](https://requests.readthedocs.io/)
- [Beautiful Soup](https://www.crummy.com/software/BeautifulSoup/)
- [PyYAML](https://pyyaml.org/)
- [Scryfall API](https://scryfall.com/docs/api)
- [MTGDecks](https://mtgdecks.net/)
- [Limitless TCG](https://limitlesstcg.com/)
