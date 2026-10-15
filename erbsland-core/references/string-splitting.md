# Split text according to how the parts are used

Read this when collecting split fields, processing them sequentially, or tracking field positions.
Paths below are relative to the Core dependency.

## Collect a list directly

When the result should be a `StringList`, call `StringList::fromSplit(...)`. Do not construct a `StringSplitter`
just to loop over it and append every part to a new list. `fromSplit` owns that collection logic, including capacity
estimation, a split limit, and the empty-field policy. It currently uses a splitter internally; the advantage is the
complete collection operation, not a promise of a universally faster splitting algorithm.

Choose `keepEmpty` deliberately: the default discards empty fields. Preserve them when their positions matter.
For example, with `<erbsland/String.hpp>`, `<erbsland/StringList.hpp>`, `<erbsland/CharSet.hpp>`, and
`<erbsland/ItemCount.hpp>`, in a function body with the literal namespace in scope:

```cpp
const auto text = el::String{"alpha..omega"_el};
const auto reversed = el::StringList::fromSplit(text, el::CharSet{U'.'}, el::ItemCount::infinite(), true)
    .reversed()
    .join("."_el);
```

The result is `omega..alpha`. Splitting produces shared slices; reversing reorders handles, and joining sizes the
final character storage once. The list itself still needs storage. An empty join separator concatenates fragments.
For a fixed number of fragments, prefer `String::fromJoined({...})`; see [formatting and joining](string-formatting.md).

A `CharSet` denotes alternative separator characters, not a multi-character delimiter, CSV quoting, or an escape-aware
grammar. Use the appropriate parser when separators can occur inside quoted or escaped fields.

## Process sequentially without collecting everything

Use `StringSplitter` when consuming parts incrementally, stopping early, retaining separators, or working with the
unread suffix. Its default mode discards separators but preserves empty parts, unlike `StringList::fromSplit()`.
Test `isAtEnd()` for termination: an empty result from `next()` can be a real field.

If the callback needs a field index, use `forEach(const String &, ItemIndex)` instead of maintaining a counter.
The index is the zero-based source part index, continues after earlier `next()`/`skip()` calls, and includes empty
parts. Filtering fields inside the callback does not renumber it.

With `<erbsland/StringSplitter.hpp>`, `<erbsland/ItemIndex.hpp>`, `<erbsland/LoopStatus.hpp>`,
`<erbsland/LoopResult.hpp>`, and `<erbsland/String.hpp>`:

```cpp
auto splitter = el::StringSplitter{el::String{"alpha;;omega"_el}, U';'};
auto emptyAt = el::ItemIndex::noIndex();
const auto result = splitter.forEach([&emptyAt](const el::String &part, const el::ItemIndex index) -> el::LoopStatus {
    if (part.isEmpty()) {
        emptyAt = index;
        return el::LoopStatus::Stop;
    }
    return el::LoopStatus::Continue;
});
```

Here `emptyAt` is item index 1 and `result` is `LoopResult::Stopped`. Each part is consumed before the callback runs,
so the next call reads `omega`; stopping does not rewind the current part. Return `Error` to report failure or
`Continue` to process the next part. `Success` means all remaining parts were consumed.

Sources: `src/erbsland/text/impl/{StringList,StringSplitter}.{hpp,tpp}`,
`doc/topics/text_strings/slicing_and_splitting_strings.rst`, and `doc/reference/text/matching_and_splitting.rst`.
