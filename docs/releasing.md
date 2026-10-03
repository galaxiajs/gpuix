# Releasing GPU-IX packages

GPU-IX releases three packages together as tarballs attached to a GitHub release. The `@gpuix/native` and `@gpuix/react` packages on npm belong to upstream. Do not run `npm publish` or `bun publish` for this fork.

## Prepare the stamp commit

1. Fetch `origin` and work from its latest `main`. Check that the changes intended for the release have landed, and read their `.changeset/` files. Confirm that no unrelated work is in the tree.
2. Choose one `<version>` with the `-fork` suffix. Set that version in `packages/native/package.json`, `packages/react/package.json` and `packages/plugins/package.json`. Set React's `@gpuix/native` dependency and plugins' `@gpuix/react` peer dependency to the same version. Update the lockfile if the package edits change it.
3. Update the release example in `README.md` and write the release entry in `CHANGELOG.md` from the changesets and merged changes. Do not edit generated files under `packages/native/dist`.
4. Run the relevant build, tests and typecheck from `AGENTS.md`. Review the version and dependency pins, then commit the version, README and changelog changes as `Stamp <version>`. Push the stamp commit to `main` and wait for its push CI run to pass.

## Pack and publish the release assets

Run `bun install --frozen-lockfile` in the stamp checkout, then pack each package there. Each package's `prepack` script builds it.

```bash
(cd packages/native && bun pm pack)
(cd packages/react && bun pm pack)
(cd packages/plugins && bun pm pack)
```

Inspect all three tarballs before publishing. Check their package versions and dependency pins, and check that the native tarball contains its loader, declarations and addon under `dist/`. The package `files` lists and `prepack` scripts in each `package.json` define what belongs in the tarballs.

Write the release notes from the changelog entry, including any breaking change or known limitation. Replace `<version>` and `<notes-file>` below after reviewing the stamp commit and tarballs. The three tags must point to that commit.

```bash
git tag '@gpuix/native@<version>' HEAD
git tag '@gpuix/react@<version>' HEAD
git tag '@gpuix/plugins@<version>' HEAD
git push origin '@gpuix/native@<version>' '@gpuix/react@<version>' '@gpuix/plugins@<version>'
gh release create '@gpuix/react@<version>' -R galaxiajs/gpuix --title '@gpuix/react@<version>' --latest --notes-file '<notes-file>' 'packages/native/gpuix-native-<version>.tgz' 'packages/react/gpuix-react-<version>.tgz' 'packages/plugins/gpuix-plugins-<version>.tgz'
```

After publishing, install the three release URLs into a fresh Bun project using the pattern in the README Quickstart. Check that React and plugins import, the native addon resolves from `dist/`, and the versioned tarball URLs all point to the same release. The standalone chat binaries are separate: `.github/workflows/ci.yml` builds them on `workflow_dispatch`, and they are attached to a release by hand when needed.
