# The Luhn Algorithm

## What it is

The **Luhn algorithm**, also called the **Luhn formula**, **modulus 10 algorithm**, or **mod 10 algorithm**, is a checksum algorithm used to detect accidental errors in sequences of decimal digits.

It was developed by IBM scientist Hans Peter Luhn and described in a patent filed in 1954. It is commonly associated with payment card numbers and is referenced by the ISO/IEC 7812 numbering system.

The algorithm does not prove that a card or identifier exists. It only verifies that the sequence of digits is internally consistent with the Luhn checksum.

## How it works

For a number that already includes its check digit:

1. Start from the rightmost digit.
2. Moving from right to left, double every second digit.
3. If a doubled value is greater than 9, subtract 9.
4. Add all resulting digits.
5. The number passes the Luhn check when the total is divisible by 10.

Equivalent transformation for a doubled digit `d`:

```text
transformed(d) =
    d * 2       when d * 2 <= 9
    d * 2 - 9   when d * 2 > 9
```

The subtraction of 9 is equivalent to adding the two digits of the doubled result:

```text
8 * 2 = 16
1 + 6 = 7
16 - 9 = 7
```

## Worked validation example

Consider the test number:

```text
79927398713
```

Starting from the right, the rightmost digit is the check digit. Double every second digit to its left:

| Original digit | Operation | Result |
|---:|---|---:|
| 7 | double: 14, subtract 9 | 5 |
| 9 | unchanged | 9 |
| 9 | double: 18, subtract 9 | 9 |
| 2 | unchanged | 2 |
| 7 | double: 14, subtract 9 | 5 |
| 3 | unchanged | 3 |
| 9 | double: 18, subtract 9 | 9 |
| 8 | unchanged | 8 |
| 7 | double: 14, subtract 9 | 5 |
| 1 | unchanged | 1 |
| 3 | check digit, unchanged | 3 |

The sum is:

```text
5 + 9 + 9 + 2 + 5 + 3 + 9 + 8 + 5 + 1 + 3 = 59
```

The table above intentionally demonstrates why parity must be implemented carefully: doubling positions from the wrong side gives an invalid result. A safer right-to-left validation procedure produces:

```text
3 + 2 + 7 + 7 + 9 + 6 + 9 + 4 + 9 + 9 + 5 = 70
```

Since:

```text
70 mod 10 = 0
```

the number is valid.

## Generating a check digit

Suppose the payload is:

```text
7992739871
```

To calculate the missing check digit:

1. Process the payload as though a check digit will be appended.
2. Double every second digit, starting from the rightmost payload digit.
3. Sum the transformed values.
4. Calculate:

```text
check_digit = (10 - (sum mod 10)) mod 10
```

For this payload, the transformed sum is `67`:

```text
check_digit = (10 - (67 mod 10)) mod 10
check_digit = (10 - 7) mod 10
check_digit = 3
```

The complete valid number is:

```text
79927398713
```

## Pseudocode

```text
function isValidLuhn(number):
    sum = 0
    shouldDouble = false

    for each digit from right to left:
        value = digit

        if shouldDouble:
            value = value * 2

            if value > 9:
                value = value - 9

        sum = sum + value
        shouldDouble = not shouldDouble

    return sum mod 10 == 0
```

## PHP example

```php
<?php

declare(strict_types=1);

function isValidLuhn(string $number): bool
{
    $normalized = preg_replace('/[\s-]+/', '', $number);

    if ($normalized === null || $normalized === '' || !ctype_digit($normalized)) {
        return false;
    }

    $sum = 0;
    $shouldDouble = false;

    for ($index = strlen($normalized) - 1; $index >= 0; --$index) {
        $digit = (int) $normalized[$index];

        if ($shouldDouble) {
            $digit *= 2;

            if ($digit > 9) {
                $digit -= 9;
            }
        }

        $sum += $digit;
        $shouldDouble = !$shouldDouble;
    }

    return $sum % 10 === 0;
}

var_dump(isValidLuhn('79927398713')); // true
var_dump(isValidLuhn('79927398714')); // false
```

### Check digit generation in PHP

```php
<?php

declare(strict_types=1);

function generateLuhnCheckDigit(string $payload): int
{
    if ($payload === '' || !ctype_digit($payload)) {
        throw new InvalidArgumentException(
            'The payload must contain decimal digits only.'
        );
    }

    $sum = 0;
    $shouldDouble = true;

    for ($index = strlen($payload) - 1; $index >= 0; --$index) {
        $digit = (int) $payload[$index];

        if ($shouldDouble) {
            $digit *= 2;

            if ($digit > 9) {
                $digit -= 9;
            }
        }

        $sum += $digit;
        $shouldDouble = !$shouldDouble;
    }

    return (10 - ($sum % 10)) % 10;
}

$payload = '7992739871';
$checkDigit = generateLuhnCheckDigit($payload);

echo $payload . $checkDigit; // 79927398713
```

## Applications

Typical applications include:

- payment card Primary Account Numbers;
- IMEI identifiers;
- some national, healthcare, insurance, and administrative identifiers;
- validation of numeric codes in data-entry systems;
- reducing false positives when detecting card-like numbers in text.

The exact rules of an identifier must still be checked. Passing Luhn is only one possible validation requirement.

## Error detection

Luhn detects:

- every single-digit substitution;
- most accidental swaps of adjacent digits;
- many common typing mistakes.

It does not detect every possible transposition. For example, some swaps involving digits whose difference is 9 can remain valid, such as combinations related to `09` and `90`.

## Advantages

- Very simple to implement.
- Linear time complexity: `O(n)`.
- Constant additional space: `O(1)`.
- Fast enough for client-side and server-side validation.
- Detects many common data-entry errors.
- Does not require a database or external service.

## Limitations

- It is not encryption.
- It is not a cryptographic hash.
- It does not authenticate the issuer or owner.
- It does not prove that a payment card is active.
- It is easy to generate a number that passes the check.
- It does not detect all transposition errors.
- It validates only the checksum, not the complete business format.

A payment card form should therefore distinguish between:

```text
syntactically plausible
Luhn-valid
issuer-format-valid
actually issued
active and usable
```

These are different levels of validation.

## Common implementation mistakes

### Starting from the wrong side

The doubling pattern depends on whether the number already includes the check digit.

- Validation: begin with the rightmost digit unchanged.
- Check-digit generation: process the payload as if the new check digit occupied the rightmost position.

### Treating the number as an integer

Identifiers should be represented as strings because:

- they can be longer than the safe integer range;
- they may contain leading zeroes;
- no arithmetic operation is required on the whole value.

### Confusing checksum validity with real-world validity

A random number can be constructed to satisfy Luhn. It must not be treated as proof that a card, account, or identity exists.

### Ignoring normalization

User input may contain spaces or hyphens:

```text
4242 4242 4242 4242
4242-4242-4242-4242
```

Normalize explicitly, but reject unexpected characters rather than silently removing everything.

## Mental model

```text
Input number
    |
    v
Read digits from right to left
    |
    v
Double every second digit
    |
    v
Subtract 9 when doubled value exceeds 9
    |
    v
Add all values
    |
    v
Is the sum divisible by 10?
    |
    +-- Yes --> checksum valid
    |
    +-- No  --> checksum invalid
```

## Evolution and alternatives

Luhn belongs to the broader family of check-digit algorithms. Other approaches include:

- **Verhoeff algorithm**: detects all single-digit errors and all adjacent transpositions, but is more complex.
- **Damm algorithm**: also detects all single-digit errors and adjacent transpositions using a quasigroup table.
- **ISBN-10 checksum**: uses modulus 11 and position-dependent weights.
- **IBAN validation**: uses a modulus 97 calculation over rearranged alphanumeric data.
- **CRC algorithms**: designed for error detection in larger binary data streams.
- **Cryptographic message authentication codes**: designed to detect deliberate tampering when a secret key is available.

The correct choice depends on the threat model. Luhn is suitable for accidental input errors, not adversarial manipulation.
