# Architecture as Code

The architecture is modelled with [LikeC4](https://likec4.dev) and kept next to
the code it describes. `likec4.config.json` at the repository root defines the
project, so every `.c4` file below it is merged into a single model.

## Layout

```
likec4.config.json                        project boundary
docs/likec4/                              cross-cutting sources
  specification.c4                        element kinds
  relationships.c4                        relationship kinds
  tags.c4                                 tags
  landscape.c4                            actors and shared infrastructure
  views.c4                                views, grouped into folders
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

## Commands

- `npm run dev` — preview the model with live reload
- `npm run validate` — validate all sources
- `npm run build` — generate the static site
