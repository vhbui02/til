---
description: A list of general guidelines of writing and reviewing clean, maintainable and human-readable code to ensure coding consistency and quality.
applyTo: "**"
---

# General Guidelines

## Separation of Concerns (SoCs)

- Keep related code together.
- Organize code in a logical hierarchy.

## Constants Over Magic Numbers and Strings

- Refactor hard-coded magic strings and numbers into named constants.
- Use semantic, descriptive constant names that explain the value's purposes.
- Write constants at the top of the file or in a dedicated constants file.

## Naming Convention

- Analyze existing files, directories, variables, functions, classes name to learn existing naming convention, therefore keep new artifacts naming consitency.
- Name MUST be semantic, reveal their purpose by explaining why something exists and how it's used.

## Encapsulation

- Functions and classes MUST hide implementation details and expose clear interfaces.
- Move nested conditionals into well-named functions

## Single Responsibility Principle (SRP)

- Each class/function MUST doexactly one thing.
- Class/Functions MUST be small and focused.
- When a class/function requires a comment to explain what it does, it MUST be splitted.

## Don't Repeat Yourself (DRY)

- Extract repeated code into reusable functions.
- Share common logic through proper abstraction.
- Design and maintain single sources of truth.

## Test-included Development

- Write tests for any implementation when receiving explicit requirements from user.
- Write edge cases.
