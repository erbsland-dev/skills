# Trivial Function Test Suite

When testing a trivial function, a minimal test suite is enough.

`HashHelperTest.cpp`:
```cpp
// copyright header


// includes and definition for tested target

#include <erbsland/unittest/UnitTest.hpp>


TESTED_TARGETS(HashHelper combineHash advanceHash) TAGS(Utilities)
class HashHelperTest : public el::UnitTest {
public:
    void testCombineHash() {
        const std::size_t h1 = 12345;
        const std::size_t h2 = 67890;
        const std::size_t combined = combineHash(h1, h2);

        // Basic property: combined hash should be different from inputs
        REQUIRE_NOT_EQUAL(combined, h1);
        REQUIRE_NOT_EQUAL(combined, h2);

        // Combining same hashes should be deterministic
        REQUIRE_EQUAL(combined, combineHash(h1, h2));

        // Order matters in hash combination
        REQUIRE_NOT_EQUAL(combineHash(h1, h2), combineHash(h2, h1));
    }

    void testAdvanceHash() {
        std::size_t hash = 0;
        const int value = 42;
        advanceHash(hash, value);

        REQUIRE_NOT_EQUAL(hash, static_cast<std::size_t>(0));
        REQUIRE_EQUAL(hash, combineHash(0, std::hash<int>{}(value)));

        const std::size_t prev_hash = hash;
        const std::string text = "hello";
        advanceHash(hash, text);

        REQUIRE_NOT_EQUAL(hash, prev_hash);
        REQUIRE_EQUAL(hash, combineHash(prev_hash, std::hash<std::string>{}(text)));
    }

    // ... more test cases ...
};

```

- This test suite directly derives from `el::UnitTest`.
- It defines individual test cases for `combineHash` and `advanceHash`.
- It makes use of the comparison methods `REQUIRE_EQUAL` and `REQUIRE_NOT_EQUAL`, that will print the
  tested values if the assertion fails.
