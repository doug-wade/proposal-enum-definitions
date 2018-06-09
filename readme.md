- `enum` is a new _LexicalDeclaration_ binding form, similar to `let` or `const`.
- `typeof enum` is `enum`; unlike Array, there's no other way to determine its enum-ness, as there is not a global Enum object, and instantiating enums is not inexpensive.

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
            let keys = Object.keys(values);
            let index = 0;
            while (index < keys.length) {
              yield keys[index];
              index++;
            }
          }
        },
        keys: {
          get: function() {
            return function () { return Object.keys(this); };
          },
        },
        values: {
          get: function() {
            return function () { return Object.values(this); };
          },
        },
        entries: {
          get: function() {
            return function () { return Object.entries(this); };
          },
        },
        has: {
          get: function() {
            return function (elem) { return Object.values(this).includes(elem); };
          },
        },
        size: {
          get: function() {
            return function() { return this[Symbol.enumSize]; };
          }
        },
        forEach: {
          get: function() {
            return function(callback) {
              for (const member in Object.keys(this)) {
                callback(member, this[member], this);
              }
            }
          }
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
    ```

    Which means that enums are iterable:

    ```js
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

    And have many familiar methods, similar to maps:

    ```js
    DaysOfWeek.size();

    // 7

    DaysOfWeek.keys();

    // [ SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY ]

    DaysOfWeek.values();

    // [ Symbol(SUNDAY), Symbol(MONDAY), Symbol(TUESDAY), Symbol(WEDNESDAY), Symbol(THURSDAY), Symbol(FRIDAY), Symbol(SATURDAY) ]

    DaysOfWeek.entries();

    /*
      [
        [ SUNDAY, Symbol(SUNDAY) ],
        [ MONDAY, Symbol(MONDAY) ],
        [ TUESDAY, Symbol(TUESDAY) ],
        [ WEDNESDAY, Symbol(WEDNESDAY) ],
        [ THURSDAY, Symbol(THURSDAY) ],
        [ FRIDAY, Symbol(FRIDAY) ],
        [ SATURDAY, Symbol(SATURDAY) ]
      ]
    */

    const day = DaysOfWeek.SUNDAY;
    DaysOfWeek.has(day);
    DaysOfWeek.has('SUNDAY')

    // true
    // false

    DaysOfWeek.forEach(day => console.log(day.toString()));

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
    BAR = 'BAR',
    BAZ,
  }
  ```

  Is equivalent to:

  ```js
  const MetasyntacticVariables = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      BAR: 'BAR',
      BAZ: Symbol('BAZ'),
      // Other methods as defined above.
    })
  );
  ```

  _EnumDeclaration_ (with _BindingIdentifier_) or _EnumExpression_ example:

  ```js
  enum MetasyntacticVariables {
    FOO,
    BAR = 'BAR',
    BAZ,
  };
  ```

  Is approximately equivalent to:

  ```js
  const MetasyntacticVariables = Object.freeze(
    Object.create(null, {
      FOO: Symbol('FOO'),
      BAR: 'BAR',
      BAZ: Symbol('BAZ'),
      // Other methods as defined above
    })
  );
  ```

  This means that the value of an `enum` entry can be whatever you want it to be:

  ```js
  enum ManyTypedValues {
    STRING = 'string',
    NUMBER = 42,
    SYMBOL = Symbol('Symbol'),
    ARRAY = ['array', 'of', 'values'],
    OBJECT = { value: 42 },
    FUNCTION = () => 'function'
  }
  ```

  This also means that values of an `enum` entry can carry data that is relevant to an entry:

  ```js
  enum Colors {
    RED: { red: 255, blue: 0, green: 0 },
    BLUE: { red: 0, blue: 255, green: 0 },
    GREEN: { red: 0, blue: 0, green: 255 }
  };
  ```

  Or even behavior:

  ```js
  const toCSS = () => ({ color: `rgb(${this.red}, ${this.green}, ${this.blue})` });
  enum Colors {
    RED: { red: 255, blue: 0, green: 0, toCSS },
    BLUE: { red: 0, blue: 255, green: 0, toCSS },
    GREEN: { red: 0, blue: 0, green: 255, toCSS }
  };
  export default () => <div style={Colors.RED.toCSS()}>I have color!</div>;
  ```

  Or even be nested:

  ```js
  const toCSS = () => ({ color: `rgb(${this.red}, ${this.green}, ${this.blue})` });
  enum COLORS {
    RED = enum {
      LIGHT = { red: 125, blue: 0, green: 0, toCSS },
      DARK = { red: 255, blue: 0, green: 0, toCSS }
    },
    BLUE = enum {
      LIGHT = { red: 0, blue: 125, green: 0, toCSS },
      DARK = { red: 0, blue: 255, green: 0, toCSS }
    },
    GREEN = enum {
      LIGHT = { red: 0, blue: 0, green: 125, toCSS },
      DARK = { red: 0, blue: 0, green: 255, toCSS }
    }
  }

- _EnumDeclaration_ does not require an identifier:
  - This means you can bind an enum to a variable
    ```js
    const MetasyntacticVariables = enum {
      FOO,
      BAR,
      BAZ
    };
    ```

  - This also means you can return them from a function
    ```js
    const MetasyntacticVariables = (() => {
      return enum { FOO, BAR, BAZ };
    })();
    ```

- _EnumDeclaration_ may not have duplicate _EnumEntryName_:
  - This throws an exception:
    ```js
    enum Foos {
      FOO,
      FOO
    }
    ```

\* Approximately means that it may not fully represent all of the semantics.
