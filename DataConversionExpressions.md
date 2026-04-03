# Data Conversion & Expressions
*Solving the "Data Type" error using toInt() and toDouble().*

Sometimes, data arrives in a format that your UI component doesn't understand. This is the most common cause of "Red X" errors on a dashboard.

### The "String" Trap
When your ESP32 board sends a light reading, it often packages it as text (a **String**), like `"400"`.
However, an analog gauge requires a strict number (**Integer** or **Double**) to move its needle. If you feed a String into a gauge, it breaks.

### The Fix: Expression Bindings
Instead of a standard Tag Binding, we use an **Expression Binding**. This allows us to intercept the data and "cast" (convert) it right before it hits the gauge.

By wrapping the tag path in a function, we force the translation:
* `toInt({path/to/tag})` — Converts the text `"400"` into the whole number `400`.
* `toDouble({path/to/tag})` — Converts the text into a decimal number like `400.0`.

*Pro Tip: If your gauge is still showing an error, check if it requires a "Double" specifically!*
