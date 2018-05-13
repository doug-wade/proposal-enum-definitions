- `enum` is a new _LexicalDeclaration_ binding form, similar to `let` or `const`.
- `typeof enum` is `object`; similar to Array, it's just a special Object (more below)

- _EnumDeclaration_ or _EnumExpression_ with a _BindingIdentifier_ creates:
  - A proto-less, frozen object;
  - Creates _PropertyName_ corresponding to each _EnumEntryName_:

    ```js
    enum DaysOfWeek {
      SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY
    };
    ```

    Is approximately\* equivalent to:

    ```js
    const DaysOfWeek = Object.freeze(
      Object.create(null, {
        [Symbol.enumSize]: {
          // Specification can define better semantics for deriving
          // and storing the size of the enum object (internal slot)
          value: 7
        },
        [Symbol.iterator]: {
          * value() {
            // Specification can define better semantics for deriving
            // and storing keys and values (internal slot)
            let keys = Object.keys(DaysOfWeek);
            let values = Object.values(DaysOfWeek);
            let index = 0;
            while (index < keys.length) {
              yield [keys[index], values[index]];
              index++;
            }
          }
        },
        values: function() {
          return Object.keys(DaysOfWeek)
        },
        SUNDAY: Symbol('SUNDAY'),
        MONDAY: Symbol('MONDAY'),
        TUESDAY: Symbol('TUESDAY'),
        WEDNESDAY: Symbol('WEDNESDAY'),
        THURSDAY: Symbol('THURSDAY'),
        FRIDAY: Symbol('FRIDAY'),
        SATURDAY: Symbol('SATURDAY'),
      })
    );

    for (let day of DaysOfWeek) { console.log(day.toString()) }

    /*
      SUNDAY
      MONDAY
      TUESDAY
      WEDNESDAY
      THURSDAY
      FRIDAY
      SATURDAY    
     */
    ```

- _EnumDeclaration_ or _EnumExpression_ will set the value of each entry to a symbol, using the name of the property as the first argument to the Symbol function (sometimes called the "description").

- _EnumDeclaration_ may have _EnumEntryName_ inline assignment overrides. If an entry has an inline assignment, the default value is not used for that entry.

  ```js
  enum MetasyntacticVariables {
    FOO,
    BAR = _AssignmentExpression_,
    BAZ,
  }
  ```

  Is equivalent to:

  ```js
  const MetasyntacticVariables = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      BAR: _AssignmentExpression_,
      BAZ: Symbol('BAZ'),
      // Other methods as defined above.
    })
  );
  ```

  _EnumDeclaration_ (with _BindingIdentifier_) or _EnumExpression_ example:

  ```js
  enum Things {
    FOO,
    BAR = _AssignmentExpression_,
    BAZ,
  };
  ```

  Is approximately equivalent to:

  ```js
  const DaysOfWeek = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      BAR: _AssignmentExpression_,
      BAZ: Symbol('BAZ'),
      // Other methods as defined above
    })
  );
  ```

  Of course, this means that the value of an `enum` entry can be whatever you want it to be:

  ```js
  enum ManyTypedValues {
    STRING = 'string',
    NUMBER = 42,
    SYMBOL,
    ARRAY = ['array', 'of', 'values'],
    OBJECT = { value: 42 },
    FUNCTION = () => 'function'
  }
  ```


- _EnumDeclaration_ may have _ComputedPropertyName_ as _EnumEntryName_:

  ```js
  enum MetasyntacticVariables {
    FOO,
    ["BAR"],
    BAZ,
  }
  ```

  Is equivalent to:

  ```js
  const DaysOfWeek = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      BAR: 'BAR'.toString(),
      BAZ: Symbol('BAZ'),
      // Other methods as defined above
    })
  );
  ```

  _EnumDeclaration_ (with _BindingIdentifier_) or _EnumExpression_ example:

  ```js
  enum Things {
    FOO,
    [Symbol(...)],
    BAZ,
  };
  ```

  Is approximately equivalent to:

  ```js
  const DaysOfWeek = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      [Symbol(...)]: Symbol(Symbol(...).toString()),
      BAZ: Symbol('BAZ'),
      // Other methods as defined above
    })
  );
  ```

- _EnumDeclaration_ may not have duplicate _EnumEntryName_:
  - These throw exceptions:
    ```js
    enum Foos {
      FOO,
      FOO
    }
    ```

    ```js
    enum Bars {
      FOO,
      ["BAR"],
      BAR,
    }
    ```

\* Approximately means that it doesn't fully represent all of the semantics.
