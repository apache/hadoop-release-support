# Gemini Project Overview: Hadoop Release Support

This document provides a summary of the `hadoop-release-support` project.

## 1. Project Overview

The `hadoop-release-support` project is a specialized toolset designed to assist in the creation, validation, and release of Apache Hadoop release candidates. It provides a collection of scripts and configurations to automate and streamline the complex process of producing a stable Hadoop release. The project uses Apache Ant for orchestration of the release tasks and Apache Maven for dependency management and running validation tests.

## 2. Technologies

*   **Java**: The core language for the validation code.
*   **Apache Ant**: Used to script and orchestrate the entire release workflow, from building and testing to staging and publishing.
*   **Apache Maven**: Used for managing project dependencies, building the Java code, and running tests to validate the classpath and basic functionality.

## 3. Project Structure

The project is organized as follows:

-   `src/`: Contains the source code, release information, and text templates.
    -   `main/java/`: Java source code for compile-time validation of the Hadoop classpath.
    -   `releases/`: Property files for different Hadoop releases, containing version numbers, commit IDs, and other release-specific information.
    -   `test/java/`: Java test source code.
-   `build.xml`: The main Ant build file that orchestrates the entire release process. It contains a large number of targets for various tasks like building, testing, signing, and publishing release artifacts.
-   `pom.xml`: The Maven project file. It defines the project's dependencies on various Hadoop modules and is used to run basic validation tests.
-   `release.properties`: A simple properties file that points to the current Hadoop release being worked on.
-   `README.md`: Provides a detailed explanation of the project, its prerequisites, and the complete workflow for creating and validating a Hadoop release.


The Ant build file also provides targets for more extensive testing, including running tests against downstream projects like Apache Spark and Google Cloud Storage connector.

The maven project is purely used to validate artifact download and their dependencies.

## 6. Release Process

The main purpose of this project is to manage the Hadoop release process. The `build.xml` Ant script is the central component of this process. It defines a comprehensive workflow that includes:

*   Fetching release candidates.
*   Verifying GPG signatures.
*   Building from source.
*   Running tests against binary distributions.
*   Building and testing downstream projects.
*   Staging artifacts for release.
*   Generating release announcements and vote messages.

The `README.md` file contains a detailed step-by-step guide on how to perform a release using the Ant targets provided in this project.
