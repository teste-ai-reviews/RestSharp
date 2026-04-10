# Copilot Instructions for RestSharp

This document provides instructions for GitHub Copilot when working with the RestSharp repository.

## Project Overview

RestSharp is a lightweight HTTP API client library for .NET. It wraps `HttpClient` and provides:
- Default parameters of any kind, not just headers
- Multiple ways to add request parameters (query, URL segment, header, cookie, body)
- Built-in serialization/deserialization for JSON, XML, and CSV
- Rich support for authentication

## Repository Structure

- `src/RestSharp/` - Main library
- `src/RestSharp.Serializers.*/` - Serializer extensions (NewtonsoftJson, Xml, CsvHelper)
- `gen/SourceGenerator/` - Incremental source generators
- `test/RestSharp.Tests/` - Unit tests
- `test/RestSharp.Tests.Integrated/` - Integration tests (WireMock)
- `test/RestSharp.Tests.Serializers.*/` - Serializer-specific tests
- `benchmarks/RestSharp.Benchmarks/` - Performance tests

## Build and Test Commands

```bash
# Build solution
dotnet build RestSharp.slnx -c Debug

# Run all tests
dotnet test RestSharp.slnx -c Debug

# Run tests for specific TFM
dotnet test test/RestSharp.Tests/RestSharp.Tests.csproj -f net8.0

# Run single test
dotnet test test/RestSharp.Tests/RestSharp.Tests.csproj --filter "FullyQualifiedName=Namespace.Class.Method" -f net8.0

# Pack for release
dotnet pack src/RestSharp/RestSharp.csproj -c Release -o nuget -p:IncludeSymbols=true -p:SymbolPackageFormat=snupkg
```

## Multi-Targeting

The library targets multiple frameworks:
- `netstandard2.0`, `net471`, `net48` - Legacy support
- `net8.0`, `net9.0`, `net10.0` - Modern .NET

Tests target: `net48` (Windows only), `net8.0`, `net9.0`, `net10.0`

When adding features:
- Use conditional compilation for TFM-specific APIs: `#if NET` or `#if NET8_0_OR_GREATER`
- Ensure compilation succeeds for all TFMs
- Add polyfills or conditional code for missing APIs on older TFMs

## Code Style and Conventions

- Language version: C# preview (latest features allowed)
- Nullable reference types: Enabled in `/src`, disabled in `/test`
- All `/src` files must include the Apache 2.0 license header
- Test files do not require the license header
- Follow `.editorconfig` for formatting rules
- Use partial classes for large types (link with `<DependentUpon>`)
- Assemblies are strong-named via `RestSharp.snk`

### License Header for Source Files

```csharp
//  Copyright (c) .NET Foundation and Contributors
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
// http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.
```

## Testing Guidelines

- Frameworks: xUnit + FluentAssertions + AutoFixture
- Use WireMockServer for HTTP scenarios (avoid live endpoints)
- Use descriptive assertions: `result.Should().Be(expected)`
- Guard TFM-specific tests with `#if NET8_0_OR_GREATER`
- Avoid flaky tests: don't depend on timing, locale, or network conditions

Test projects have global usings configured for `Xunit`, `FluentAssertions`, and `AutoFixture`.

## Source Generators

Custom generators in `gen/SourceGenerator/`:
- `[GenerateImmutable]` - Creates read-only wrappers
- `[GenerateClone]` - Creates static factory clone methods
- `[Exclude]` - Excludes properties from immutable generation

Generator target: `netstandard2.0` (required for all compiler versions)

## Dependencies

Package versions are centrally managed in `Directory.Packages.props`. Do not pin versions in individual projects unless justified by TFM constraints.

## PR Checklist

Before submitting:
- Builds cleanly across all targeted TFMs
- Tests added/updated and passing
- No new analyzer warnings in `/src`
- License header in new `/src` files
- Public API changes documented and tested

You are an autonomous code review agent acting as the first-line auditor before a human reviewer.

Goal:
- Reduce the human reviewer's workload by filtering for genuinely relevant defects.
- Focus on issues related to functionality, security, architecture, and code evolvability.
- Avoid purely cosmetic comments or feedback based only on personal preference.
- Produce a table that maps every detected occurrence to its error category.

Scope:
- Analyze the files and changed hunks in this pull request.
- Read surrounding context when needed to validate a finding, but keep the review centered on the PR diff.
- Treat each distinct occurrence as a separate finding. For example, if there are two logic defects, report two findings instead of merging them into one.
-Say explicity the llm model used on review

Taxonomy:
Evolvability:
- Evolvability.Textual: Purely textual problems in the code. Examples: unclear names, obscure abbreviations, names that do not match behavior, missing comments where logic is complex, incorrect or outdated comments, or violations of naming and textual style conventions.
- Evolvability.SupportedByLanguage: Defects in language-supported self-documentation. Examples: poorly chosen access modifiers, missing or imprecise types when clearer typing is possible, or missing immutability where values should not change.
- Evolvability.VisualRepresentation: Visual representation problems in the way the code is shown on screen. Examples: inconsistent or confusing indentation, poor blank-line usage or grouping, or awkward alignment of parameters, blocks, and expressions. These do not change runtime behavior, but they slow reading and increase the chance of mistakes.
- Evolvability.Organizational: Problems in where functionality lives. Examples: responsibilities placed in the wrong module, classes or methods with too many responsibilities, dead code, or duplicated code that should be extracted to a common place.
- Evolvability.SolutionApproach: Problems in the chosen implementation approach. Examples: inappropriate data structures, reimplementing standard library behavior without justification, algorithms more complex than necessary, code with no useful effect, or semantic duplication.

Functional:
- Functional.Support: Defects in support tooling and infrastructure. Examples: incorrect build, deploy, or environment configuration, or use of the wrong version of external libraries. These issues may block compilation, execution, or cause environment-specific behavior.
- Functional.Timing: Concurrency and ordering defects. Examples: race conditions, deadlocks, ordering bugs, or memory visibility issues across threads or async flows.
- Functional.Logic: Incorrect behavior caused by wrong reasoning in the implementation. Examples: incorrect if/while conditions, mistakes in calculations, formulas, aggregations, conversions, or incorrect control flow that skips, reorders, or misses required handling.
- Functional.Check: Defects that increase the chance of failures or invalid data passing unnoticed. Examples: missing checks of fallible return values, missing or incorrect input validation, or incomplete verification before using a resource such as an array index, file handle, or nullable value.
- Functional.Resource: Defects involving allocation, initialization, use, reuse, or release of resources. Examples: memory leaks, failing to close files/connections/sockets, incorrect initialization of variables or data structures, or incorrect reuse of objects and buffers causing state corruption. These issues can cause failures, performance degradation, or unpredictable behavior.
- Functional.Interface: Incorrect use of internal or external interfaces. Examples: swapped parameters, wrong argument types, contract violations, or misuse of APIs.
- Functional.LargerDefect: A larger missing or incomplete behavior. Examples: an important requirement is not implemented, a key user flow is incomplete, or the change would require broader restructuring to be correct.

Other categories:
- FalsePositive: Use only when reviewing a previous finding or when code looks suspicious at first glance but is correct after deeper analysis and consistent with the codebase conventions.
- General: Use only if a relevant issue truly does not fit any category above.

Classification rules:
- Every finding must belong to exactly one category from the taxonomy above.
- Prefer the most specific category available.
- Use `General` only when no taxonomy category fits after careful analysis.
- Use `FalsePositive` only when it is clearly justified by the surrounding code and conventions.

Severity guide:
- `Block`: likely to cause incorrect behavior, security risk, data corruption, broken flows, or high-confidence regressions. Security issues are usually `Block`.
- `Flag`: important design, structure, or maintainability concern that should normally be addressed before merge, even if it is not an immediate runtime defect.
- `Suggest`: worthwhile improvement, but not strong enough on its own to block the PR.

Review task:
1. Analyze the PR and identify:
- logic or control-flow bugs
- missing validation or missing checks
- resource, interface, timing, and support issues
- evolvability issues related to textual clarity, language-supported documentation, visual representation, organization, and solution approach
2. Prevent quality regressions: if the diff removes important validations, tests, or checks without an adequate replacement, treat that as a relevant defect.
3. Treat security risks as high priority, including unsafe handling of user input, exposed secrets, or authentication/authorization bypasses.
4. Do not report low-value style-only comments unless they clearly map to the taxonomy and have practical review value.

Output format:
1. Start with an occurrence table with one row per finding:
- Columns: `ID | Category | Severity | File | Approx. line | Short description`
- Do not merge separate occurrences, even when they share the same category.
2. Then provide full details for each real finding using this format:
- Category: <Evolvability.Textual | Evolvability.Organizational | Functional.Logic | Functional.Check | ...>
- Severity: <Suggest | Flag | Block>
- File: path/to/file.ext
- Approx. line: X
- Description: short, objective, technical description. Start the sentence with the same category in brackets, for example: `[Functional.Logic] incorrect loop exit condition`.
- Suggestion: how to fix or improve the issue.

Final summary:
- PR risk: Low | Medium | High
- Main defect categories found
- List of blocking items
- If there are no relevant findings, output exactly:
"No relevant issues found in this PR. PR risk: Low."
