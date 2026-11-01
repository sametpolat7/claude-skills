# Recognizing comments that tools read

A comment is tool-read when it has one of the shapes below. Position alone does not make it
tool-read. The examples show each shape; they are not a complete list.

## By position

- **File top** (the first lines, or before the first statement): shebangs, encoding and magic
  comments, parser directives, file-wide suppressions, reference directives, type-checker
  modes.
  `#!/usr/bin/env ruby` · `# frozen_string_literal: true` · `# -*- coding: utf-8 -*-` ·
  `# syntax=docker/dockerfile:1` · `/// <reference types="vite/client" />` · `// @ts-nocheck` ·
  `# typed: strict`
- **Directly above a target** (applies to the next line or statement): next-line
  suppressions, embed and export directives, safety notes, pure-call hints, and the C
  preamble above a cgo import.
  `// eslint-disable-next-line no-console` · `// biome-ignore lint/style/useConst: reason` ·
  `// @ts-expect-error reason` · `# hadolint ignore=DL3008` · `//go:embed static` ·
  `// SAFETY: …` · `/*#__PURE__*/`
- **End of line** (applies to that line): line suppressions, type ignores, and version notes
  that bots update.
  `# noqa: E501` · `# type: ignore[attr-defined]` · `# rubocop:disable Metrics/AbcSize` ·
  `// NOLINT` · `uses: actions/checkout@<sha> # v4.1.1`

## By form

- **A bare keyword or setting, not a sentence**: the whole comment is a lowercase word,
  optionally followed by `:` or `=` and a value, often with no space after the delimiter.
  `//go:build`, `//nolint:errcheck`, `#checkov:skip=…`, `/* rtl:ignore */`, `# fmt: off`,
  `//noinspection unchecked`, `// language=SQL`, `/*retries=*/3`
  A lone everyday word used as a label (`// helpers`, `# setup`) is not a setting: rule 14
  handles it.
- **Starts with a tool name**: `eslint-disable`, `prettier-ignore`, `biome-ignore`,
  `rubocop:`, `pylint:`, `istanbul ignore`, `c8 ignore`, `shellcheck`, `cspell:`,
  `markdownlint-`, `svelte-ignore`, `tfsec:ignore`
- **An `@` pragma, or a doc tag that changes what a tool does**: `@ts-…`, `@jsxImportSource`,
  `@license`, `@preserve`, `@generated`, `@deprecated`, `@internal`, `@vitest-environment`,
  `@jest-environment`, `@vite-ignore`. A descriptive tag (`@param`, `@returns`, `@throws`
  with prose) is an ordinary doc comment; only a type in it can be tool-read (see "Read by
  doc tools").
- **A delimiter with special meaning**:
  - `/*!`: kept by minifiers, and executed by MySQL;
  - `/*+`: an SQL optimizer hint;
  - `/**` versus `/*`: doc tools read only `/**`;
  - `///` and `//!`: doc comments in some languages, directives in others;
  - `#:` and `---@`: type or doc annotations;
  - `<%#`: a template comment that may declare inputs.
- **A marker a tool matches**: `NOLINT`, `NOSONAR`, `DO NOT EDIT`, `FALLTHROUGH`, `SAFETY:`,
  `MARK:`, `Deprecated:`, `Output:`, `region`/`endregion`. Task and emphasis words (`TODO`,
  `FIXME`, `HACK`, `XXX`, `NOTE`, `IMPORTANT`, `WARNING`) are not tool markers: rule 8
  handles tasks and rule 14 handles emphasis.
- **Carries data a tool parses** (key=value, JSON, rule IDs, schema URLs, a version next to a
  pinned hash): `# yaml-language-server: $schema=…`, `/* webpackChunkName: "x" */`,
  `# renovate: datasource=…`
- **Part of a pair** (start/end, begin/end, disable/enable, on/off, up/down):
  `-- +goose Up`, `# BEGIN MANAGED BLOCK`
- **A tool keyword inside prose**: `:reek:`, `NOSONAR`, `gitleaks:allow`,
  `pragma: allowlist secret`

## By what it does

- **Runs as a test**: doctest prompts (`>>>`, `iex>`), code blocks in doc comments,
  expected-output comments in examples, type-assertion comments.
- **Read at runtime**: docstrings used as CLI help, API descriptions or grammar rules;
  docblock annotations that frameworks read (routes, ORM mappings, test data providers);
  template comments that declare inputs.
- **Read by doc tools**: deprecation notices, visibility markers (`@internal`, `@private`,
  `:nodoc:`) and inheritance markers (`{@inheritDoc}`, `<inheritdoc/>`). Also doc-comment
  types the code does not declare: every type in a language without type declarations, and
  any type more precise than the declared one (element types, generics, unions, shapes),
  which static analyzers read.
- **Generated**: blocks a tool writes and rewrites (schema annotations, managed blocks,
  tables of contents).

## Unused suppressions

Remove a suppression only when its own tool, run on the unedited file with the project's full
configuration, reports it unused. Examples: ESLint `--report-unused-disable-directives`, ruff
`RUF100`, RuboCop `Lint/RedundantCopDisableDirective`, and TypeScript's "Unused
'@ts-expect-error' directive". Never correct a suppression's syntax, because that can change
what it suppresses.
