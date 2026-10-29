The Hermes sandbox cgroup caps this container at 2 GiB memory. The Next
production build (Turbopack compile of the full LearnHarness app) was
OOM-killed twice: once at default heap (~10 GB in use by other tenants,
killed), once with --max-old-space-size=1200. The kill happens at
"Creating an optimized production build" - the memory ceiling is the
sandbox's, not Node's.

On a normal machine (the user's Manjaro PC), `npm run build` will run
without this constraint. The PKGBUILD's build() does not need a
NODE_OPTIONS cap; if the user's machine has <4 GB RAM they can add one.
