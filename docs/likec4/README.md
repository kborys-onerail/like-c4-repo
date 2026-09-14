# Architecture as code with LikeC4

This repository models Workline in [LikeC4](https://likec4.dev) and keeps the
sources next to the code they describe. `likec4.config.json` at the repository
root defines one LikeC4 project, so every `.c4` file below it is merged into a
single model.

The landscape PNG on the [root README](../../README.md) is the `index` view of
that model. The same model is also published as an interactive site on GitHub
Pages, where every element links to its own view and the request flows can be
replayed as sequence diagrams.

## Layout

```
likec4.config.json                        project boundary
docs/likec4/                              cross-cutting sources
  specification.c4                        element kinds
  relationships.c4                        relationship kinds
  tags.c4                                 tags
  landscape.c4                            actors and shared infrastructure
  views.c4                                views, grouped into folders
  index.png                               exported landscape for the root README
apps/<app>/docs/likec4/model.c4           one model per application
```

Each application owns the file that describes it, so a change to a service and
the change to its model land in the same pull request.

## Conventions

- An application's element id, directory name, and file location all match,
  for example `web-app` lives in `apps/web-app/docs/likec4/model.c4`.
- Technology is carried by the element kind, not repeated per element. Picking
  `dotnet-service` or `kafka-topic` sets both the icon and the technology
  label, so `specification.c4` is the one place a technology choice changes.
- An application file declares its own element and the relationships it owns
  (its outgoing calls, the topics it publishes, the data it reads or writes).
- Shared elements every application refers to — actors, the event buses, the
  database — stay in `docs/likec4/landscape.c4`.
- Views are cross-cutting and stay in `docs/likec4/views.c4`. Folders are
  created from view titles, not from directories.
- A README overview is a named view exported to PNG. The monorepo uses `index`.
  An application would add a scoped `view <app>-overview of <app>` next to its
  model and embed the resulting PNG in that application's README.

## Commands

From the repository root:

- `npm run dev` — preview the model with live reload
- `npm run validate` — validate all sources
- `npm run build` — generate the static site into `dist/`

## Embedding overviews in README files

LikeC4 does not inject diagrams into markdown. A README embeds an exported
artifact. GitHub renders PNG (and Mermaid); it will not run the interactive
LikeC4 player.

PNG export renders views through a headless browser. Playwright ships with
`likec4`; only the browser binary has to be downloaded before the first run:

```sh
npx playwright install chromium
```

Then export the views that READMEs embed:

```sh
npx likec4 export png --filter index --filter "*-overview"
```

With no `-o` given, each image is written next to the `.c4` file that declares
the view, so `index` becomes `docs/likec4/index.png`, and an application's own
overview stays in that application's folder. Pass `-o <dir>`, optionally with
`--flat`, to collect the images somewhere else instead.

`--filter "*-overview"` is combined with `index` using OR. It is a no-op until
those views exist; it does not fail the export.

## CI

- `c4-pages` builds the interactive site and deploys it to GitHub Pages.
- `c4-regenerate-overview` runs the PNG export on every push to `main` that
  touches the model, and commits the images when they differ. Pushes made with
  `GITHUB_TOKEN` do not retrigger workflows, so this cannot loop.

Diagrams are generated artifacts. A forgotten local export is fine: the
regenerate workflow will catch up on `main`. A pull request still shows the
previous PNG until that commit lands.
