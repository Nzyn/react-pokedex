# React Pokedex

A Pokedex single-page app built with **React**, **React Router** and **Tailwind CSS**, using data
from [PokeAPI](https://pokeapi.co/). This is a React rewrite of the Express + EJS Pokedex in the
folder above.

> **New here?** There is a full step-by-step tutorial in [`guide/`](./guide/00-introduction.md) that
> builds this app from a fresh Vite starter, one feature at a time, and ends with opening a Pull
> Request.

## Getting started

**Requires Node 22.12 or newer** (Vitest 5 does not support Node 20).

```bash
npm install
npm run dev
```

Then open http://localhost:5173.

| Command            | What it does                              |
| ------------------ | ----------------------------------------- |
| `npm run dev`      | Start the dev server with hot reload      |
| `npm run build`    | Build the site into `dist/`               |
| `npm run preview`  | Preview the built site                    |
| `npm test`         | Run the tests and watch for changes       |
| `npm run test:run` | Run the tests once                        |
| `npm run test:ci`  | Run the tests once with a coverage report |
| `npm run lint`     | Check the code with ESLint                |
| `npm run format`   | Format the code with Prettier             |
| `npm run ci`       | Format check, lint, test and build        |

## The files

Each page loads its own data, so you can read any single page top to bottom and understand
everything it does.

```
src/
├── main.jsx                    starts the app
├── App.jsx                     the list of pages and their addresses
├── index.css                   Tailwind setup and theme colours
├── utils.js                    shared helpers (see below)
│
├── layout/
│   └── Layout.jsx              header, search box and footer
│
├── components/
│   ├── PokemonGrid.jsx         the card grid, cards and type filters
│   ├── PokemonDetail.jsx       the big card on the details page
│   └── ui.jsx                  loading, empty, error and pagination
│
└── pages/
    ├── HomePage.jsx            the full Pokedex, 20 at a time
    ├── SearchPage.jsx          search results
    ├── TypePage.jsx            Pokemon of one type
    └── PokemonDetailsPage.jsx  everything about one Pokemon

tests/                          one test file per source file
├── setup.js
├── utils.test.js
├── App.test.jsx
├── layout/
├── components/
└── pages/

guide/                          the step-by-step tutorial
.github/
├── workflows/ci.yml            the submission pipeline
└── pull_request_template.md    the PR description template
```

## Pages

| Address              | Page               | Example              |
| -------------------- | ------------------ | -------------------- |
| `/`                  | HomePage           | `/?page=3`           |
| `/search`            | SearchPage         | `/search?q=char`     |
| `/type/:type`        | TypePage           | `/type/water?page=2` |
| `/pokemon/:nameOrId` | PokemonDetailsPage | `/pokemon/pikachu`   |

The page number and search text live in the address bar, so the back button and shared links both
work.

## How a page loads its data

Every page does the same thing, with only `useState` and `useEffect`:

```jsx
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

useEffect(() => {
  let ignore = false;

  async function load() {
    try {
      const result = await get('/pokemon/pikachu');
      if (!ignore) {
        setData(result);
        setLoading(false);
      }
    } catch {
      if (!ignore) {
        setError('Something went wrong.');
        setLoading(false);
      }
    }
  }

  load();

  // Runs when the input changes, so a slow old answer is thrown away.
  return () => {
    ignore = true;
  };
}, [page]);
```

Read one page and you can read them all. There are no custom hooks and no data-fetching library.

## What is in utils.js

Only the things more than one page needs, so they are not copy-pasted four times:

| Helper                       | What it does                                        |
| ---------------------------- | --------------------------------------------------- |
| `get(path)`                  | One request to PokeAPI. Returns `null` on a 404     |
| `loadPokemon(nameOrId)`      | One Pokemon, tidied up. `null` if it does not exist |
| `loadMany(entries)`          | Loads details for a list of Pokemon at once         |
| `getTypeColor(type)`         | The colour for a type, e.g. fire is orange          |
| `formatName`, `formatNumber` | `"mr-mime"` → `"Mr Mime"`, `25` → `"025"`           |

## Tests

The tests use [Vitest](https://vitest.dev/) and
[React Testing Library](https://testing-library.com/react). They live in `tests/`, laid out to match
`src/`, so the tests for `src/pages/HomePage.jsx` are in `tests/pages/HomePage.test.jsx`.

```bash
npm test           # watches for changes while you work
npm run test:run   # runs once
npm run test:ci    # runs once and prints a coverage report
```

The tests never touch the real PokeAPI. Instead they replace `axios.get` with a fake that returns
whatever the test needs:

```js
vi.spyOn(axios, 'get').mockResolvedValue({ data: pikachuResponse });
```

Testing Library checks what a person would see on screen — a heading, a link, a button — rather than
how the component is built inside. So the tests keep passing when you change the styling.

What is covered:

| File            | What its tests check                                                   |
| --------------- | ---------------------------------------------------------------------- |
| `utils`         | Formatting, 404s becoming `null`, and building the tidy Pokemon object |
| `ui`            | Pagination links, and the loading / empty / error messages             |
| `PokemonGrid`   | Cards show the name, number, types and picture, and link correctly     |
| `PokemonDetail` | Stats, abilities, hidden badge, height and weight                      |
| `Layout`        | Searching takes you to the search page                                 |
| Pages           | Loading first, then results — plus "not found" and error states        |
| `App`           | Each address shows the right page                                      |

## CI pipeline

`.github/workflows/ci.yml` runs on every pull request. It has four jobs:

| Job                                 | What it does                                                                                                                                                               |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Extract Student Info**            | Reads `First Name`, `Last Name`, `Program` and `UMindanao Email` from the PR description, checks all four are present, validates the email domain, and checks the PR title |
| **Verify PR Author Wrote The Code** | Compares every commit's GitHub account against whoever opened the PR                                                                                                       |
| **Format, Lint, Test & Build**      | `format:check` → `lint` → `test:ci` → `build`, then confirms `dist/` really contains the HTML, JS and CSS                                                                  |
| **Submit Student Info**             | Runs only after all three pass, and POSTs the details to the submission API                                                                                                |

Run the same gates locally before opening a PR:

```bash
npm run ci
```

### Opening a pull request

`.github/pull_request_template.md` pre-fills the description. Two things the pipeline is strict
about:

- **All four student fields are required**, with the labels spelled exactly as in the template, and
  the email must end in `@umindanao.edu.ph`.
- **The PR title must be `<Last Name>/pokedex-pull-request`**, using the Last Name field exactly as
  you typed it. `Last Name: Dela Cruz` means the title is `Dela Cruz/pokedex-pull-request` — note
  this is _not_ the same as your lowercase branch name.

### Secrets

The submit job needs two repository secrets: `SUBMISSION_API_URL` and `SUBMISSION_API_TOKEN`.
Without them the first three jobs still run; only the final submission step will fail.

## Notes

- The Express version fetched Pokemon on the server. Here the browser does it, so loading a page of
  20 Pokemon means 20 small requests. `loadMany` uses `Promise.all` to run them at the same time.
- Tailwind v4 is configured in `src/index.css` with `@theme`, so there is no `tailwind.config.js`.
- CI pins **Node 22**. Vitest 5 requires `^22.12 || ^24 || >=26`, so the Node 20 used by the Express
  version of this project will not work here.
