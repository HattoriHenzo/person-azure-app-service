# Upgrade Plan: Person Manager (20261102000036)

- **Generated**: 2026-11-01 19:05 -05:00
- **HEAD Branch**: master
- **HEAD Commit ID**: HEAD (origin/master, no uncommitted tracked changes except `.mvn/wrapper/maven-wrapper.jar`)

## Available Tools

**JDKs**
- JDK 11.0.26: C:\Develop\java\11\bin (current project JDK, used by Step 2 baseline)
- JDK 17: C:\Develop\java\17\bin (required by Step 3)
- JDK 21: C:\Develop\java\21\bin (required by Step 5, intermediate)
- JDK 25.0.1: C:\Develop\java\25\bin (required by Step 6, final target)

**Build Tools**
- Maven Wrapper: 3.8.6 → **<TO_BE_UPGRADED>** to 3.9.16 (3.8.x is EOL; required for reliable Java 21/25 support)
- Maven (system): 3.8.1 (EOL; wrapper will be used for all build/test commands)

## Guidelines

> Note: You can add any specific guidelines or constraints for the upgrade process here if needed, bullet points are preferred.

## Options

- Working branch: appmod/java-upgrade-20261102000036
- Run tests before and after the upgrade: true

## Upgrade Goals

- Java: 11 → 25 (latest LTS)
- Spring Boot: 2.7.3 → 4.1.2 (latest stable release)

## Technology Stack

| Technology/Dependency      | Current | Min Compatible | Why Incompatible                                              |
| --------------------------- | ------- | --------------- | --------------------------------------------------------------- |
| Java                        | 11      | 25               | User requested                                                   |
| Spring Boot (parent)        | 2.7.3   | 4.1.2            | User requested                                                   |
| Maven Wrapper               | 3.8.6   | 3.9.16           | 3.8.x is EOL; recommended for Java 21/25 support                |
| maven-compiler-plugin       | inherited from parent | inherited | Managed by spring-boot-starter-parent; no explicit override needed |
| javax.persistence (JPA)     | javax   | jakarta.persistence | Namespace removed/replaced starting Spring Boot 3.0         |
| H2 Database                 | (parent-managed) | (parent-managed) | Managed by Spring Boot BOM; no explicit version pin found    |
| Lombok                      | (parent-managed) | (parent-managed) | Managed by Spring Boot BOM; compatible across target range   |
| JUnit 5 / AssertJ / Mockito | (parent-managed, via spring-boot-starter-test) | (parent-managed) | Managed by BOM, upgraded automatically with parent          |

## Derived Upgrades

- Spring Boot 3.x → Java 17+, Jakarta EE 9+ namespace (`javax.*` → `jakarta.*`), Hibernate 6.x, Spring Framework 6.x
- Spring Boot 4.x → Java 17+ (Java 25 LTS targeted), Jakarta EE 10+, Spring Framework 7.x
- Java 21/25 → Maven 3.9+ recommended; wrapper upgraded to 3.9.16
- No Kotlin present — Kotlin version rule not applicable
- No `javax.servlet`/Spring Security usage found — no servlet/security namespace rewrite needed

## Impact Analysis

### Dependency Changes

| File | Dependency | Current | Action | Target | Reason |
|------|-----------|---------|--------|--------|--------|
| pom.xml | spring-boot-starter-parent | 2.7.3 | upgrade | 3.5.16 | Intermediate stable 3.x LTS line before jumping to 4.x |
| pom.xml | java.version | 11 | upgrade | 17 | Required minimum for Spring Boot 3.x (Step 3) |
| pom.xml | spring-boot-starter-parent | 3.5.16 | upgrade | 4.1.2 | User requested latest Spring Boot (Step 6) |
| pom.xml | java.version | 17 | upgrade | 25 | User requested latest Java LTS (Step 6, via intermediate 21 in Step 5) |
| .mvn/wrapper/maven-wrapper.properties | distributionUrl / wrapperUrl | 3.8.6 / 3.1.0 | upgrade | 3.9.16 / 3.3.2 | 3.8.x EOL; required for stable Java 21/25 builds |

### Source Code Changes

| File | Location | Current | Required Change | Reason |
|------|----------|---------|----------------|--------|
| src/main/java/com/javathinked/application/spring/model/Person.java | import line 3 | `import javax.persistence.*;` | Replace with: `import jakarta.persistence.*;` | Jakarta EE 9+ namespace required by Spring Boot 3.x |

### Configuration Changes

None required — `application.yaml`/`application-test.yaml` use only `spring.datasource.*` and `spring.jpa.*` properties, which remain valid (not part of the renamed property set, e.g. `spring.redis.*`).

### CI/CD Changes

None found — no Dockerfile, GitHub Actions workflow, or pipeline files referencing a hardcoded JDK version exist in this repository (only `.github/modernize/appcat/assessment-config.yaml`, which is a modernization-tool config, not a build/deploy pipeline).

### Risks & Warnings

- **`LocalServerPort` import in `PersonControllerITest.java`**: Already imports `org.springframework.boot.test.web.server.LocalServerPort`. This package path must be re-verified after each Spring Boot major bump (2.7→3.5→4.1) since this annotation's package has moved across Spring Boot versions. **Mitigation**: Fix the import to the version-correct package at each step if compilation fails; rely on `mvn clean test-compile` to catch it immediately.
- **H2/Hibernate dialect changes**: Hibernate 6.x (Spring Boot 3.x) changed some dialect auto-detection behavior. `database-platform: org.hibernate.dialect.H2Dialect` is explicit and should continue to work, but **mitigation**: verify via `PersonRepositoryTest` and integration tests after each framework bump.
- **Spring Boot 4.x is very new (GA within the last few months)**: some third-party/transitive libraries may lag in compatibility. **Mitigation**: run full test suite at Step 6 and Final Validation; if issues arise, pin a compatible transitive version with documented justification.

## Upgrade Steps

- Step 1: Setup Environment
  - **Rationale**: Ensure all required JDKs and build tools are available before making changes.
  - **Changes to Make**: Verify JDK 11 (baseline), 17, 21, 25 availability via `#appmod-list-jdks`. Upgrade Maven Wrapper properties (`.mvn/wrapper/maven-wrapper.properties`) to Maven 3.9.16 / wrapper 3.3.2 per Dependency Changes table.
  - **Verification**: `./mvnw -v` using JDK 11 → Expected: Maven 3.9.16 reported, no errors.

- Step 2: Setup Baseline
  - **Rationale**: Establish compile/test baseline on current JDK 11 + Spring Boot 2.7.3 before any upgrade changes, to form the acceptance bar for test pass rate.
  - **Changes to Make**: None (no code changes — pure baseline measurement).
  - **Verification**: `./mvnw clean compile test-compile -q && ./mvnw clean test -q` with JDK 11 → Document SUCCESS/FAILURE and pass rate.

- Step 3: Upgrade to Spring Boot 3.5.16 + Java 17 (intermediate)
  - **Rationale**: Spring Boot 3.5.16 is the final stable 3.x LTS release, bridging 2.7.3 → 4.1.2 and requiring only Java 17 (already available), minimizing risk versus a direct 2.7→4.1 jump. Includes the mandatory Jakarta namespace migration.
  - **Changes to Make**: Apply `pom.xml` Dependency Changes (spring-boot-starter-parent → 3.5.16, java.version → 17). Apply Source Code Changes (`Person.java` javax.persistence → jakarta.persistence).
  - **Verification**: `./mvnw clean test-compile -q` with JDK 17 → Expected: compiles cleanly (main + test sources).

- Step 4: Validate Spring Boot 3.5.16 Runtime Behavior
  - **Rationale**: Confirm the 3.x intermediate is functionally correct (Hibernate 6 dialect behavior, `LocalServerPort` import, webflux test client) before progressing further, isolating any 3.x-specific issues from the 4.x jump.
  - **Changes to Make**: Fix any compilation/runtime issues surfaced by tests (e.g., adjust `LocalServerPort` import package if changed in 3.5.16).
  - **Verification**: `./mvnw clean test -q` with JDK 17 → Expected: all tests pass (or ≥ Step 2 baseline pass rate); document any deferred failures.

- Step 5: Upgrade Java to 21 (intermediate)
  - **Rationale**: Move the JDK target incrementally from 17 to 21 (Java 21 LTS) while still on Spring Boot 3.5.16, isolating JDK-version risk from the Spring Boot 4.x jump.
  - **Changes to Make**: Update `pom.xml` `<java.version>` to 21.
  - **Verification**: `./mvnw clean test-compile -q` with JDK 21 → Expected: compiles cleanly.

- Step 6: Upgrade to Spring Boot 4.1.2 + Java 25 (final targets)
  - **Rationale**: Apply the final user-requested targets — latest Spring Boot (4.1.2) and latest Java LTS (25) — now that the Jakarta namespace migration and intermediate JDK bump are already complete, isolating remaining Spring Boot 4.x-specific breaking changes into their own step.
  - **Changes to Make**: Update `pom.xml` (spring-boot-starter-parent → 4.1.2, java.version → 25). Re-check `LocalServerPort` import and any other Spring Boot 4.x breaking changes against the Risks & Warnings section.
  - **Verification**: `./mvnw clean test-compile -q` with JDK 25 → Expected: compiles cleanly.

- Step 7: CVE Validation & Fix
  - **Rationale**: Ensure no known vulnerable dependency versions remain after the upgrade.
  - **Changes to Make**: Extract direct dependencies via `./mvnw dependency:list -DexcludeTransitive=true`, scan with `#appmod-validate-cves-for-java`, upgrade any flagged versions (patch-level bumps acceptable), re-scan to confirm resolution.
  - **Verification**: `./mvnw clean test-compile -q` with JDK 25 → Expected: compiles cleanly; CVE re-scan shows no unresolved findings.

- Step 8: Final Validation
  - **Rationale**: Confirm all Upgrade Success Criteria are met end-to-end on the final target stack.
  - **Changes to Make**: Resolve any remaining TODOs/workarounds from prior steps. No further dependency changes expected.
  - **Verification**: `./mvnw clean test -q` with JDK 25 → Expected: 100% test pass rate (or ≥ Step 2 baseline); iterative fix loop until achieved.
