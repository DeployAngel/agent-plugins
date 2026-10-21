---
name: verify-deploy
description: Wait for DeployAngel's production verdict on a release and act on it. Use after deploying, or after pushing a commit that deploys, in an app that runs the DeployAngel agent (the deployangel gem, Python package, or .NET package); when asked whether a release is safe, cleared, why it failed, or whether a fix worked; and at the start of a session in such an app, to pick up production problems left from earlier sessions.
---

# Verify a deploy with DeployAngel

DeployAngel compares production traffic after a deploy with the last healthy
release and gives the release a verdict: cleared, failed, or not cleared yet.
Never call a release done because CI passed or the deploy finished; wait for
the verdict.

## At the start of a session

Releases keep being verified after a session ends: a nightly job runs, a
rarely used page fails, a release isn't cleared until the next morning.
Before changing code, call `list_deployments` and `list_late_regressions`
(or run the project's CLI with `status`). If a recent release failed, isn't
cleared yet, or has a late regression, tell the user before starting other
work, with the release and what DeployAngel found.

## Wait for the verdict

Use the commit that was deployed (usually `git rev-parse HEAD` after pushing).

- With the `deployangel` MCP server: call `wait_for_verification` with the
  `commit` and `until: "initial"`. Each call waits up to 5 minutes; call again
  while it's still in progress. The deploy can take a few minutes to be
  registered, and the call waits for that too.
- Without it, run the project's own CLI with the same options:
  `bundle exec deployangel` in a Rails app, `uv run deployangel` or
  `poetry run deployangel` in a Python app that uses them, `dotnet deployangel`
  in a .NET app whose `.config/dotnet-tools.json` lists `DeployAngel.Tool`,
  otherwise `deployangel`:

  ```bash
  bundle exec deployangel verify --commit=<sha> --wait --until=initial
  ```

## Act on the result

- Exit 0 / verified: the release is cleared. Report the clearance line, what
  it says it doesn't cover ("Not assessed", "Not run since the deploy"), and
  anything DeployAngel is still watching, then move on.
- Exit 6: no problems so far, but NOT cleared. Report "no problems so far, not
  yet cleared" and the expected clearance time. DeployAngel keeps verifying and
  alerts on failure.
- Exit 7: warnings at the initial check. Report them and review the findings.
  The release is NOT cleared.
- Exit 2 / inconclusive: the release is NOT verified. Do not claim success.
- Exit 1 / failed: read the findings and new exceptions (`get_exception`, or
  `deployangel exception <fingerprint>`, has the stack trace), investigate the
  likely cause, and propose a fix. Do not roll back or change production
  without explicit approval.
- Exit 3: still verifying; call or run it again.
- Exit 5: a usage, authentication, or network error. If it's authentication,
  DEPLOYANGEL_API_TOKEN is missing or wrong: ask for a "CLI & coding agents"
  token from the app's Settings in DeployAngel, set in the environment, never
  written into a file in the repository.
- Confidence `medium` clears a release just as `high` does (high also needs 80%
  coverage and a full baseline), so don't call a medium clearance weak.
  Warnings (`suspicious: true`) aren't failures: report them as "watching
  closely:" and the `watching_closely` items, never as "suspicious".

## After deploying a fix

A failed release stays failed, so a fix is confirmed by the release that
follows it. Its verdict's `fix_check` (in `get_verification` and
`wait_for_verification`, or the CLI's `--format=json`) lists each problem
from the failed releases and late regressions before it, looked up in the
new release's own evidence:

- `fixed`: the release cleared and every earlier problem is healthy again.
  Say so, with the statement.
- `healthy_so_far`: every problem looks healthy, but the release isn't
  cleared yet. Keep waiting for the verdict.
- `still_failing`: a problem is failing again, or happening again in the new
  release's telemetry (`seen_again`). The fix didn't work: investigate it
  like a failure.
- `partly_confirmed` or `not_confirmed`: some problems haven't run since the
  fix (`not_run`) or have too little traffic to judge. Report which ones, and
  use the exercise plan below if they're routes it can run.

`fix_check` is absent when nothing before the release failed.

## A release that isn't cleared yet

Call `get_exercise_plan` (or run `deployangel plan`). If its status is
"exercisable" or "waiting_for_activity", run the project's CLI with
`exercise --url=<production URL>` (gem 0.1.20 or Python package 0.1.8 and
later, or any version of the .NET tool), for example:

```bash
bundle exec deployangel exercise --url=https://example.com
```

It sends the plan's read-only requests from here and records them on the
release, which lists them as exercised from your side. Ask for the
production URL if you don't know it. Offer to exercise what it skips: routes
that change data only with a test account or after asking, and routes that
need a path parameter with a real ID. Then wait for the verdict again. If
the status is "warm_up", nothing run can clear it: a new app's first day
can't clear releases.
