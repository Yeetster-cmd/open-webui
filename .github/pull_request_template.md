
# Pull Request Checklist

**Before submitting, make sure you've checked the following:**

- [ ] **Description:** Provide a concise description of the changes made in this pull request down below.
- [ ] **Changelog:** Ensure a changelog entry following the format of [Keep a Changelog](https://keepachangelog.com/) is added at the bottom of the PR description.
- [ ] **Dependencies:** Are there any new or upgraded dependencies? If so, explain why, update the changelog/docs, and include any compatibility notes. Actually run the code/function that uses updated library to ensure it doesn't crash.
- [ ] **Testing:** Perform manual tests to **verify the implemented fix/feature works as intended AND does not break any other functionality**. Include reproducible steps to demonstrate the issue before the fix. Test edge cases (URL encoding, HTML entities, types). Take this as an opportunity to **make screenshots of the feature/fix and include them in the PR description**.
- [ ] **Agentic AI Code:** Confirm this Pull Request is **not written by any AI Agent** or has at least **gone through additional human review AND manual testing**. If any AI Agent is the co-author of this PR, it may lead to immediate closure of the PR.
- [ ] **Code review:** Have you performed a self-review of your code, addressing any coding standard issues and ensuring adherence to the project's coding standards?
- [ ] **Design & Architecture:** Prefer smart defaults over adding new settings; use local state for ephemeral UI logic. Open a Discussion for major architectural or UX changes.
- [ ] **Git Hygiene:** Keep PRs atomic (one logical change). Clean up commits and rebase on `dev` to ensure no unrelated commits (e.g. from `main`) are included. Push updates to the existing PR branch instead of closing and reopening.

# Changelog Entry

### Description

- [Concisely describe the changes made in this pull request, including any relevant motivation and impact (e.g., fixing a bug, adding a feature, or improving performance)]

### Added

- [List any new features, functionalities, or additions]

### Changed

- [List any changes, updates, refactorings, or optimizations]

### Deprecated

- [List any deprecated functionality or features that have been removed]

### Removed

- [List any removed features, files, or functionalities]

### Fixed

- [List any fixes, corrections, or bug fixes]

---

### Additional Information

- [Insert any additional context, notes, or explanations for the changes]
  - [Reference any related issues, commits, or other relevant information]


