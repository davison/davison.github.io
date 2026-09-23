# M1 — Publish the blog at davisononline.org

Tracking issue: [#13](https://github.com/davison/davison.github.io/issues/13). Its
single implementation task, [#14](https://github.com/davison/davison.github.io/issues/14),
is merged on `master` through [#15](https://github.com/davison/davison.github.io/pull/15)
at [`28280fd`](https://github.com/davison/davison.github.io/commit/28280fd4e389bfdd1be9b89e5eac5f863cab1a69).

## Goal and outcome

The goal, as stated on [#13](https://github.com/davison/davison.github.io/issues/13),
was to serve the blog over HTTPS at davisononline.org and to redirect the existing
davison.github.io URLs with their paths preserved.

What shipped is three lines of configuration and a great deal of infrastructure that
is not in the diff. `site_url` in `zensical.toml` and the description in
`pyproject.toml` name the new domain, and because `main.py` reads `site_url` when it
regenerates the feed, every canonical link, Atom entry and sitemap URL follows from
that one value. GitHub Pages holds the custom domain with a Let's Encrypt certificate
and HTTPS enforced, and old URLs return 301 to the same path on the new host.

The proportion is worth recording: the intended change was three lines, the generated
`docs/atom.xml` accounts for 70 more, and a new CI workflow — added only to satisfy a
merge gate, below — for 25. The milestone's real work was DNS, certificate issuance
and deployment, none of which a diff can show.

## Decisions

Each is recorded on [#14](https://github.com/davison/davison.github.io/issues/14#issuecomment-5785257970)
unless noted.

- **davisononline.org is canonical; www redirects to it.** Page paths stay unchanged
  and GitHub Pages supplies the redirects natively. **Rejected:** committing a `CNAME`
  file, which an Actions deployment does not need — the custom domain lives in the
  repository's Pages settings instead.
- **GitHub Issues enabled on the repository.** Creating the milestone returned HTTP
  410 because Issues were disabled; the CodeCrew record requires them. This is a
  decision about the repository rather than the site.
- **The drifted role contracts stay as they are.** The CLI reports all five
  `.codecrew/roles/` contracts differing from the embedded v2.0.1 text. Reviewed and
  deliberately not reconciled: contracts are this project's own fork, and adopting
  upstream is its own change, not a rider on a domain migration.
- **The generator's Atom ID behaviour is preserved.** **Trade-off:** entry IDs derive
  from the canonical URL, so changing the domain changes every existing entry's ID and
  subscribers may see old posts resurface once. **Rejected:** pinning IDs to the old
  domain, which would have made the feed permanently inconsistent with the site for
  the sake of a single re-delivery.

## Deviation

**A pull-request build workflow was added alongside the domain change**
([#14](https://github.com/davison/davison.github.io/issues/14#issuecomment-5785276945)).
`gh codecrew task finish 14 --dry-run` refused with `NO_CHECKS`: the only existing
workflow builds after merge or on a schedule, so no check ran on the PR itself.
`.github/workflows/build.yml` runs the same housekeeping and clean build read-only,
without deploying. It is 25 of the 63 lines this milestone changed, and it exists
because of the gate rather than because of the goal.

## What the plan did not anticipate

None of the following was recorded before deployment; all of it was found while
verifying M1-R2, and it is the part most worth having in three months.

- **The deploy workflow had disabled itself.** GitHub disables scheduled workflows in
  a public repository after 60 days of inactivity. The last successful scheduled run
  was [2026-07-12](https://github.com/davison/davison.github.io/actions/runs/29177941560);
  the repository was then quiet until September, and neither the September pushes nor
  the M1 merge triggered a deploy. The live site was serving a July build with the old
  canonical URL throughout — so the configuration change was correct and invisible,
  which no gate in this milestone would have caught.
- **A dispatched run can be orphaned.** The first manual dispatch,
  [35793033285](https://github.com/davison/davison.github.io/actions/runs/35793033285),
  sat queued with zero jobs and zero billable time. `gh run cancel` reported it
  complete while the API reported it queued, and force-cancel reported it never
  queued: the run row existed but nothing was ever enqueued. It cannot be cancelled and
  remains in the run list. A second dispatch,
  [35794660366](https://github.com/davison/davison.github.io/actions/runs/35794660366),
  ran normally and deployed, which is what made the domain change live.
- **DNS and the certificate do not arrive together.** Between the A records resolving
  and Let's Encrypt issuing, `https://davisononline.org` was served the `*.github.io`
  wildcard certificate — a name mismatch — and GitHub would not accept **Enforce
  HTTPS** until the certificate existed. During that window the redirects from the old
  domain were correct but pointed at `http://`. The order is DNS, then certificate,
  then enforcement, and the middle step is GitHub's to take on its own schedule.
  A browser that met the mismatched certificate in that window stores the bypass per
  origin and keeps reporting the site insecure afterwards; it clears on browser
  restart, not on a cache clear.

One finding is out of this milestone's scope and is noted only because it was found
here: `reconcile_drafts()` in `main.py` can never promote a draft in CI, because
`_drafts/` is gitignored and the directory the runner creates is always empty. The
demotion half works, so future-dated posts committed to `docs/blog/posts/` are hidden
until their date — that is the mechanism that actually schedules a post.

## Requirement outcomes

Verdicts are recorded on [#13](https://github.com/davison/davison.github.io/issues/13).

| Requirement | Outcome |
|---|---|
| M1-R1 — site configuration and generated feed use https://davisononline.org/, and the clean build succeeds | satisfied |
| M1-R2 — Pages serves davisononline.org over HTTPS, and old URLs return 301 preserving paths | satisfied |
