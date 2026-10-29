# Review: accordion-toggle PR #36 - Replace the deprecated DOMNodeInserted listener

PR: https://github.com/EssentialBlocks/accordion-toggle/pull/36
Verdict: **Approve** (small rebase onto dev `46fd488` suggested)

## What the PR does
Replaces the deprecated `DOMNodeInserted` listener in `assets/js/eb-animation-load.js` with one capture-phase `change` listener on `document` that checks `event.target.id === 'eb-animation-style'`.
It also goes further than the other plugins' PRs: it updates blocks inside the editor canvas iframe too (`getEditorDocuments()`), measures `isInViewport` against the element's own window, and enqueues `animate.min.css` so the canvas actually has the animation keyframes (`accordion-toggle.php`, +19 lines).
Chrome removed mutation events in v127, so on `dev` the old admin handler never runs and choosing an animation in the inspector does not swap the `eb___*` classes.

## Real merge impact
The PR page can show up to 2 changed files, because the branch was cut from other work that was later merged. A simulated merge into `dev` (`git merge-tree`) changes **2 files, +49/-14** (`accordion-toggle.php`, `assets/js/eb-animation-load.js`). No conflicts.

## Checks
- `node --check` (and `php -l` where PHP changed): no syntax errors.
- Browser test, Local WordPress 7.1.3, Chrome 153, only this plugin active. The PR's JS file was served in place of the plugin's file for the "PR merged" column; for accordion-toggle the PHP was also taken from the merge.

| Check | `dev` | PR merged |
|---|---|---|
| `DOMNodeInserted` listeners registered in the editor | 1 | 0 |
| Admin document: injected `eb___animated eb___fadeIn` after select `change` | stays `eb___...` (handler never ran) | becomes `eb__animated eb__fadeIn` |
| Editor canvas iframe: `animate.min.css` loaded | no | yes |
| Canvas iframe: injected block after select `change` | stays `eb___...`, computed animation-name `none` | becomes `eb__animated eb__fadeIn`, computed animation-name `fadeIn` |
| Console errors / page errors | none | none |

The "select change" check injects a block element and a `#eb-animation-style` select after page load, then fires `change` (a synthetic select, not the real Inspector UI).

## Notes
- This PR is bigger than the same-named PRs in other plugins: it fixes the deprecation and also makes the animation preview work in the iframed canvas. Both parts were verified.
- Rebase onto current `dev` (`46fd488`) before merging.

## Not covered
- Chrome 153 only (no Firefox/Safari), PHP 8.2 only (the plugin supports 7.4), WordPress 7.1.3 on Local.
- The drag chip (third place the icon CSS targets) was not tested.
- The author had not tested this plugin in a browser before opening the PR; these are the first browser measurements.
