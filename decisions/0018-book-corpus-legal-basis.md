# ADR-0018: Legal basis for the book corpus behind Lampada articles

- Status: Accepted
- Date: 2026-10-17
- Ticket: ClickUp 123pfqn37tp
- Approval: Maria, 2026-10-17

## Context

Articles on lampada.app are written from books by Christian authors on prayer
and spiritual life instead of the team's own experience. The books are kept in
a search corpus (channel `lampada` in channel-ai): full texts are extracted,
split into fragments and embedded on a company server; articles retell and
quote the fragments with a reference to the book. Lampada is free and has no
monetisation.

Restricting the corpus to the public domain would leave only old books and
exclude most living authors on the topic. The question is what makes the use
of a copyrighted book lawful.

## Decision

1. **Any lawfully obtained copy may enter the corpus**, regardless of the
   book's age: a purchased copy, a copy published by the rights holder or with
   their permission (for example on predanie.ru), or a public-domain text.
   Pirated copies are excluded even when they are easy to find. The source of
   each book's copy is recorded with the book.
2. **Articles publish retellings and quotations, not the books.**
   - A retelling conveys the author's ideas in our own wording and our own
     structure, with a reference to the book. It does not follow the book's
     sequence chapter by chapter or keep its distinctive wording, and it does
     not attribute to the author views or words they did not express.
   - A quotation is verbatim and names the author, the work and the chapter or
     page; for a published translation the translator is named too. Its length
     is limited to what the article's point needs. Editorial guideline, not a
     legal threshold: one quotation stays within a short passage of a few
     sentences, and quotations never carry the article.
3. **The corpus is an internal tool.** Its fragments are not served to
   Lampada users or published as such. Doing so would be distribution of the
   text and needs a separate decision.
4. **Source terms are honoured.** predanie.ru requires a reference and a
   hyperlink to the material's page naming Predanie.ru, forbids changes to its
   materials (shortening only without distorting the meaning) and requires a
   written contract for commercial use. Quotations from such sources carry
   that link.

## Rationale

- **Ideas are not protected.** Copyright does not extend to ideas, concepts,
  principles, methods, processes, systems, solutions, discoveries or facts
  (Civil Code of the Russian Federation, art. 1259 §5). A retelling of what an
  author teaches, in our own wording and structure, uses no protected
  expression. A close paraphrase or a chapter-by-chapter summary that follows
  the book's sequence, structure or distinctive wording can be a derivative
  work (переработка, art. 1270 §2(9)), which needs consent; point 2 excludes it.
- **Quotation is a statutory free use.** Art. 1274 §1(1) allows quoting, in
  the original and in translation, for scientific, polemical, critical,
  informational and educational purposes, or to reveal the author's creative
  intent, of lawfully made public works, in a volume justified by the purpose
  of quoting, without consent and without payment, but with mandatory naming of
  the author and the source. The law sets no percentage.
- **Moral rights.** The rights of attribution (art. 1265) and to the integrity
  of the work (art. 1266) are why quotations are verbatim and attributed and why
  a retelling must not distort the author's views.
- **Internal analysis of a lawful copy elsewhere.** The EU allows text and
  data mining of lawfully accessible works unless the rights holder has
  expressly reserved it in an appropriate manner (machine-readable means for
  content made publicly available online; for other content, such as a
  purchased e-book, a notice or the licence terms suffice); copies may be kept
  only as long as the mining needs them (Directive (EU) 2019/790, art. 4). In
  the United States, *Bartz v. Anthropic* (N.D. Cal., order of 23 June 2025,
  partial summary judgment) held that using books to train a language model,
  and scanning purchased print books into a digital library, was fair use,
  while downloading and keeping pirated copies in a permanent library was not;
  the case then settled. Both are comparative only: neither governs us, the US
  ruling is a district-court decision, and neither decides on outputs that
  reproduce the books.
- **Residual risk in Russian law.** Storing a work in computer memory is
  reproduction (art. 1270 §2(1)); only a short-term, temporary or incidental
  record that is an integral part of a technological process and has no
  independent economic value is not, and a persistent corpus does not
  qualify. The personal-use exception (art. 1273) covers individuals only.
  Russian law has no general text-and-data-mining exception. Federal Law
  No. 243-FZ of 26.07.2026 on artificial intelligence is reported to exempt
  analysis of works only for training large foundation models; published
  summaries do not extend it to a search corpus. Its text was not checked
  against the official source when this ADR was written; verify on
  pravo.gov.ru before relying on it. The risk is low because the copies are
  lawful, never leave the corpus and only free uses (retelling, quotation) are
  published. Points 1 to 3 are what keep it low.

## Consequences

- Book selection for the corpus is driven by relevance and quality, not by the
  public domain.
- channel-ai's book `license` field must be able to record a purchased copy;
  its current values (`public_domain`, `noncommercial_attribution`,
  `rightsholder_permission`) do not cover it. The translator is already a book
  field.
- The quote checker and the article prompts enforce point 2: every quotation
  and retelling is tied to a corpus fragment and verified against it.
- Serving corpus fragments in the Lampada app, or publishing them elsewhere,
  requires a new ADR.

## References

- Civil Code of the Russian Federation, arts. 1259, 1265, 1266, 1270, 1273,
  1274 ([art. 1274](https://legalacts.ru/kodeks/GK-RF-chast-4/razdel-vii/glava-70/statja-1274/))
- [Directive (EU) 2019/790, art. 4](https://www.legislation.gov.uk/eudr/2019/790/article/4)
- [Bartz v. Anthropic, summary](https://www.akingump.com/en/insights/ai-law-and-regulation-tracker/district-court-rules-ai-training-can-be-fair-use-in-bartz-v-anthropic)
- [predanie.ru terms of use](https://predanie.ru/page/tos)
