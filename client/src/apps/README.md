# `client/src/apps/`

Home for bespoke quiz/game components that used to live inline in `App.jsx`
(see #234 for the plan and #181 for why `App.jsx` is a merge bottleneck).
`GuessNumberApp` is the first one moved, to prove the pattern.

## Convention

- **One component per file**, named exactly like the component: `apps/GuessNumberApp.jsx`.
- **Default export** of the component (`export default GuessNumberApp`), matching how
  the other top-level apps (`./SudokuApp`, `./BattleApp`, ...) are imported.
- **No barrel `index.js`.** A barrel would pull every app into every import and defeats the
  point of splitting; `App.jsx` imports each file directly so diffs stay one line per app.
- **`App.jsx` imports it back** at the end of the top import block:
  `import GuessNumberApp from './apps/GuessNumberApp'`. The `modeMap` entry and
  any other use of the name in `App.jsx` stay exactly as they were.
- Move the component **verbatim** (plus the comment block directly above it). Refactors
  and behaviour changes belong in separate PRs.

## Reaching shared helpers (important)

`useAuth`, `useTimer`, `useAutoAdvance`, `useProgressSubmit`, `fetchQuestionForType`,
`QuizLayout`, `API` and the quiz factories are all defined in `App.jsx`, and `App.jsx`
imports every file in this directory. An app here that imports them from `'../App'` would
create a circular import. Today only a few are even exported (`API`, `useTimer`,
`QuizLayout`); `useAutoAdvance`, `useProgressSubmit`, `fetchQuestionForType` and
`makeQuizApp` are module-private. Two files already do the circular thing
(`EquationCraftingLab.jsx`, `WaterJugLab.jsx` import `QuizLayout` from `./App`); it works
only because the import is used at render time, and it is not a pattern to copy.

So the rule for now:

- An app that needs **no** shared helper from `App.jsx` can move here as-is. This is why
  `GuessNumberApp` was chosen.
- An app that needs one of them **cannot move cleanly yet**. Moving it needs the shared
  hooks/helpers extracted out of `App.jsx` first (for example into `src/hooks/` and
  `src/lib/`). Note that `src/hooks/useTimer.js` already exists and is a different
  implementation from `useTimer` in `App.jsx`, so that needs reconciling before anything
  imports it. That extraction is a separate step and is deliberately not done here.

## Shared dependencies `GuessNumberApp` needed

Only `useState` from `react`. It uses none of `useTimer`, `useAutoAdvance`,
`fetchQuestionForType` or any other `App.jsx` helper, so it imports nothing from `App.jsx`.
