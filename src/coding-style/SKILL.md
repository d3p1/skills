---
name: coding-style
description: >
  Enforce project coding style conventions: naming, comments,
  readability, formatting, and structural consistency.
  Use this skill after writing or modifying code, and before
  considering an implementation task done, to check it against
  the desired code style.
---

# Goal

Enforce clean, maintainable, consistent, and highly readable code.

Ensure code follows project-specific
naming standards, formatting practices,
documentation patterns, and repository consistency.

Prefer consistency with the existing repository style unless it is
clearly harmful.

Always inspect nearby files, project configuration, formatter rules,
and repository conventions before applying generic standards.

Use any available tools (like the ones related to JetBrains IDEs), 
available linters, formatters, static analysis tools, and type
checkers to improve development accuracy.

## Formatting

Prefer:

* 80-column line width
* 4-space indentation

Always try to keep contiguous elements aligned, in both code and
documentation. For example:

```javascript
let aaa = 'test1'
let b   = 'test2'

/**
 * Function description
 *
 * @param   {type} variableName Description
 * @returns {type} Description
 * @throws  {type} Description
 * @note    Additional clarification
 */
function test(...) {
    ...
}
```

If the project runs an automatic formatter (e.g. Prettier, gofmt)
that removes manual alignment, defer to it rather than fighting it —
a project's enforced formatting conventions always take priority
over this preference.

## Naming conventions

Respect language ecosystem standards.

Additionally, take into consideration the 
following naming conventions:

### Boolean variables

Boolean variables should read naturally:

```text
isEnabled
hasAccess
canRetry
shouldUpdate
```

### Functions

Functions should use verbs describing behavior:

```text
calculateTotal
fetchUser
validateSchema
buildPayload
```

### Classes

Classes should use descriptive nouns:

```text
PaymentProcessor
SessionManager
NotificationService
```

### Extra: Protected & Private

Prefix every protected or private function and property
with an underscore (`_`). For example:

```php
...
protected $_prop;
...
private function _privFunc() {
  ...
}
```

Use this convention in most languages, except in languages that
already provide special syntax for private or protected members.

For example, in JavaScript, private properties 
and methods are prefixed with `#`:

```javascript
...
#privateProp
...
#privateFunc() {
  ...
}
```

Do not use the underscore-prefix style for this language (and similar ones);
use the language's native syntax instead.

Also use the underscore (`_`) prefix for functions that are only used
internally by other functions. This happens a lot in bash scripts.
For instance:

```bash
#!/bin/bash
...
main() {
  ...
  _process
  ...
  exit 0
}
...
_process() {
  ...
}
...
main "$@"
```

## Documentation

Avoid redundant or obvious comments.

Only add comments when they provide:

* meaningful clarification
* architectural or business context
* explanation for non-obvious logic

However, always prefer the creation of
protected and private methods/functions
with descriptive names to encapsulate
logic in a self-explanatory way.

### File

Files should contain descriptive headers similar to:

```javascript
/**
 * @description Explains file purpose
 * @author      C. M. de Picciotto <d3p1@d3p1.dev> (https://d3p1.dev/)
 * @note        Additional clarification
 */
```

Check nearby files to determine the correct information
that should be used for the `@author`.
In most cases, it will be:
`C. M. de Picciotto <d3p1@d3p1.dev> (https://d3p1.dev/)`
which is my personal information.
However, the work may be under
an agency account instead. In that case, use the corresponding
agency author information (for example, an account with an agency email).
Always check nearby files to determine which author
information should be used.

### Function

Functions and methods should follow 
documentation patterns similar to:

```javascript
/**
 * Function description
 *
 * @param   {type} variableName Description
 * @returns {type} Description
 * @throws  {type} Description
 * @note    Additional clarification
 */
```

### Variable

Document variables or properties using 
documentation similar to:

```javascript
/**
 * @type {type}
 * @note Additional clarification
 */
#privateProp
```

### Extra: `@note` and `@link`

For documentation related to logic inside
a function body, or additional clarifications
regarding function logic, use
the `@note` tag.

Additionally, when there is an issue link related to the specific
implementation, or a link to useful documentation that motivates
an implementation, include it with the `@link` tag.

For instance:

```javascript
/**
 * Add logic that handles vertex/point/pixel displacement to image
 *
 * @param   {string} noiseImageSrc
 * @param   {number} noiseFrequency
 * @param   {number} noiseAmplitude
 * @param   {number} displacementFrequency
 * @param   {number} displacementAmplitude
 * @returns {void}
 * @note    Take into consideration that the pointer canvas/image/texture
 *          is called `uDisTexture` inside the shader because
 *          it is considered that the shader does not need to know
 *          that this texture is related to a pointer
 * @note    Force shader compilation with `compile()`.
 *          If it is not forced the shader compilation, then
 *          uniforms will be undefined until first render of the scene
 * {@link   https://github.com/mrdoob/three.js/pull/10960}
 */
#addDisplacementHandlerToImage(...) {
   ...
}
```
