# Locate Core outside the usual submodule path

Load this only when Core is unavailable at `<project root>/erbsland/core`.

1. Check the consuming project's instructions and dependency declarations, including `.gitmodules`, for another
   Core location or an uninitialized submodule.
2. Search project CMake files for `erbsland::core`. Follow the relevant `add_subdirectory`, dependency-fetching, or
   package configuration to the checkout or installed package. Linkage can be `PRIVATE`, `PUBLIC`, or `INTERFACE`;
   do not match only one complete `target_link_libraries` spelling.
3. If CMake does not reveal the location, inspect an existing compile command or CMake cache for the consumer target's
   include/package paths. Identify the dependency supplying `erbsland` headers; similarly named standalone libraries
   do not establish that Core is available.
4. Record the resolved location and reuse it for the task. With an installed package, inspect public headers and use
   indexed documentation for sources/topics that are not installed.

Keep detection local to the project and its known dependency paths. Do not scan unrelated checkouts or install/fetch
Core merely to complete discovery. If the dependency is unavailable, report the concrete missing dependency and
continue work that does not require its API.
