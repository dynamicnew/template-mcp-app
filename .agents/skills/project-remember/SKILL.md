---
name: project-remember
description: Record a verified, durable project lesson in docs/ai/memory.md or a decision in docs/ai/decisions.md. Use only when the fact will matter in a future session.
---

# Record a lesson

1. Confirm the fact is verified: you ran a check or read the source. Treat
   prior notes and memory as context to verify, not proof. If unverified, note
   it as an unknown in `docs/ai/handoff.md` instead.
2. Skip it if it is cheap to rediscover, specific to one task, or already
   covered. Inspect existing project docs and `docs/ai/memory.md` first so you
   update the right canonical doc instead of creating duplicate or conflicting
   records.
3. Add one entry above `## Entry format` in `docs/ai/memory.md`: a one-sentence
   fact, the evidence (file, command or commit) and today's date.
4. A choice that future work must respect, with its reason, goes in
   `docs/ai/decisions.md` instead. Keep durable shared decisions in reviewed
   repository docs, not only in session or expiring memory.
5. Keep project lessons in this repository. Do not modify user-level or other
   repositories' instruction or memory files without explicit authorization.
   Never record secrets, personal data, raw transcripts or guesses.
6. Remove or correct an entry that has stopped being true.
