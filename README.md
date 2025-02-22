# Food Tracker

## Description

A smart food companion app that helps users find and save their favorite restaurants and grocery stores, manage
a personalized recipe list, scan product barcodes to check ingredients and allergens, and explore new cooking ideas.

## Development

### Installation

User [uv][uv-docs] to install the project dependencies:

```bash
uv sync
```

[uv-docs]: https://docs.astral.sh/uv/

### Documentation

The project documentation is available in the `docs` folder.

The documentation can be generated and served by [mkdocs][mkdocs-docs].

First, install the `mkdocs` package:

```bash
uv sync --group docs
```

Then, serve the documentation:

```bash
mkdocs serve
```

[mkdocs-docs]: https://www.mkdocs.org/
