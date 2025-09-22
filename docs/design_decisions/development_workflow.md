# Development Workflow for the S-CORE Project

This document outlines the mandatory guidelines for all contributors and developers of the S-CORE project. Its purpose is to ensure smooth collaboration across our distributed repositories, maintain the stability and consistency of the `main` branches, and facilitate efficient development and integration.

---

## 1. Our Philosophy: Always Green `main` Branch

The `main` branch of each of our repositories **must always be stable and deployable**. This means:
*   No build failures.
*   All tests passing successfully.
*   No regressions that negatively impact the functionality of dependent components.

To achieve this, we rely on rigorous automated checks and a coordinated merge process.

---

## 2. Versioning and Package Management

*   **Semantic Versioning (SemVer):** Every module adheres to SemVer (MAJOR.MINOR.PATCH).
    *   **PATCH:** Backward-compatible bug fixes.
    *   **MINOR:** New, backward-compatible features.
    *   **MAJOR:** **NOT backward-compatible changes (Breaking Changes).** These require special coordination.
*   **Internal Package Repository:** All compiled artifacts of the modules are published to and consumed from a package repository.
*   **Dependency Declaration:** Modules declare their dependencies using SemVer-compatible version ranges (or pinned versions - has to be decided).

---

## 3. The Development Process: From Idea to `main`

### 3.1. Working with Feature Branches

All new developments (features, bug fixes, refactorings) must be done on dedicated feature branches. Direct commits to `main` are strictly prohibited.

1.  **Create a Branch:** Create a feature branch from `main` in your repository:
    ```bash
 git checkout main
 git pull origin main
 git checkout -b feature/<short_description_of_change>
    ```
2.  **Develop & Test:** Implement your changes and ensure your code is well-tested (unit tests, integration tests).
3.  **Commit & Push:** Regularly commit your changes and push them to your feature branch.

### 3.2. The Pull Request (PR)

Once your changes are ready, create a Pull Request (PR) from your feature branch to `main`.

1.  **Code Review:** Your PR will undergo code review by other team members.
2.  **CI Checks (Your Repo):** The Continuous Integration pipeline will run for your PR:
    *   Build of your module.
    *   Execution of all unit and integration tests *within your repository*.
    *   **IMPORTANT:** If you introduce changes to publicly exposed APIs (method signatures, class structures, etc.) in a "lower-level" module (e.g., `baselibs`), the CI pipeline will generate a **temporary pre-release version** of your module (e.g., `baselibs@1.2.3-PR-456.snapshot`) and publish it to the internal package repository. This happens **ONLY for the PR, not to `main`**.

### 3.3. The "Impact Test" (The Critical Step!)

After your repository's CI checks pass and a pre-release version is generated, the "Impact Test" is triggered:

*   **Trigger:** The central Integration CI pipeline is automatically triggered.
*   **Creation of "Integration Branches" in Dependent Repos:** For all "higher-level" repositories that depend on your module (e.g., `communications` depends on `baselibs`), a temporary "Integration Branch" will be created. This branch is typically branched off the current `main` branch of the dependent repo.
*   **Dependency Update:** On this Integration Branch, the dependency to *your* module will be updated to the **temporary pre-release version from your PR**.
*   **CI Checks (Dependent Repo):** The CI pipeline of the dependent repository will run on this Integration Branch:
    *   Build of the dependent module using your pre-release version.
    *   Execution of all unit, integration, and potentially end-to-end tests of the dependent module.
*   **Status Update:** The results of these "Impact Tests" will be reported back directly as a status check on **your original PR**.

### 3.4. Handling "Impact Test" Results

*   **Impact Test Successful (GREEN):**
    *   Congratulations! The changes are (from a compatibility perspective) safe and do not introduce regressions in dependent systems.
    *   The PR can be **automatically merged** to `main` once all status checks are green and code review is complete. (See "Automated Merge" below).
*   **Impact Test Failed (RED):**
    *   **STOP! Your PR must NOT be merged yet.** Your changes have introduced issues in a dependent system.
    *   **Who is Responsible?**
        *   **If the breaking change was unintended:** You (the developer of the lower-level module) are responsible for revising your PR to restore backward compatibility. This is the preferred solution.
        *   **If the breaking change is intended and necessary (e.g., a major refactoring, new architecture, Major version bump):**
            *   Your `baselibs` PR will remain open and blocked.
            *   You (the `baselibs` developer) **must actively coordinate** with the affected team(s) (e.g., `communications` team).
            *   Explain the reasons for the breaking change, document the new API, and provide migration guidance.
            *   The affected team(s) will then create **their own feature branch** (e.g., `feature/comm-adapt-baselibs-v2`) in their repository. They will adapt their code to the new API, pointing their dependency to your `baselibs` pre-release version.
            *   Their PR will also trigger Impact Tests against your blocked `baselibs` PR. Once their adaptations are complete and pass all tests, their PR will be ready.
            *   **Coordinated Merging:** Both your `baselibs` PR and the adaptation PR(s) from dependent repositories will need to be **merged simultaneously or in rapid succession** by the automated system to `main`. This ensures `main` stays green throughout the transition.

### 3.5. Automated Merge to `main`

Once all required reviews are complete, and all CI/CD status checks (including Impact Tests) are green, your PR will be automatically merged to `main`.
*   **No Manual Commits to `main`:** Developers cannot directly merge PRs to `main`. Only the automated CI/CD system can perform this action.
*   **Merge Conflict Resolution:** The CI/CD system will attempt a rebase-merge or merge of your feature branch into `main`. If merge conflicts occur, the automated merge will fail, and you will be notified to resolve the conflicts on your feature branch.

### 3.6. Post-Merge Actions

*   **New Stable Version:** Upon successful merge to `main`, the CI pipeline of your repository will publish a new, official stable version (e.g., `baselibs@1.2.4` or `baselibs@2.0.0`) to the internal package repository.
*   **Downstream `main` Branch Triggers:** The publication of this new stable version will automatically trigger the CI pipelines of all dependent repositories on their `main` branches to update their dependencies and verify full compatibility with the new official version.

---

## 4. Key Principles for Collaboration

*   **Communicate Early and Often:** Especially when planning or implementing changes that might impact other teams.
*   **Clear Ownership:** Each repository has a designated owning team responsible for its maintenance and evolution.
*   **Documentation:** Maintain up-to-date API documentation and migration guides for significant changes.
*   **Help Each Other:** If your changes cause issues in another repository, actively help the affected team resolve them.

By following this workflow, we ensure that the shared codebase remains robust, maintainable, and continuously shippable.
