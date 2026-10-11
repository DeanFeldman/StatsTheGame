
/
Claude finished the response
Handover — Sport Analytics Tool
Paste this at the start of a new conversation.
Who I am and what this is
I'm Ben Swartz, one of six on team Git Push Pray building the Sport Analytics Tool for COMS3011A at Wits. Cricket, event-driven, deriving every statistic from ball-by-ball data rather than storing totals. Sprint 1 was marked on 25 August; Sprint 2 closes 15 September, Sprint 3 on 29 September (which clashes with our CGV project), final submission 11 October.
We're targeting the advanced tier.
Project state
The platform works end to end: Google sign-in, submitter approval with competition scoping, event submission with validation, derivation, public read API, React frontend, external API integration (Open-Meteo), deployed to Azure, documentation site on Cloudflare Pages.
Data. The full Cricsheet corpus is imported: 14,011 fixtures, 3,207,109 deliveries, 741 competitions, 13,481 participants. 798 MB total, of which delivery is 705 MB (385 heap, 319 indexes) — about 249 bytes per delivery.
Database. Hosted PostgreSQL 17.6 in eu-west-2, in a Supabase organisation belonging to my employer, used with permission (ADR-005). Authentication remains on the previous Supabase project, so SUPABASE_URL and the VITE_ equivalents point there while DATABASE_URL points at the new one.
Latency. ~173 ms round trip from Johannesburg. This shapes everything: a page of fifty fixtures costs ~190 ms (one round trip), while the statistics endpoints cost ~2,400 ms (roughly thirteen sequential queries).
What I've owned

#27 the delivery event schema and its migration, validated against a published scorecard by 22 SQL assertions (fixture 729307)
#46 / #63 the development seed — four fixtures chosen to exercise the awkward cases — and three invalid submission examples
#104 super-over exclusion from batting and bowling aggregates
#121 / ADR-005 the database host migration
#49 / #50 audited the submission contract and validation pipeline rather than rebuilding them; found three defects, two of which returned an unexplained 500 where the brief requires a specific rejection
#175 the corpus importer and batched ingestion (0.05 → 0.28 matches/sec, eight days → fifteen hours), plus verification, documentation and tests
#105 endpoint response measurements
Known issues, raised or not
Raised as issues: none of the below yet, unless I've done it since writing this.

Statistics endpoints issue ~13 sequential queries per request. /fixtures/{id}/statistics and /participants/{id}/fixtures each ~2,400 ms.
The backend serves every request 2.03 seconds late after idling, including /health, which does no database work. Restarting fixed it. Cause unconfirmed. A deployed backend would show this silently.
Password reset isn't implemented. Google OAuth is the only sign-in, so we never hold a credential to reset. Documented in docs/security/password-recovery.md; needs a lecturer ruling.
Event correction is Partial. The schema supports it — corrections insert a revision and supersede the prior row — but the authorised workflow isn't built.
Imported scope diverges from ADR-003: 741 competitions against eleven recorded as agreed.
match_type: "IT20" is not a reliable international filter — 320 fixtures carry it while 5,635 further internationals carry T20. Use team_type = 'international'.
The printed ball number disagrees between paths. Ingestion derives it from a legal ball count; submission accepts it from the client unchecked.
Fixture and innings creation has no submission path, so innings penalty runs can't be submitted — and a team total is delivery totals plus innings penalties.
Conventions

Every change: an issue, a branch type/<issue>-slug, a reviewed PR. Nobody merges their own.
Commit bodies carry Refs #<issue> and, where AI helped, Assisted-by: Claude-Web[Claude Opus 5].
Every AI-generated document carries an AI Declaration; usage goes in evidence/ai/registers/ben-swartz.csv with a session transcript.
npm run check is the only merge gate — no CI runners.
Documentation style: formal, third person, numbered sections, imperative requirements, no marketing tone.
Practical traps

npm run <script> --workspace=@sport-analytics/backend -- <path> resolves paths relative to apps/backend, not the repository root.
Rebuild contracts after changing them or the backend validates against a stale dist: npm run build --workspace=@sport-analytics/contracts.
site/ is committed build output and dirties git status. Discard with git checkout -- site/.
Gitea auth fails intermittently; retrying works.
Corporate laptop: no admin rights, and a proxy delays local HTTP. Use curl.exe --noproxy "*" http://127.0.0.1:3000/....
Database tests run against a disposable embedded PostgreSQL: npm run test:database --workspace=@sport-analytics/backend.
Measure nothing against a long-running backend — see known issue 2.
How to work with me
Assume the context above. Be direct and disagree when I'm wrong — I'd rather be corrected now than at marking. Say when you're uncertain rather than asserting. If something is already built, tell me instead of helping me rebuild it: three times in Sprint 1 I started an issue someone had already finished.
For long files, ask me to paste the file rather than reconstructing it from greps. Working from fragments broke things twice.
Two recurring process problems worth knowing: PRs have merged without their final commit five times, because reviewers approve from a page opened before the last push; and work gets duplicated when nobody announces what they've picked up.

help me pick up new issues

I can’t see the Gitea board from here, so let me ground this in the project docs first.

Used 2 tools
I can’t see the Gitea board from here, so I can’t tell you what’s unclaimed — but I read the Sprint 2 rubric, and it changes the shape of the answer.

First, a correction. Your project instructions carry the Sprint 1 weighting (80% process). Sprint 2 is not that. Core Features 25% + Automated Testing 10% + API 15% is half the milestone, and the advanced band for Core Features is “implemented with at most one non-severe bug.” You are carrying eight known defects. You cannot fix your way to that band with one issue, which means triage matters more than picking the most interesting bug.

Do this today regardless of what you pick up. File all eight as Gitea issues. Bug Tracker is 5% and the advanced band is “extensive and continuous use” — eight defects discovered and none recorded is the opposite of the evidence you want, and the fix costs an afternoon. It also directly addresses your duplication problem: an issue on the board is the announcement you say nobody makes.

My read on severity, since Core Features hinges on it:

#8 (no fixture/innings submission path) — severe. A team total is delivery totals plus innings penalties, and penalties can’t be submitted. A published figure is structurally incomplete, which cuts against the brief’s traceability claim.
#7 (ball number disagrees between paths) — severe-ish. The same delivery gets a different printed number depending on how it arrived. Ingestion derives it; submission takes the client’s word. Cheap fix, and it’s squarely yours after #27 and #49/#50.
#4 (event correction Partial) — the brief names corrections explicitly, including carrying an accepted correction through to every aggregate downstream. “Schema supports it, workflow isn’t built” reads as not-implemented to a marker.
#2 (2.03s idle lag) — small but disproportionately risky. It hits /health, which touches no database, and a marker demoing a deployed backend sees it silently.
Deprioritise #1. There is no Performance criterion in the Sprint 2 rubric at all — it appears in Sprint 3 (5%) and Milestone 4 (5%). The 2,400 ms statistics endpoints feel like the biggest problem you have, and they are the wrong thing to spend the next 18 days on. File it, milestone it to Sprint 3.

Two criteria that may be entirely unowned. User Feedback is 10% and the advanced band requires a formal collection process plus evidence of integration — that takes calendar time, so it can’t be done in the last week. Automated Testing is 10% and the advanced band is “UI and API testing”; you have no CI runners and npm run check as the only gate. Neither appears anywhere in your handover. If nobody has claimed these, that’s 20% sitting loose and worth more than any single backend fix — raise it Thursday before you commit to a lane.

Cheapest thing that’s already yours: Database Documentation, 5%, advanced band is “full schema documentation, deployment info and motivation.” You did the migration, ADR-005 and the importer — you hold all of it already. #5 (ADR-003 divergence: 741 competitions vs eleven agreed) belongs in the same sweep and feeds Project Methodology evidence.

#3 (password reset) isn’t work, it’s a question. Take it to the tutor on Tuesday 1 September and record the ruling.

Anything I draft for you will need an AI declaration and a register line.

What should I work up first? (Check the board before claiming any of these — I can't see it.)
look at what i just pasted for you
WhatsApp Image 20260826 at 20.50.55.jpeg
Image
Image
[Display contextual weather on the fixture overview](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/300)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=41&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#300](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/300)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 17
[Expose contextual fixture weather using venue and match-date data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/299)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: backend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=116&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#299](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/299)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 14
[Conduct the Sprint 2 stakeholder review, user feedback round, retrospective and milestone tag](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/298)

[Generate accepted and rejected batch processing reports](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/279)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#279](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/279)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement asynchronous batch validation and processing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/278)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#278](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/278)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Implement whole-season and back-catalogue batch upload and staging](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/277)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#277](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/277)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Implement batch, batch-item and processing-checkpoint database models](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/276)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#276](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/276)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Design the batch staging, file storage and processing pipeline](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/275)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: research](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=33&milestone=0&assignee=&poster=)
[#275](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/275)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 10
[Complete the Basic security, privacy and dependency hardening review](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/274)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#274](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/274)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Complete the Basic accessibility and responsive-design audit and fixes](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/273)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#273](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/273)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Complete the full Basic integration and browser-level end-to-end acceptance workflow](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/272)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#272](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/272)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Conduct formal Basic user testing and integrate prioritised feedback](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/271)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#271](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/271)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Add filtered dataset export controls to the public data interface](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/270)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#270](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/270)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement filtered CSV and JSON dataset export endpoints](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/269)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#269](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/269)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Build the approved event-correction interface](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/268)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#268](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/268)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Add user-friendly file-upload and guided submission controls to the submitter interface](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/266)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#266](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/266)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)9 / 9
[Prepare the Sprint 2 formal user-testing protocol and evidence template](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/264)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: basic](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=45&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#264](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/264)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Run monorepo architecture and repository-hygiene checks in CI/CD](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/259)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=29&milestone=0&assignee=&poster=)
[#259](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/259)opened3 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Expose repository guides and Sprint evidence through the public documentation site](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/254)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=8&milestone=0&assignee=&poster=)
[#254](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/254)opened3 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 18
[Align the documentation site with project branding and improve external-link behaviour](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/253)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=8&milestone=0&assignee=&poster=)
[#253](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/253)opened3 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Establish Azure frontend deployment foundation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/17)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[type: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=29&milestone=0&assignee=&poster=)
[#17](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/17)opened3 weeks agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)13 / 13
[2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/17)
[Establish the Azure backend deployment foundation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/15)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[type: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=29&milestone=0&assignee=&poster=)
[#15](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/15)opened3 weeks agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)16 / 16
[2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/15)
[Restore green CI and reproducible installation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/10)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[type: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=29&milestone=0&assignee=&poster=)
[#10](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/10)opened3 weeks agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)13 / 16

[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: common](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=44&milestone=0&assignee=&poster=)
[type: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=8&milestone=0&assignee=&poster=)
[#298](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/298)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Publish Intermediate API, database, testing and third-party code documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/297)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=8&milestone=0&assignee=&poster=)
[#297](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/297)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Complete Intermediate integration, correctness, performance and API tests](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/296)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#296](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/296)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 11
[Build the dataset release catalogue and download experience](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/295)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#295](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/295)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement versioned dataset releases, snapshots, field documentation and checksums](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/294)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#294](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/294)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Implement caching for repeated reads with correct invalidation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/293)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#293](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/293)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Implement API key issuance, rotation, revocation, rate limits and quotas](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/292)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#292](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/292)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 9
[Implement explicit API versioning and compatibility behaviour](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/291)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#291](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/291)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Optimise database indexes, storage layout and query plans under load](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/290)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#290](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/290)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Create representative large-scale data and define API response-time targets](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/289)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#289](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/289)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Build golden reference results and automated correctness checks](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/287)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: documentation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=9&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: testing](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=40&milestone=0&assignee=&poster=)
[#287](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/287)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement dependency-aware selective recomputation after event changes](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/286)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#286](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/286)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement season, career and competition-wide aggregate derivation](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/285)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#285](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/285)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement immutable correction history and audit logs](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/284)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#284](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/284)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement submission review and publication workflow in the API and frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/283)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: frontend](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=35&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#283](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/283)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 8
[Implement impossible and conflicting event validation rules](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/282)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#282](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/282)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 7
[Implement resumable batch processing from durable checkpoints](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/281)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[area: infrastructure](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=30&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#281](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/281)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 6
[Make batch resubmission idempotent and prevent double counting](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/280)
[area: api](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=42&milestone=0&assignee=&poster=)
[area: data](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=43&milestone=0&assignee=&poster=)
[area: database](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=32&milestone=0&assignee=&poster=)
[priority: high](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=37&milestone=0&assignee=&poster=)
[tier: intermediate](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=221&milestone=0&assignee=&poster=)
[type: feature](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues?q=&type=all&state=open&labels=46&milestone=0&assignee=&poster=)
[#280](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/issues/280)opened2 days agoby[Shayna](https://sdp.ms.wits.ac.za/Shayna)[Sprint 2](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/milestone/12)[Sport-Analytics-Tool-Proj](https://sdp.ms.wits.ac.za/git-push-pray/Sport-Analytics-Tool/projects/2)0 / 6

That changes the picture. Reading the board against the roadmap:

What the board actually says
Week 1 is three days from closing and most of it hasn’t moved. Of the fourteen Week 1 targets on your roadmap, these are still open at 0/N: #253, #254, #268, #269, #270, #275, #276, #277, #291. Closed or done: #255, #256, #265, #267, and #266 sits at 9/9 waiting to be closed. So roughly two-thirds of Week 1 is untouched with Friday gone. That’s the conversation for Thursday’s standup, ahead of anything you personally claim.

Every issue in the milestone was opened by Shayna, and there is not one type: bug among them. Forty-odd issues, five work-type labels in use, zero bugs. Bug Tracker is 5% of Sprint 2 and the advanced band is “extensive and continuous use” — a milestone that is entirely a feature backlog does not read as a bug tracker to a marker. None of your eight defects are in the plan.

I can’t see assignees in what you pasted, so I still can’t tell you what’s free. Claim in Gitea before you start.

Your eight defects against the plan
Already covered — don’t rebuild:

#4 event correction → #267 (done), #268, #284. Your schema work is the foundation; leave it.
#1 statistics latency → #290, #293. Week 3.
Not covered anywhere, and each has a natural home:

#7 ball number unchecked on the submission path → belongs inside #282 (impossible and conflicting event validation). Same defect class, same issue.
#8 no fixture/innings submission path → nothing in the backlog creates fixtures or innings. Penalties can’t be submitted, so a team total is unreachable through the API. This will surface as a failure in #285 (aggregates) and #287 (golden reference correctness) and blow up in week 3 when someone tries to validate against a scorecard. Raise it now, not then.
#2 the 2.03 s idle lag → will silently corrupt #289’s response-time targets. Anyone measuring against a long-running backend gets numbers that are 2 s wrong.
#5 ADR-003 divergence, #6 IT20 filter → fold into #297.
#3 password reset → tutor question, Tuesday 1 Sept.
What I’d claim
#275 — design the batch staging, file storage and processing pipeline. This is the standout. It’s a Week 1 target at 0/10, it blocks #276, #277, #280 and #281, and that’s the entire Intermediate batch lane stalled behind it. It’s type: research, so the deliverable is a document, not a build — fast to produce. And you have already built the real version of this in #175: batched ingestion, 3.2 M deliveries, 0.05 → 0.28 matches/sec, eight days down to fifteen. Nobody else on the team can write that design from experience. If it’s unassigned, take it today.

Then, in your lane and in rough order:

#287 golden reference results — you already have this. Fixture 729307, 22 SQL assertions against a published scorecard. The issue is largely “generalise what Ben did in #27 and wire it into npm run check.”
#282 impossible and conflicting validation — direct continuation of your #49/#50 audit, and the home for the two unexplained 500s you already found plus defect #7.
#289 representative-scale data — you imported the corpus and did the measurements in #105. You hold both halves.
I’d leave #290 alone for now; it’s Week 3 and it needs #289’s targets to exist first.

One structural warning. The plan aims at full Intermediate by 15 September. The Sprint 2 advanced band for Core Features is “at most one non-severe bug” — you’re carrying eight, three of which I’d argue are severe. Breadth won’t reach advanced if the defects stay unrecorded and unfixed. Worth putting to Shayna as a plan question rather than filing eight issues into someone else’s milestone unannounced.

Want me to draft #275’s design document, or the eight bug issues in your methodology’s template first? Either will need an AI declaration and a register line in evidence/ai/registers/ben-swartz.csv.

Dates this conversation actually ran across, as the environment reported them:

28 August 2026 — session start (the env header when it opened; the conversation resumed from a compacted earlier session)
14 September 2026 — the bulk of the work: #475, #536, the retest records, the deployment incident
15 September 2026 — Sprint 2 milestone day: the retrospective, the rubric estimate, the stack and schema review
11 October 2026 — today
So the span is 2026-08-28 to 2026-10-11.

One thing to watch: your existing transcript is 2026-08-30-2026-09-14-continuous-project-sessions.md, which overlaps the 14 September portion of this chat. Either extend that file and rename it to the full span, or start a new one from where it stops — 2026-09-14-to-2026-10-11-continuous-project-session.md — and say in a line at the top of each which file carries the rest.

Don’t leave both claiming overlapping periods without saying so; that’s the sort of thing that looks careless when someone checks a register citation against a file.

not iclusing today, when did it go until?

15 September 2026.

The conversation ran 28 August to 15 September, then went quiet until today. The last substantive exchange before the gap was the Sprint 2 milestone work on the 15th — the retrospective draft, the mark estimate, the libraries and stack list, and the schema documentation review.

So excluding today: 2026-08-28 to 2026-09-15.





Claude is AI and can make mistakes.
