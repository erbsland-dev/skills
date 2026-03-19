# Test a Target with Many Values

As soon as you test the same target with more than five values, use one of the following approaches:

## Use a Helper Method and Call it Using `WITH_CONTEXT` 

The test data in the following example was redacted `<...>`. 

`DecoderHashTest.cpp`:
```cpp
// copyright

// target includes

#include <erbsland/unittest/UnitTest.hpp>

TESTED_TARGETS(Decoder ShaHash)
class DecoderHashTest final : public el::UnitTest {
public:
    CharStreamPtr decoder;
    DecodedChar decodedChar;

    void verifyHash(const String &content, const Bytes &expectedDigest) {
        auto source = createTestMemorySource(content);
        REQUIRE_NOTHROW(source->open());
        decoder = CharStream::create(source);
        REQUIRE(decoder != nullptr);
        decodedChar = decoder->next();
        while (decodedChar != Char::EndOfData) {
            decodedChar = decoder->next();
        }
        REQUIRE_EQUAL(decoder->digest(), expectedDigest);
    }

    void testNoHash() {
        WITH_CONTEXT(verifyHash("<testdata1>\n", Bytes{}));
    }

    void testWithHash() {
        REQUIRE_EQUAL(impl::defaults::documentHashAlgorithm, impl::crypto::ShaHash::Algorithm::Sha3_256);
        WITH_CONTEXT(verifyHash("<testdata2>\n", Bytes::fromHex("<result2>")));
        WITH_CONTEXT(verifyHash("<testdata3>\n", Bytes::fromHex("<result3>")));
        WITH_CONTEXT(verifyHash("<testdata4>\n", Bytes::fromHex("<result4>")));
        WITH_CONTEXT(verifyHash("<testdata5>\n", Bytes{}));
        WITH_CONTEXT(verifyHash("<testdata6>\n", Bytes::fromHex("<result5>")));
    }
};

```

- Using `verifyHash()` helper, avoiding repetition and encapsulating a test sequence.
- Using `WITH_CONTEXT()` to get a call stack, pointing to the actual location in `test...()` where the test failed.
- Using member variables to keep the last test state on failure (for debugging).

## Use a Vector with Test Data

If you have more than five similar test cases, consider using a vector with test data.

`U8DecoderTest.cpp`
```cpp
// copyright

// target includes

#include <erbsland/unittest/UnitTest.hpp>

#include <vector>


TESTED_TARGETS(U8Decoder)
class U8DecoderTest final : public el::UnitTest {
public:
    // ...
    void testInvalidUtf8Sequences() {
        const auto invalidSequences = std::vector<std::pair<Bytes, std::string_view>>{
            {Bytes::fromHex("C0 80"), "Overlong two-byte sequence (U+0000 encoded in two bytes)"},
            {Bytes::fromHex("C1 80"), "Overlong two-byte sequence (U+0001 encoded in two bytes)"},
            {Bytes::fromHex("C2"), "Truncated two-byte sequence"},
            {Bytes::fromHex("C2 41"), "Invalid continuation byte in two-byte sequence"},
            {Bytes::fromHex("E2 82"), "Truncated three-byte sequence"},
            {Bytes::fromHex("E2 41 80"), "Invalid continuation byte in three-byte sequence"},
            {Bytes::fromHex("E0 80 80"), "Overlong three-byte sequence (U+0000 encoded in three bytes)"},
            {Bytes::fromHex("E0 9F BF"), "Overlong three-byte sequence (U+07FF encoded in three bytes)"},
            {Bytes::fromHex("ED A0 80"), "UTF-16 surrogate half U+D800 encoded"},
            {Bytes::fromHex("F0 41 80 80"), "Invalid continuation byte in four-byte sequence"},
            {Bytes::fromHex("F0 9F BF"), "Truncated four-byte sequence"},
            {Bytes::fromHex("F0 80 B0 B0"), "Overlong four-byte sequence (U+0FFF encoded in four bytes)"},
            {Bytes::fromHex("F0 8F BF BF"), "Overlong four-byte sequence (U+03FFF encoded in four bytes)"},
            {Bytes::fromHex("F4 90 80 80"), "Codepoint above U+10FFFF"},
            {Bytes::fromHex("F8 80 80 80 80"), "Invalid start byte (five-byte sequence)"},
            {Bytes::fromHex("FF"), "Invalid start byte (0xFF)"},
        };
        for (const auto &[bytes, description] : invalidSequences) {
            std::size_t pos = 0;
            runWithContext(SOURCE_LOCATION(), [&] {
                try {
                    static_cast<void>(U8Decoder<const std::byte>::decodeChar(bytes, pos));
                    REQUIRE(false);
                } catch (const Error &e) {
                    REQUIRE(e.category() == ErrorCategory::Encoding);
                }
                REQUIRE(pos == 0);
            }, [&]() -> std::string {
                return std::format("Decoded invalid sequence: {}", description);
            });
        }
    }
    // ...
};
```

For complex test scenarios, use a `struct`:

```cpp
struct TestCase {
    int a;
    int b;
    int c;
    int expected;
};
const auto testCases = std::vector<TestCase>{
    // ...
};
for (const auto &[a, b, c, expected] : testCases) {
    // ...
}
```

- Keeps the test code minimal and readable.
- Avoids repetition.
- Provides good context in case of failure.
