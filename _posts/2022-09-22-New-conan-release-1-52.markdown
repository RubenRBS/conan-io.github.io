---
layout: post
comments: false
title: "Conan 1.52: Build system improvements: MSBuild, CMakeToolchain, continue working in easing migration to 2.0, new export_conandata_patches tool, new build folder argument for cmake_layout."
meta_title: "Version 1.52 of Conan C++ Package Manager is Released" 
meta_description: "Conan 1.52: Build system improvements: MSBuild, CMakeToolchain, continue working in easing migration to 2.0, new export_conandata_patches tool and much more"
---

<script type="application/ld+json">
{ "@context": "https://schema.org", 
 "@type": "TechArticle",
 "headline": "Version 1.52 of Conan C++ Package Manager is Released",
 "alternativeHeadline": "Learn all about the new 1.52 Conan C/C++ package manager version",
 "image": "https://docs.conan.io/en/latest/_images/frogarian.png",
 "author": "Conan Team", 
 "genre": "C/C++", 
 "keywords": "c c++ package manager conan release", 
 "publisher": {
    "@type": "Organization",
    "name": "Conan.io",
    "logo": {
      "@type": "ImageObject",
      "url": "https://media.jfrog.com/wp-content/uploads/2017/07/20134853/conan-logo-text.svg"
    }
},
 "datePublished": "2022-09-22",
 "description": "Build system improvements: MSBuild, CMakeToolchain, continue working in easing migration to 2.0, new export_conandata_patches tool, new build folder argument for cmake_layout.",
 }
</script>

We are pleased to announce that Conan 1.52 has been released and brings some significant
new features and bug fixes. First, this release comes with some improvements in the
[MSBuild](https://docs.conan.io/en/latest/reference/conanfile/tools/microsoft.html#msbuild)
and
[CMakeToolchain](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake/cmaketoolchain.html)
tools. Also, we continue the work of easing the migration to Conan 2.0 with features like
supporting installing *remotes.json* in 1.X or making the
[to_apple_arch](https://docs.conan.io/en/latest/reference/conanfile/tools/apple.html#to-apple-arch)
tool public. We added a new
[export_conandata_patches](https://docs.conan.io/en/latest/reference/conanfile/tools/files/patches.html#conan-tools-files-export-conandata-patches)
tool and a new argument for
[cmake_layout](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake/cmake_layout.html)
to define the build folder.

Also, it's worth noting that [Conan
2.0-beta3](https://github.com/conan-io/conan/releases/tag/2.0.0-beta3) was released this
month with several new features and fixes.

## Improvements in build-systems tools

### MSBuild

Support ``targets`` argument in
[MSBuild.build()](https://docs.conan.io/en/latest/reference/conanfile/tools/microsoft.html#msbuild)
method. With this argument, you can pass the specific target you want to build. You can use
it in your recipes like:

```python
...
class MylibConan(ConanFile):
    ...
    def build(self):
        msbuild = MSBuild(self)
        msbuild.build("MyProject.sln", targets=["mytarget"])
```

And the MSBuild *build()* method will internally add the ``/target=mytarget`` argument to
the call.

### CMakeToolchain

Added support for
[BUILD_TESTING](https://cmake.org/cmake/help/latest/command/enable_testing.html) CMake
variable in
[CMakeToolchain](https://docs.conan.io/en/latest/reference/conanfile/tools/cmake/cmaketoolchain.html).
This variable is added to the *CMakePresets.json* file generated by the toolchain and it
will set the variable to ``OFF``, when the ``tools.build:skip_test`` configuration is
true.

## Continue working on easing migration to Conan 2.0

As a continuation of the effort to make migration to Conan 2.0 easier, we added some
features to Conan 1.52 like:

- Support for *remotes.json* in ``conan config install`` command. This file, that replaces
  the 1.X *remotes.txt* can be installed in the local cache with the ``conan config
  install`` command. Please, note that **only one of remotes.json or remotes.txt** should
  be  installed.

- Support for traits in 1.X ``self.requires()``. One of the new features in Conan 2.0 is
  the support for
  [traits](https://github.com/conan-io/tribe/blob/main/design/026-requirements_traits.md)
  to enhance the Conan dependency model. As there are recipes that will take advantage of
  the traits model we have enabled the possibility to set those traits arguments in Conan
  1.X as well. Be aware that these arguments will not have any effect in Conan 1.X but
  will not make the recipe throw an error enabling the migration.

- Add
  [conan.tools.apple.to_apple_arch](https://docs.conan.io/en/latest/reference/conanfile/tools/apple.html#to-apple-arch)
  tool. This tool is useful to convert between Conan-style arch settings and the format
  understood by the Apple build tools (*x86* to *i386*, *armv8* to *arm64*, etc.).

-  Added ability to pass additional arguments to
   [conan.tools.scm.Git.clone()](https://docs.conan.io/en/latest/reference/conanfile/tools/scm/git.html#clone).
   Using that argument you can add extra arguments as a list to the ``git clone`` call:

```python
  from conan import ConanFile
  from conan.tools.scm import Git

  class App(ConanFile):
      version = "1.2.3"

      def source(self):
          git = Git(self)
          clone_args = ['--depth', '1', '--branch', self.version]
          git.clone(url="https://path/to/repo.git", args=clone_args)
```

Please, do not forget to check our [migration to Conan 2.0
guide](https://docs.conan.io/en/latest/conan_v2.html) in the Conan documentation.

## New export_conandata_patches tool

Very similar to the
[apply_conandata_patches()](https://docs.conan.io/en/latest/reference/conanfile/tools/files/patches.html#conan-tools-files-apply-conandata-patches)
tool, we added a [new
export_conandata_patches](https://docs.conan.io/en/latest/reference/conanfile/tools/files/patches.html#conan-tools-files-export-conandata-patches)
tool to exports patches declared in the *conandata.yml* file. Use it like:

```python
from conan import ConanFile
from conan.tools.files import export_conandata_patches

class MyLibrary(ConanFile):
  ...
  def export_sources(self):
      export_conandata_patches(self)
```

It will copy all the patches defined in the ``conanfile.conan_data`` from the
``conanfile.recipe_folder`` to the ``conanfile.exports_sources_folder``.

## New build folder argument for cmake_layout

When declaring the ``cmake_layout`` in recipes, it will set the value for
``conanfile.folders.build`` to *build* if the cmake generator is multi-configuration or
*build/Debug* or *build/Release* if the cmake generator is single-configuration, depending
on the build_type. Now, using the ``build_folder`` argument you can redefine the value of
the *build* string:

```python
from conan import ConanFile
from conan.tools.cmake import cmake_layout

class MyLibrary(ConanFile):
  ...
  def layout(self):
      cmake_layout(self, build_folder="mybuildfolder")
```

That will result setting the ``conanfile.folders.build`` to *mybuildfolder* for
multi-configuration and *mybuildfolder/Debug* or *mybuildfolder/Release* for
single-configuration.

## Conan 2.0-beta3 released

Conan 2.0 beta3 [is already
out](https://github.com/conan-io/conan/releases/tag/2.0.0-beta3). You can install it using
*pip*:

```bash
$ pip install conan --pre
```

Don't forget to check the [documentation for Conan 2.0](https://docs.conan.io/en/2.0/).

---

<br>

Besides the items listed above, there were some minor bug fixes you may wish to read
about. If so please refer to the
[changelog](https://docs.conan.io/en/latest/changelog.html#aug-2022) for the complete
list.

We hope you enjoy this release and look forward to [your
feedback](https://github.com/conan-io/conan/issues).
