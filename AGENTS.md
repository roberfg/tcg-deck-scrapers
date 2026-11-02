# Contexto del Proyecto

Este repositorio contiene dos scrapers independientes para generar colecciones de cartas desde listas de mazos:

- `magic.app/`: Magic: The Gathering.
- `pokemon.app/`: Pokémon TCG.

La documentación y los comentarios nuevos deben estar en español. Los identificadores, rutas, claves de datos, mensajes de consola y código deben permanecer en inglés cuando formen parte del comportamiento del programa.

## Estructura

```text
scraper-tcg-python/
├── magic.app/scraper.py
├── magic.app/decks/decks.yaml.example
├── magic.app/cards_db.json.example
├── magic.app/exports/
├── pokemon.app/scraper.py
├── pokemon.app/decks/decks.yaml.example
├── pokemon.app/cards_db.json.example
├── pokemon.app/decks_cache.json.example
├── pokemon.app/export/
├── tests/test_magic_scraper.py
├── tests/test_pokemon_scraper.py
├── requirements.txt
├── README.md
└── AGENTS.md
```

Los archivos locales `decks.yaml`, `cards_db.json`, `decks_cache.json` y las salidas generadas están ignorados por Git. No mezclar configuración, cachés ni salidas entre las dos apps.

## Magic

### Flujo

1. Ejecutar desde `magic.app/`.
2. Leer `decks/decks.yaml`, cuyo objeto `decks` contiene formatos con listas de URLs.
3. Descargar cada URL de `mtgdecks.net` o usar `decks_cache.json`.
4. Extraer la lista del selector HTML `#arena_deck`, incluyendo el sideboard.
5. Ignorar tierras básicas.
6. Consultar Scryfall y guardar los resultados en `cards_db.json`.
7. Clasificar cada carta por legalidad Pauper y categoría.
8. Limpiar y regenerar `exports/`.

El scraper usa `curl/8.0` como `User-Agent` para `mtgdecks.net`. No cambiarlo sin validar una URL real, porque el sitio puede responder con contenido distinto o `403` según el cliente.

Las categorías son `creatures`, `lands`, `planeswalkers`, `spells`, `artifacts` y `enchantments`. La prioridad de clasificación es Creature, Land, Planeswalker, Instant/Sorcery, Artifact, Enchantment.

### Salida y cantidades

Se generan ambos archivos para cada categoría, incluso si quedan vacíos:

```text
collection_pauper_<category>.txt
collection_non_pauper_<category>.txt
```

Las cantidades repetidas entre mazos se combinan tomando el máximo. Las cartas desconocidas solo se informan en consola; no crear archivos auxiliares como `unknown.txt` o `review_pauper_nonlegal.txt`.

Comandos:

```bash
cd magic.app
python scraper.py
python scraper.py --refresh
```

`--refresh` ignora la caché de mazos, pero conserva la caché de cartas de Scryfall.

## Pokémon

### Flujo

1. Ejecutar desde `pokemon.app/`.
2. Leer `decks/decks.yaml`, cuyo objeto `decks` contiene formatos con listas de URLs.
3. Descargar decklists desde `play.limitlesstcg.com` o usar `decks_cache.json`.
4. Separar las secciones Pokémon, Trainer y Energy.
5. Ignorar energías básicas.
6. Consultar `limitlesstcg.com` para clasificar los Trainers como Supporter, Item, Tool o Stadium.
7. Guardar la información consultada en `cards_db.json`.
8. Limpiar y regenerar `export/`.

El formato `glc` fuerza la cantidad `1`. Los demás formatos conservan la cantidad máxima por carta e impresión. Las cartas Pokémon se escriben con su impresión, por ejemplo `4 Pikachu (SVI-062)`.

Se generan estos patrones por formato:

```text
collection_<format>_pokemon.txt
collection_<format>_items.txt
collection_<format>_supporters.txt
collection_<format>_tools.txt
collection_<format>_stadiums.txt
collection_<format>_energies.txt
unknown_<format>.txt
```

`unknown_<format>.txt` contiene Trainers cuyo subtipo no se pudo determinar. No eliminar ni renombrar este archivo sin actualizar el README y las pruebas.

Comandos:

```bash
cd pokemon.app
python scraper.py
python scraper.py --refresh
```

## Pruebas y validación

No asumir que `pytest` está instalado. Ejecutar desde la raíz:

```bash
python -m unittest discover -s tests
python -m py_compile magic.app/scraper.py pokemon.app/scraper.py
```

Después de cambiar el formato YAML, los selectores HTML, la clasificación o las reglas de cantidades, ejecutar ambas comprobaciones.

## Reglas de mantenimiento

- Mantener los flujos de Magic y Pokémon separados.
- Usar las plantillas `.example` como referencia para configurar URLs locales.
- No versionar configuraciones personales, cachés, bases de datos ni salidas generadas.
- No cambiar `#arena_deck` ni los `User-Agent` sin probar una URL real.
- Mantener el formato `premodern` en Magic; no usar `premoder`.
- No crear nuevas salidas para Magic sin actualizar este archivo, `README.md` y las pruebas correspondientes.
- Si una caché produce resultados inesperados, ejecutar el scraper con `--refresh` antes de diagnosticar el parser.
