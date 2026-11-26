# gradle-osvvm

Packages the complete [OSVVM](https://github.com/OSVVM/OsvvmLibraries)
distribution as one Maven source ZIP, including the utility library,
verification components, Tcl scripts, documentation, and upstream licenses.
The Git submodule at `upstream/osvvm` pins
[OSVVM 2026.09](https://github.com/OSVVM/OsvvmLibraries/tree/ab47a2637008a0709bbf593aa1ab0060dcc39aac).
Its nested submodules pin the individual components.

## Build locally

Initialize all submodules and publish to a file-based Maven repository.

```sh
git submodule update --init --recursive
./gradlew publishOsvvmPublicationToBuildRepository -PpackageVersion=2026.09.0
```

The ZIP is in `build/distributions` and the Maven repository is in
`build/repository`. The Maven example version `2026.09.0` corresponds to the
upstream release `2026.09`.

## Publish a release

Update `upstream/osvvm` to the desired upstream revision, run
`git submodule update --init --recursive`, and commit the updated gitlink.
Publish a GitHub Release from that commit with a tag such as `v2026.09.0`.
The workflow checks out all pinned submodules and publishes
`io.github.vhdlivery.osvvm:osvvm` to GitHub Packages. The Maven version is the
release tag without its leading `v`. The workflow then attaches
`osvvm-<version>-src.zip` and `osvvm-<version>.pom` to the GitHub Release.

## Use with gradle-vhdlArtifacts

Add the Maven repository at
`https://maven.pkg.github.com/vhdlivery/gradle-osvvm` to your Gradle build with
GitHub Packages credentials. Declare the simulation dependency using the
version of the published release.

```groovy
dependencies {
    sim 'io.github.vhdlivery.osvvm:osvvm:2026.09.0'
}
```

Run `./gradlew unzipSimArtifacts`. The plugin selects the `src` ZIP and strips
its outer `osvvm-<version>/` directory, leaving the upstream tree in
`build/dependency/osvvm`. The utility library is in
`build/dependency/osvvm/osvvm` and the simulator setup script is
`build/dependency/osvvm/Scripts/StartUp.tcl`. In the simulator's Tcl shell,
source `StartUp.tcl` and use `build` with the path to `OsvvmLibraries.pro` to
compile the libraries.

Gradle's default file filters omit `.gitattributes` and `.gitignore` when
extracting the ZIP. To extract these upstream files too, add this to your
`settings.gradle`.

```groovy
org.apache.tools.ant.DirectoryScanner.removeDefaultExclude('**/.gitattributes')
org.apache.tools.ant.DirectoryScanner.removeDefaultExclude('**/.gitignore')
```
