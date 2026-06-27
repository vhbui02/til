# When to inline vs Extract into a component?

Extract when at least **TWO** of these hold, otherwise inline:
1. It will be rendered in 2+ place by the same parent, or different parents.
2. It has its own internal complexity.
3. Its props describe itself and do not depend on the parent's state machine.

A presentational component must not own layout that depends on where it is used.

> E.g., a child component hardcodes `position: absolute; top: X; right: Y;` which is more appropriate to be the parent's decision.
