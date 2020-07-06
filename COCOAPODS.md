# CocoaPods integration

Kotlin/Native provides integration with the [CocoaPods dependency manager](https://cocoapods.org/). You can add
dependencies on Pod libraries stored in the CocoaPods repository or locally as well as use a Kotlin/Native Gradle
project as a CocoaPods dependency.

You can manage Pod dependencies right in Intellij IDEA enjoying its benefits of working with code, such as code
 highlighting and completion. You can build the whole Kotlin project with Gradle as a whole not switching to Xcode. 
 Go to Xcode only when you need to write Swift/Objective-C code or run your application on a simulator or device.
 
Depending on your purpose, you can add dependencies between:
* [A Kotlin project and Pod libraries from the CocoaPods repository](#add-a-dependency-between-a-kotlin-project-and-pod-libraries-from-the-cocoapods-repository)
* [A Kotlin project and Objective-C Pod libraries stored locally](#add-a-dependency-between-a-kotlin-project-and-objective-c-pod-libraries-stored-locally)
* [A Kotlin Pod (Kotlin project used as a CocoaPods dependency) and an Xcode project with one target](#add-a-dependency-between-a-kotlin-pod-and-xcode-project-with-one-target) 
or [several targets](#add-a-dependency-between-a-kotlin-pod-with-an-xcode-project-with-several-targets)
* [Several Kotlin Pods and an Xcode project](l#add-dependencies-between-multiple-kotlin-pods-and-an-xcode-project)

[Complete initial configuration](#install-the-cocoapods-dependency-manager-and-plugin), and when you add a new dependency 
to `cocoapods`, just re-import the project in IntelliJ IDEA. The new dependency will be added automatically. No additional 
steps are required.

## Install the CocoaPods dependency manager and plugin

1. Install the [CocoaPods dependency manager](https://cocoapods.org/).
    
    <div class="sample" markdown="1" theme="idea" mode="ruby" data-highlight-only>
    
    ```ruby
    $ sudo gem install cocoapods
    ```
    
    </div>

2. Install the `cocoapods-generate` plugin.

    <div class="sample" markdown="1" theme="idea" mode="ruby" data-highlight-only>
    
    ```ruby
    $ sudo gem install cocoapods-generate
    ```
    
    </div>
    
3. In `build.gradle.kts` (or `build.gradle`) of your IDEA project, apply the CocoaPods plugin `kotlin("native
.cocoapods")`.  
> The CocoaPods plugin is based on the multiplatform project model and requires applying the
`kotlin("multiplatform")` plugin. See details about the 
[multiplatform plugin](https://kotlinlang.org/docs/reference/building-mpp-with-gradle.html).
{:.note}

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
plugins {
   kotlin("multiplatform") version "{{ site.data.releases.latest.version }}"
   kotlin("native.cocoapods") version "{{ site.data.releases.latest.version }}"
}
```

</div> 

When applied, the CocoaPods plugin does the following:

* Adds both `debug` and `release` frameworks as output binaries for all iOS and macOS targets.
* Creates a `podspec` task which generates a [Podspec](https://guides.cocoapods.org/syntax/podspec.html)
file for the project.

The `Podspec` file includes a path to an output framework and script phases which automate building
this framework during a build process of an Xcode project. 

`version` is a version of the Gradle project. You can configure `summary`, `homepage`, and `frameworkName`of the
 `Podspec` file in the `cocoapods` block. 

<div class="sample" markdown="1" theme="idea" data-highlight-only>

```kotlin
plugins {
    kotlin("multiplatform") version "{{ site.data.releases.latest.version }}"
    kotlin("native.cocoapods") version "{{ site.data.releases.latest.version }}"
}

// CocoaPods requires the podspec to have a version.
version = "1.0"

kotlin {
    cocoapods {
        // Configure fields required by CocoaPods.
        summary = "Some description for a Kotlin/Native module"
        homepage = "Link to a Kotlin/Native module homepage"

        // You can change the name of the produced framework.
        // By default, it is the name of the Gradle project.
        frameworkName = "my_framework"
    }
}
```

</div>

## Add dependencies on Pod libraries

You can add dependencies on Pod libraries [stored in the CocoaPods repository](#add-a-dependency-between-a-kotlin-project-and-pod-libraries-from-the-cocoapods-repository) 
and [stored locally](#add-a-dependency-between-a-kotlin-project-and-objective-c-pod-libraries-stored-locally).

### Add a dependency between a Kotlin project and Pod libraries from the CocoaPods repository

1. Add dependencies to Pod libraries that you want to use from the CocoaPods repository in your project with `pod()`.

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
    
    ```kotlin
    kotlin {
        ios()
    
        cocoapods {
            summary = "CocoaPods test library"
            homepage = "https://github.com/JetBrains/kotlin"
            pod("AFNetworking", "~> 3.2.1")
        }
    }
    ```
    
    </div>

2. Re-import the project.

To use these dependencies from Kotlin code, import a package `cocoapods.<library-name>`.
In the example above, it's `cocoapods.AFNetworking`.

### Add a dependency between a Kotlin project and Objective-C Pod libraries stored locally

1. To the `cocoapods` block of `build.gradle.kts` (`build.gradle`), add dependencies to Pod libraries stored locally with
 `pod()`.  
As the third argument, specify the path to `Podspec` of the local Pod using `projectDir.resolve("..")`.  
> You can also add dependencies as subspecs.  
The `cocoapods` element can include dependencies to Pods stored locally and Pods from the CocoaPods repository at the same time.  
{:.note}

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
    
    ```kotlin
    kotlin {
        ios()
    
        cocoapods {
            summary = "CocoaPods test library"
            homepage = "https://github.com/JetBrains/kotlin"
            pod("pod_dependency", "1.0", projectDir.resolve("../pod_dependency/pod_dependency.podspec"))
            pod("subspec_dependency/Core", "1.0", projectDir.resolve("../subspec_dependency/subspec_dependency.podspec"))
            pod("AFNetworking", "~> 3.2.1")
        }
    }
    ```
    
    </div>

2. Re-import the project.

You can find a sample project [here](https://github.com/zoldater/KotlinWithLocalObjCPods). 
 
## Use a Kotlin Gradle project as a CocoaPods dependency

You can use a Kotlin/Native Gradle project as a CocoaPods dependency (Kotlin Pod). Such a representation provides the
 following advantages:

 * Such a dependency can be included in a `Podfile` of an Xcode project and automatically built (and rebuilt)
 along with this project. It simplifies importing to Xcode by removing a need to write corresponding Gradle tasks and
  Xcode build steps manually.
 * When building from Xcode, you can use CocoaPods libraries without writing
 `.def` files manually and setting `cinterop` tool parameters. In this case, all required parameters can be
 obtained from the Xcode project configured by CocoaPods.
 
You can add dependencies between:
* [A Kotlin Pod and an Xcode project with one target](#add-a-dependency-between-a-kotlin-pod-and-xcode-project-with-one-target)
* [A Kotlin Pod and an Xcode project with several targets](#add-a-dependency-between-a-kotlin-pod-with-an-xcode-project-with-several-targets)
* [Several Kotlin Pods and an Xcode project](l#add-dependencies-between-multiple-kotlin-pods-and-an-xcode-project)

> To correctly import the dependencies into the Kotlin/Native module, the
`Podfile` must contain either [`use_modular_headers!`](https://guides.cocoapods.org/syntax/podfile.html#use_modular_headers_bang)
or [`use_frameworks!`](https://guides.cocoapods.org/syntax/podfile.html#use_frameworks_bang)
directive.
{:.note}

### Add a dependency between a Kotlin Pod and Xcode project with one target

1. Create an Xcode project with `Podfile` if you haven’t done this yet.
2. Add the path to your `Podfile` with `podfile = projectDir.resolve("..")` to `build.gradle.kts` (`build.gradle`) 
of your Kotlin project.

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
        
    ```kotlin
    kotlin {
        ios()
    
        cocoapods {
            summary = "CocoaPods test library"
            homepage = "https://github.com/JetBrains/kotlin"
            pod("AFNetworking", "~> 3.2.1")
            podfile = projectDir.resolve("../ios-app/Podfile")
        }
    }
    ```
    
    </div>

3. Add the name of the Kotlin Pod you want to include in the Xcode project and its path to `Podfile`.

    <div class="sample" markdown="1" theme="idea" mode="ruby" data-highlight-only>
    
    ```ruby
    use_frameworks!
    
    platform :ios, '9.0'
    
    target 'ios-app' do
            pod 'kotlin_library', :path => '../kotlin-library'
    end
    ```
    
    </div>

4. Re-import the project.

### Add a dependency between a Kotlin Pod with an Xcode project with several targets

1. Create an Xcode project with `Podfile` if you haven’t done this yet.
2. Add the path to your `Podfile` with `podfile = projectDir.resolve("..")` to` build.gradle.kts` (`build.gradle`) of
 your Kotlin project.
3. Add dependencies to Pod libraries that you want to use in your project with `pod().`
4. For each target, specify the minimum target version for the Pod library.

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
    
    ```kotlin
    kotlin {
        ios()
        tvos()
    
        cocoapods {
            summary = "CocoaPods test library"
            homepage = "https://github.com/JetBrains/kotlin"
            ios.deploymentTarget = "13.5"
            tvos.deploymentTarget = "13.4"
       
            pod("AFNetworking", "~> 4.0.0")
            podfile = projectDir.resolve("../severalTargetsXcodeProject/Podfile") // specify the path to Podfile
        }
    }
    ```
    
    </div>

2. Add the name of the Kotlin Pod you want to include in the Xcode project and its path to `Podfile`.

    <div class="sample" markdown="1" theme="idea" mode="ruby" data-highlight-only>
    
    ```ruby
    target 'iosApp' do
      use_frameworks!
      platform :ios, '13.5'
      # Pods for iosApp
      pod 'kotlin_library', :path => '../kotlin-library'
    end
    
    target 'TVosApp' do
      use_frameworks!
      platform :tvos, '13.4'
      
      # Pods for TVosApp
      pod 'kotlin_library', :path => '../kotlin-library'
    end
    ```
    
    </div>
    
3. Re-import the project.

You can find a sample project [here](https://github.com/zoldater/severalXcodeTargetsDemo).

### Add dependencies between multiple Kotlin Pods and an Xcode project

1. Create an Xcode project with `Podfile` if you haven’t done this yet.
2. Create a hierarchical structure of your Kotlin projects - a root and child projects.
3. Add the path to your `Podfile` with `podfile = projectDir.resolve("..")` to `build.gradle.kts` (`build.gradle)` of
 the root project.
4. Specify that you don’t need a `Podspec` for this root project - `noPodspec()`

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
    
    ```kotlin
    kotlin {
        ios()
        cocoapods {
            noPodspec()
            podfile = projectDir.resolve("ios-app/Podfile")
        }
    }
    ```
    
    </div>

5. Add dependencies to Pod libraries that you want to use with `pod()` to the build script of your child projects`.  
`Do NOT add a path to `Podspec`.

    <div class="sample" markdown="1" theme="idea" data-highlight-only>
    
    ```kotlin
    kotlin {
        ios()
        cocoapods {
            summary = "Working with external pods from Kotlin/Native using CocoaPods"
            homepage = "https://github.com/JetBrains/kotlin-native"
            pod("AFNetworking", "~> 3.2.1")
        }
    }
    ```
    
    </div>


6. Add names of the Kotlin Pods you want to include in the Xcode project and their paths to `Podfile`.

    <div class="sample" markdown="1" theme="idea" mode="ruby" data-highlight-only>
    
    ```ruby
    use_frameworks!
   
    platform :ios, '9.0'
    
    target 'ios-app' do
       pod 'firstKotlinPod', :path => '../firstKotlinPod'
       pod 'secondKotlinPod', :path => '../secondKotlinPod'
    end
    ```
    
    </div>

7. Re-import the project.

You can find a sample project [here](https://github.com/zoldater/cocoapodsMPPDemo).


