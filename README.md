# Engineering

Cross-repository engineering documentation for SEKTEK projects. This repository
holds decisions and guidance that apply to more than one project, so each
project doesn't have to restate them.

Use it to find out why we build things the way we do, and to record a new
decision that affects several repositories.

## Contents

- [`decisions/`](decisions/): engineering decision records (EDRs).

### Decisions

An engineering decision record (EDR) captures one engineering decision: the
context that led to it, what we decided, and the tradeoffs we accepted. Each
record is a single Markdown file in [`decisions/`](decisions/).

- [0001](decisions/0001-prefer-typescript-access-modifiers.md): prefer
  TypeScript `private` and `protected` over JavaScript `#` hard-private
  fields.

## Read a decision

Each EDR starts with front matter that gives its `title`, `status`, `date`, and
`tags`. The body follows a fixed order:

1. **Context:** the problem and the forces behind it.
2. **Decision:** what we decided, with guidelines you can apply directly.
3. **Rationale:** why we chose it.
4. **Consequences:** the positive and negative results we accept.
5. **Alternatives considered:** the options we rejected and why.
6. **Implementation guidance:** examples of the decision applied.

If you only need to apply a decision, read its Decision section.

## Propose a decision

Record a decision when it affects more than one repository, or when someone
joining a project would otherwise have to ask why it works that way. Decisions
that affect a single repository belong in that repository's own documentation.

1. Pick the next number. Find the highest-numbered file in
   [`decisions/`](decisions/) and add one.
2. Create `decisions/NNNN-short-description.md`. Use four digits for `NNNN`
   and lowercase words separated by hyphens for the description.
3. Add the front matter and the six sections described in
   [Read a decision](#read-a-decision). Copy the structure of an existing
   record, such as
   [0001](decisions/0001-prefer-typescript-access-modifiers.md).
4. Add the record to the [Decisions](#decisions) table in this file.
5. Open a pull request. Merging the pull request accepts the decision.

Don't edit an accepted record to reverse it. Write a new record that
supersedes it, and say which record it replaces.

## Writing conventions

Follow the technical-writing skill for style and language. In addition:

- Wrap prose at 80 columns.
- Use exact identifiers, flags, and package names from the source.
- Keep examples short and limited to the behavior the decision describes.
