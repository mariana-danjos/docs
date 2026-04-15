# CSW Docs

Documentação do projeto CSW, gerada com Zensical.

## Pré-requisitos

- Python 3.13+
- [uv](https://docs.astral.sh/uv/) instalado

## Como rodar o projeto

1. Instale as dependências:

```bash
uv sync --group dev
```

2. Inicie o servidor local da documentação:

```bash
uv run zensical serve
```

3. Abra no navegador o endereço mostrado no terminal (normalmente `http://127.0.0.1:8000`).

As alterações em arquivos dentro de `docs/` são recarregadas automaticamente.

## Gerar versão estática (build)

```bash
uv run zensical build
```

Os arquivos gerados ficam no diretório `site/`.

## Estrutura principal

- `docs/`: conteúdo em Markdown
- `zensical.toml`: configuração do site
- `site/`: saída gerada do build
