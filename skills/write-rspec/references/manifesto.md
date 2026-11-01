# The Testing Manifesto — Rails · RSpec

Good tests give us the freedom to change code without fear. They document intent, catch real
regressions, and stay out of the way. These are the principles we hold to.

## Philosophy

**1. Test behavior, not implementation.**
Verify what code promises to the outside world — never how it does it internally. When we
refactor, the tests survive; they break only when behavior actually breaks.

**2. Test our code, not the framework.**
We don't test that `validates :presence` works — that's Rails' job. We test the effect of *our*
rules and *our* logic.

**3. Few tests, high signal.**
The suite is minimal in quantity, maximal in quality. We keep the pyramid's shape, but every
layer stays lean: a handful of tests that genuinely catch bugs beats hundreds that only pad
coverage. If a test wouldn't fail for a real defect, it doesn't earn its place.

## Structure & Style

**4. Every test follows Arrange–Act–Assert.**
Set up the world, exercise the behavior, verify the outcome — in that order, clearly separated.

**5. Names describe behavior.**
An `it` block reads as a sentence and, on failure, tells us exactly what broke — without opening
the test body.

**6. DAMP over DRY.**
In test code, readability beats dryness. We avoid over-abstraction; no one should have to hunt
through three files to understand a single test.

## Dependencies & Layers

**7. Integration first, isolation last.**
We lean on real, end-to-end paths and test through the seams of the app rather than around them.
Mocks and stubs are reserved for true external boundaries — payment gateways, email providers,
third-party HTTP. Everything inside our own code runs for real.

**8. Request specs over controller specs.**
We exercise the full stack — routing, middleware, real HTTP — for realistic confidence.
Controller specs are the past.

## Data & Factories

**9. FactoryBot over fixtures.**
Factories give us readable, flexible setup — used with discipline, never as a dumping ground.

**10. Lean factories, rich traits.**
A factory defines only what's required to be valid. Variations live in traits
(`create(:user, :admin)`), not in bloated defaults.

**11. Faker for realistic data.**
We seed objects with varied, lifelike values instead of brittle hand-picked constants — closer
to production, quicker to expose hidden assumptions.

---

*We write tests we can trust, and delete the ones we can't.*
