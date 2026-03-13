# YAML Notes (Quick Reference)

## What is YAML?
YAML (**YAML Ain’t Markup Language**) is a human-readable data-serialization format (often used instead of JSON) commonly used for:
- configuration files (docker, app-properties, etc.)
- CI/CD pipelines (GitHub Actions, etc.)
- Can reduce token used to communicate with AI tools

Note:
- YAML works on indentation (ie. every indent level = one level deeper in the property hierarchy).
- Indentation is the equivalent of dots (.)

---

## Headings / Formatting Tip
YAML has no “heading” syntax like Markdown. If you want structure, use keys and nesting, for example:
```yaml
h1:
  h2: value1      #h1.h2 ===h1+'\n'+'(2 spaces)'+h2
  h3:
    h4: value3
```

application.properties(spring) equivalent:
```yaml
h1.h2=value1
h1.h3.h4=value3
```

---

## Comment
```yaml
# This is a comment
```

---

## Basic Mapping (Key: Value)
```yaml
chai_name: Masala & Chai                      #might give error
description: "Chai with cardamom & ginger"
tagline: "The best chai in town"
```

---

## Multiline Strings
### Literal block (keeps new lines): `|`
```yaml
brewing_instruction: |
  boil water
  add tea leaves
  add milk
```

### Folded block (folds lines into spaces): `>`
```yaml
brewing_instruction_two: >
  boil water
  add tea leaves
  add milk
```

---

## Lists (Sequences)
```yaml
spices:
  - ginger
  - cloves
  - black_pepper
  - true            #multi valued type
```

Inline list:
```yaml
spices: [cardamom, ginger, cloves]
```

---

## Numbers, Booleans, Null
```yaml
cups_per_day: 3
cups_per_day_two: 0xFF22FF   # hex number
cups_per_day_two: 3.2342     # floating point
cups_per_day_two: 3.5e+3     # scientific notation

is_hot: true/false                 # boolean
add_sugar: yes/no                  # YAML boolean-style (commonly treated as true in many parsers)
instant: on/off                    # YAML boolean-style

sweetener: null                    # null
alternative_milk: ~                # null shorthand
```

---

## Dates / Timestamps (often parsed as date-time)
```yaml
morning_brew: 2025-01-15             #YYYY-DD-MM
local_time: 2025-01-15 08:30:01      #YYYY-DD-MM HH-MM-SS
```

---

## Type Casting (Explicit Tags)
```yaml
zip_code: !!str 12345      #to string
count: !!int "123"         #to int
```

---

## Set Creation
```yaml
unique_spices: !!set
  ? cardamom
  ? ginger
  ? cloves
```

---

## Anchors & Aliases (Reuse blocks)
```yaml
default_chai_base: &default_base
  tea: black_tea
  water: 200ml
  brewing_time: 5

morning_chai:
  <<: *default_base        #same values as default_chai_base
  milk: 100ml

evening_chai:
  <<: *default_base        #same values as default_chai_base
  milk: 200ml
```

---

## Nested Lists + Objects (Categories Example)
```yaml
chai_categories:
  - name: Traditional
    varieties:
      - Masala Chai
      - Ginger Chai
      - Cardamom Chai

  - name: Modern
    varieties:
      - Vanilla Chai
      - Chocolate Chai
```

Java equivalent:
```yaml
public record ChaiCategory (
    private String name,
    private List<String> varieties
)

List<ChaiCategory> chaiCategories = List.of(
    new ChaiCategory(
        "Traditional",
        List.of("Masala Chai", "Ginger Chai", "Cardamom Chai")
    ),
    new ChaiCategory(
        "Modern",
        List.of("Vanilla Chai", "Chocolate Chai")
    )
);
```

---

## Array of Objects (JSON API request/response)
```yaml
menu:
  - id: 1
    name: "Classic Masala Chai"
    price: 25
    ingredients: [black_tea, milk, sugar, cardamom, ginger]
    available: true

  - id: 2
    name: "Adrak Chai"
    price: 35
    ingredients: [black_tea, milk, sugar, ginger]
    available: true
```

JSON equivalent:
```yaml
{
  "menu": [
    {
      "id": 1,
      "name": "Classic Masala Chai",
      "price": 25,
      "ingredients": ["black_tea", "milk", "sugar", "cardamom", "ginger"],
      "available": true
    },
    {
      "id": 2,
      "name": "Adrak Chai",
      "price": 35,
      "ingredients": ["black_tea", "milk", "sugar", "ginger"],
      "available": true
    }
  ]
}
```

