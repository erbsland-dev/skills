# Templates

## Small complete class

This header is a complete example without project dependencies. It uses only the sections needed for a small value class.
Replace the copyright placeholders with the project's two-line copyright block.

`Counter.hpp`:

```cpp
// <copyright line 1>
// <copyright line 2>
#pragma once

namespace example::a {

/// Store a non-negative count.
/// @notest{Documentation example only.}
class Counter {
public:

    // defaults
    Counter() = default;

public: // accessors/tests
    /// Test whether the count is zero.
    /// @return True if the count is zero.
    [[nodiscard]] auto isEmpty() const noexcept -> bool { return _value == 0; }
    /// Get the current count.
    [[nodiscard]] auto value() const noexcept -> unsigned int { return _value; }
    /// Set the count.
    void setValue(unsigned int value) noexcept { _value = value; }

public: // operations
    /// Reset the count to zero.
    void reset() noexcept { _value = 0; }

private:
    unsigned int _value{0}; ///< The current count.
};

}
```

## Annotated class layout

This annotated example collects optional class sections to illustrate their placement.
Include only the sections your class needs.
The names and behavior are illustrative; the declarations demonstrate valid C++20 syntax and documentation placement.
For a real class:

- Omit all elements that are not required.
- Prefer placing complex types outside a class in a separate unit and only keep trivial types inline.
- Follow the usual writing style for API documentation and include `@param`, `@return`, and `@throws` where applicable.
- Use `noexcept` only for operations guaranteed not to propagate exceptions, and match declarations and definitions.
- Replace the example test-suite names with actual coverage, or use `@needtest` or `@notest` with a reason.

The project headers below are illustrative.

`ExampleClass.hpp`:

```cpp
// <copyright line 1>
// <copyright line 2>
#pragma once

#include "ExampleSuperClass.hpp"
#include "String.hpp"

#include <compare>

namespace example::a {

/// Store a named value with a description.
/// Demonstrates the placement of optional class sections.
/// @tested{ExampleClassTest}
class ExampleClass : public ExampleSuperClass {
    friend class ExampleFieldClass;

    /// Represent the stored numeric value.
    using Value = int;
    /// Group the object's descriptive labels.
    struct Labels {
        el::String _name; ///< The name.
        el::String _description; ///< The description.
    };

public:
    /// Create a named value.
    /// @param name The object's name.
    /// @param description The object's description.
    /// @param value The initial numeric value.
    ExampleClass(el::String name, el::String description, int value);

    // defaults
    ~ExampleClass() override = default;
    ExampleClass(const ExampleClass&) = default;
    ExampleClass(ExampleClass&&) = default;
    auto operator=(const ExampleClass&) -> ExampleClass& = default;
    auto operator=(ExampleClass&&) -> ExampleClass& = default;

public:
    [[nodiscard]] auto operator<=>(const ExampleClass& other) const noexcept -> std::strong_ordering;
    [[nodiscard]] auto operator==(const ExampleClass& other) const noexcept -> bool;
    // ...

public: // accessors/tests
    /// Test whether the numeric value is zero.
    /// @return True if the numeric value is zero.
    [[nodiscard]] auto isEmpty() const noexcept -> bool;
    /// Get the name.
    [[nodiscard]] auto name() const -> el::String;
    /// Set the name.
    void setName(el::String name);
    /// Get the description.
    [[nodiscard]] auto description() const -> el::String;
    /// Get the numeric value.
    [[nodiscard]] auto value() const noexcept -> int;
    // ...

public: // implement ExampleSuperClass
    [[nodiscard]] auto typeName() const -> el::String override;
    void clear() override;

public: // operations
    /// Increase the stored value.
    /// @param value The positive amount to add.
    /// @return The updated numeric value.
    /// @throws el::ParameterError If the amount is zero or negative, or the result would overflow.
    [[nodiscard]] auto increase(int value) -> int;
    // ...

public: // conversion
    /// Convert this object into a string.
    /// @return A text representation containing the name, description, and numeric value.
    [[nodiscard]] auto toString() const -> el::String;

public: // factory methods
    /// Parse the text representation produced by toString().
    /// @param text The text containing a name, description, and numeric value.
    /// @return A new object with the parsed attributes.
    /// @throws el::ParseError If the text is malformed or the value is outside the supported range.
    [[nodiscard]] static auto fromString(el::String text) -> ExampleClass;

protected: // extension points
    /// Customize how the stored value is increased.
    /// @param value The positive amount to add.
    /// @throws el::ParameterError If the amount is zero or negative, or the result would overflow.
    virtual void increaseImpl(int value);
    // ...

private:
    /// Validate an increase before changing the stored value.
    /// @param value The positive amount to add.
    /// @throws el::ParameterError If the amount is zero or negative, or the result would overflow.
    void validateIncrease(Value value) const;
    /// Notify observers that the numeric value has changed.
    void notifyValueChanged();
    // ...

private:
    el::String _name; ///< The name.
    el::String _description; ///< The description.
    Value _value; ///< The numeric value.
};

}
```

`ExampleClass.cpp`:

```cpp
// <copyright line 1>
// <copyright line 2>
#include "ExampleClass.hpp"

#include <utility>

namespace example::a {

ExampleClass::ExampleClass(el::String name, el::String description, int value)
    : _name(std::move(name)), _description(std::move(description)), _value(value) {
}

auto ExampleClass::isEmpty() const noexcept -> bool {
    return _value == 0;
}

auto ExampleClass::name() const -> el::String {
    return _name;
}

void ExampleClass::setName(el::String name) {
    _name = std::move(name);
}

auto ExampleClass::value() const noexcept -> int {
    return _value;
}

// ...

}
```

## Optional forward declaration

Add a separate forward-declaration header only when code needs one.

`ExampleClass_fwd.hpp`:

```cpp
// <copyright line 1>
// <copyright line 2>
#pragma once

namespace example::a {

class ExampleClass;

}
```
